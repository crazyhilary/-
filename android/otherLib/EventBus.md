# EventBus

## 一、文档

官网：https://greenrobot.org/eventbus/

下载地址：https://github.com/greenrobot/EventBus

## 二、流程分析

#### 首先先理解下重要的几个类

1. ThreadMode

   ```java
   /**
    *	每个订阅方法都有一个订阅线程模式，该模式决定EventBus将在哪个线程中调用该方法
    * 	EventBus负责管理调用线程与方法要执行的线程之间的切换。
    *	默认为：POSTING，
    **/
   public enum ThreadMode {
   		//使用发布事件的线程来执行方法
       POSTING,
   		//使用主线程来执行方法，Android上是UI线程，如果非安卓中使用，则同：POSTING效果一样；
     	//如果在主线程上调用会直接调用订阅者方法，会阻塞提交线程，非主线程调用则不会阻塞线程。
       MAIN,
   		//与MAIN一样，区别是MAIN_ORDERED的调用者在主线程调用时是非阻塞调用。
       MAIN_ORDERED,
   		//在Android中，订阅者将在后台线程中被调用，如果提交线程不是主线程，则在提交线程中直接调用订阅者方法；
     	//如果发布线程是主线程，则会创建一个线程顺序调用订阅者方法。
       BACKGROUND,
      	//与‘BACKGROUND’相比，BACKGROUND模式，会顺序执行订阅者的方法；ASYNC模式，所有订阅者是并发执行。
       ASYNC
   }
   ```

2. SubscriberMethod

   ```java
   /**
    *	订阅方法解析类
    *	在调用 EventBus#register(Object)时，通过解析（SubscriberMethodFinder）
    *	把当前类的每个订阅方法解析成一个SubscriberMethod对象。
    *
    **/
   public class SubscriberMethod {
     	//订阅方法
       final Method method;
     	//线程模型，见：ThreadMode
       final ThreadMode threadMode;
     	//发送的消息
       final Class<?> eventType;
     	//方法优先级，值越大优先级越高，默认是0，同优先级先注册先执行
       final int priority;
     	//是否为粘性方法
       final boolean sticky;
       /** Used for efficient comparison */
       String methodString;
   
   }
   ```

3. Subscription

   ```java
   package org.greenrobot.eventbus;
   
   /**
    *	订阅者信息类，包含订阅者与订阅者里面的所有订阅方法
    *
    **/
   final class Subscription {
     	//注册订阅者对象
       final Object subscriber;
     	//订阅方法解析类
       final SubscriberMethod subscriberMethod;
       /**
        * Becomes false as soon as {@link EventBus#unregister(Object)} is called, which is checked by queued event delivery
        * {@link EventBus#invokeSubscriber(PendingPost)} to prevent race conditions.
        */
     	//当调用取消注册后（EventBus#unregister(Object)），此标志立即标记为：false
       volatile boolean active;
   
       Subscription(Object subscriber, SubscriberMethod subscriberMethod) {
           this.subscriber = subscriber;
           this.subscriberMethod = subscriberMethod;
           active = true;
       }
   }
   ```

#### 接下来分析注册流程

```java
public class EventBus {
	...
    public void register(Object subscriber) {
				//获取当前类名
        Class<?> subscriberClass = subscriber.getClass();
    		//提取订阅类中的订阅方法
        List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
        synchronized (this) {
            for (SubscriberMethod subscriberMethod : subscriberMethods) {
                subscribe(subscriber, subscriberMethod);
            }
        }
    }
  
  //注册主要类
  //订阅时，共用两个集合来存储数据对应关系
  //1. subscriptionsByEventType：事件(Event)->所有列表关于该Event的方法；
  //2. typesBySubscriber：订阅者(Object)->订阅者中的所有订阅方法，订阅者可以看作是一个Activity，
  //	订阅者方法可以看作是此Activity中带有注解：@Subscriber的方法。
   private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
     		//订阅方法参数类型
        Class<?> eventType = subscriberMethod.eventType;
     		//构建一个新的订阅类，订阅类与订阅方法对应
        Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
     		//通过订阅事件查找所有订阅类，如果事件是第一次订阅，则创建一个集合，用于存储所有关于此事件的订阅方法
        CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
        if (subscriptions == null) {
            subscriptions = new CopyOnWriteArrayList<>();
            subscriptionsByEventType.put(eventType, subscriptions);
        } else {
          	//检查是否重复注册，如一个Activity已注册过，再次注册就会拋出异常
            if (subscriptions.contains(newSubscription)) {
                throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                        + eventType);
            }
        }
				//处理订阅方法的优先级：优先级是按priority数值大小及订阅顺序来排序的
     		//priority值大则优先级高，相同时，先添加优先级高。
        int size = subscriptions.size();
        for (int i = 0; i <= size; i++) {
            if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
                subscriptions.add(i, newSubscription);
                break;
            }
        }

     		//此处代码是建立订阅者与当前订阅者中的方法建一个关联。
        List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
        if (subscribedEvents == null) {
            subscribedEvents = new ArrayList<>();
            typesBySubscriber.put(subscriber, subscribedEvents);
        }
        subscribedEvents.add(eventType);
				
     		//对sticky事件的处理
     		//遍历已发送的粘性事件，找出当前注册的订阅方法中是否有接受粘性事件的方法，有则发送事件给当前注册的订阅方法
     		//这里需要注意，注册事件的使用时机，在Activity#onCreate中注册事件时，当此Activity为非活动页面发送的事件
     		//再次变为活动页面时，是不会收到粘性事件调用的。这也是为什么提倡在onStart中注册事件的原因。
        if (subscriberMethod.sticky) {
          	//默认为true，在创建单例的时候可以修改
            if (eventInheritance) {
                Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
                for (Map.Entry<Class<?>, Object> entry : entries) {
                    Class<?> candidateEventType = entry.getKey();
                    if (eventType.isAssignableFrom(candidateEventType)) {
                        Object stickyEvent = entry.getValue();
                        checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                    }
                }
            } else {
                Object stickyEvent = stickyEvents.get(eventType);
                checkPostStickyEventToSubscription(newSubscription, stickyEvent);
            }
        }
    }
  
  private void checkPostStickyEventToSubscription(Subscription newSubscription, Object stickyEvent) {
        if (stickyEvent != null) {
            postToSubscription(newSubscription, stickyEvent, isMainThread());
        }
    }
  
  //根据订阅方法上的线程注解来分别做不同的线程调用
  private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
        switch (subscription.subscriberMethod.threadMode) {
            case POSTING:
                invokeSubscriber(subscription, event);
                break;
            case MAIN:
                if (isMainThread) {
                    invokeSubscriber(subscription, event);
                } else {
                    mainThreadPoster.enqueue(subscription, event);
                }
                break;
            case MAIN_ORDERED:
                if (mainThreadPoster != null) {
                    mainThreadPoster.enqueue(subscription, event);
                } else {
                    // temporary: technically not correct as poster not decoupled from subscriber
                    invokeSubscriber(subscription, event);
                }
                break;
            case BACKGROUND:
                if (isMainThread) {
                    backgroundPoster.enqueue(subscription, event);
                } else {
                    invokeSubscriber(subscription, event);
                }
                break;
            case ASYNC:
                asyncPoster.enqueue(subscription, event);
                break;
            default:
                throw new IllegalStateException("Unknown thread mode: " + 			 	 
                      subscription.subscriberMethod.threadMode);
        }
    }
  ...  
}
```

至此，注册流程已分析完成。接下来看下解除注册

```java
public class EventBus {
  ...
  //注册的时候，通过两个集合对数据进行存储，取消注册相应的操作就是把这两个集合中的相应数据删除
	public synchronized void unregister(Object subscriber) {
        List<Class<?>> subscribedTypes = typesBySubscriber.get(subscriber);
        if (subscribedTypes != null) {
            for (Class<?> eventType : subscribedTypes) {
                unsubscribeByEventType(subscriber, eventType);
            }
          	//删除注册对象
            typesBySubscriber.remove(subscriber);
        } else {
            logger.log(Level.WARNING, "Subscriber to unregister was not registered before: " + 
                       subscriber.getClass());
        }
  }
  
  //删除注册对象中关于监听EventType的方法
  //通过EventType获取所有关于此事件的监听方法封装类，通过遍历把关于当前类的监听方法找出来，并删除。
    private void unsubscribeByEventType(Object subscriber, Class<?> eventType) {
        List<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
        if (subscriptions != null) {
            int size = subscriptions.size();
            for (int i = 0; i < size; i++) {
                Subscription subscription = subscriptions.get(i);
                if (subscription.subscriber == subscriber) {
                    subscription.active = false;
                    subscriptions.remove(i);
                    i--;
                    size--;
                }
            }
        }
    }
  ...
}

```

取消注册流程相对简单，注册与取消注册已解决了。下面该发送事件

```java
public class EventBus {
	...
    //发送事件的时候，有两种选择，分别是postSticky/post，区别只有postSticky会把粘性事件放到stickyEvents中
   public void postSticky(Object event) {
        synchronized (stickyEvents) {
            stickyEvents.put(event.getClass(), event);
        }
        // Should be posted after it is putted, in case the subscriber wants to remove immediately
        post(event);
   }  
  
  //发送事件入口
  public void post(Object event) {
    		//获取当前线程的消息队列，并把要发送的事件放到队列中
        PostingThreadState postingState = currentPostingThreadState.get();
        List<Object> eventQueue = postingState.eventQueue;
        eventQueue.add(event);
				//判断当前线程是否在发送消息，如果正在发送消息则不处理，等待正在发送的消息完成后再处理。
        if (!postingState.isPosting) {
            postingState.isMainThread = isMainThread();
            postingState.isPosting = true;
            if (postingState.canceled) {
                throw new EventBusException("Internal error. Abort state was not reset");
            }
            try {
              	//进入队列循环发送消息模式
                while (!eventQueue.isEmpty()) {
                  	//获取队列第一个消息进行处理
                    postSingleEvent(eventQueue.remove(0), postingState);
                }
            } finally {
                postingState.isPosting = false;
                postingState.isMainThread = false;
            }
        }
  }
  
  //发送单个event事件到所有订阅方法中
  private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
        Class<?> eventClass = event.getClass();
        boolean subscriptionFound = false;
        if (eventInheritance) {
            List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);
            int countTypes = eventTypes.size();
            for (int h = 0; h < countTypes; h++) {
                Class<?> clazz = eventTypes.get(h);
                subscriptionFound |= postSingleEventForEventType(event, postingState, clazz);
            }
        } else {
            subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
        }
    		//根据调用结果，如果有调用订阅方法，则忽略，没有发现调用方法
        if (!subscriptionFound) {
            if (logNoSubscriberMessages) {
                logger.log(Level.FINE, "No subscribers registered for event " + eventClass);
            }
          	//当没有发现订阅方法时调用默认事件
            if (sendNoSubscriberEvent && eventClass != NoSubscriberEvent.class &&
                    eventClass != SubscriberExceptionEvent.class) {
                post(new NoSubscriberEvent(this, event));
            }
        }
  }
  
  //
  private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> 
                                              eventClass) {
        CopyOnWriteArrayList<Subscription> subscriptions;
        synchronized (this) {
          	//查询发送事件对应的方法列表
            subscriptions = subscriptionsByEventType.get(eventClass);
        }
    		//如果有订阅方法，则进入方法调用中
        if (subscriptions != null && !subscriptions.isEmpty()) {
            for (Subscription subscription : subscriptions) {
                postingState.event = event;
                postingState.subscription = subscription;
                boolean aborted;
                try {
                  	//进入发送消息调用方法
                    postToSubscription(subscription, event, postingState.isMainThread);
                  	//在方法执行完成后检查是否取消继续向后调用
                    aborted = postingState.canceled;
                } finally {
                    postingState.event = null;
                    postingState.subscription = null;
                    postingState.canceled = false;
                }
                if (aborted) {
                    break;
                }
            }
            return true;
       }
       return false;
    }
  
  ...
}
```

其它非主线程源码分析，在文章最后进行讲解。

# 三、使用

EventBus使用很简单总共分三步：

1. 定义事件类

   ```java
   public class MessageEvent {
    
       public final String message;
    
       public MessageEvent(String message) {
           this.message = message;
       }
   }
   ```

   最好定义有具体意义的类名，并加上Event标识，这样后期看到类名就知道是干什么用的

2. 准备订阅

   ```java
   @Subscribe(threadMode = ThreadMode.MAIN)
   public void onEventMessage(MessageEvent event) {
       Toast.makeText(getActivity(), event.message, Toast.LENGTH_SHORT).show();
   }
   ```

   EventBus 3.0以后虽然对订阅的方法名称没有做规定，但建议使用onEvent开头，这样在类中搜索起来比较方便，IDE的方法列表中也显示在一起。

   添加订阅方法完后，需要把当前类注册到EventBus中，这样订阅方法才可以收到事件调用，在安卓中，注册一般是按Activity或Fragment的生命周期进行注册和解除注册的。

   ```java
   @Override
   public void onStart() {
       super.onStart();
       EventBus.getDefault().register(this);
   }
    
   @Override
   public void onStop() {
       EventBus.getDefault().unregister(this);
       super.onStop();
   }
   ```

   如果没有特殊情况，不建议在onCreate中注册，如果类中的订阅方法只是接收数据操作，不做UI更新在onCreate做是比较好的，但在正常场景下都不会这样使用

3. 发送事件

   ```java
   EventBus.getDefault().post(new MessageEvent("Hello everyone!"));
   ```

   在代码的任何地方都可以使用，根据不同的目的使用post或postSticky。

# 四、其它源码分析

## EventBus配置

平常使用的时候傻傻的使用EventBus.getDefault()获取EventBus对象，但EventBus为我们提供了自定义对象的实现，我们可以通过EventBusBuilder可以配置一些信息

```java
public class EventBusBuilder {
  	//线程池，在我们应用中我们都会创建自己的一个线程池进行线程管理，每个涉及多线程操作的框架都会提供自定义线程池功能
    private final static ExecutorService DEFAULT_EXECUTOR_SERVICE = Executors.newCachedThreadPool();
    //是否打印订阅异常Log，触发此异常是在反射调用方法的时候invokeSubscriber
    boolean logSubscriberExceptions = true;
  	//没有订阅方法异常，在没有发现事件对应的订阅方法时，是否打异常信息
  	/**
  	 *	if (logNoSubscriberMessages) {
                logger.log(Level.FINE, "No subscribers registered for event " + eventClass);
            }
  	 *此事件调用基于throwSubscriberException设置为false
  	 */
    boolean logNoSubscriberMessages = true;
  	//EventBus会捕获异常，如果为true，当遇到反射方法异常时，会发送一个SubscriberExceptionEvent异常事件
  	//此事件的调用基于throwSubscriberException设置为false
    boolean sendSubscriberExceptionEvent = true;
  	//当发送的事件，未匹配到相应的接收者，会发送一个NoSubscriberEvent事件
    boolean sendNoSubscriberEvent = true;
  	//发送的事件，在调用方法时，有异常，是否把异常拋出来，如果此项设置为true，logSubscriberExceptions、		
  	//sendSubscriberExceptionEvent设置，也不会执行，默认为false，其它两事件为true
    boolean throwSubscriberException;
  	//事件eventType是否发送给他的父类监听者
    boolean eventInheritance = true;
  	//是否使用注解处理器预处理注解方法，还是通过反射运行时获取注解方法回调
  	//注解处理器可以提升运行时效率但，但减慢编译时间，可根据实际情况使用
    boolean ignoreGeneratedIndex;
  	//当注解方法不合法时，是否抛出异常信息，默认为false
    boolean strictMethodVerification;
  	//线程池，每个App都会有自己的池程池，为了不破坏线程池规则，第三方涉及多线程操作的框架一般都会提供使用者创建自己的线程池
    ExecutorService executorService = DEFAULT_EXECUTOR_SERVICE;
  	//旧版本使用，现在不现使用
    List<Class<?>> skipMethodVerificationForClasses;
  	//注解处理器预解析数据，解决反射带来的性能问题
    List<SubscriberInfoIndex> subscriberInfoIndexes;
  	//自定义Logger
    Logger logger;
  	//主线程处理器
    MainThreadSupport mainThreadSupport;
    
}
```





# 五、注意事项

注册时机、粘性事件或普通事件配合，来解决事件的操作与存储，比如App全局使用的环境变量，可以发送粘性事件，实现全局共享；如果想通过事件更新UI，须把注册时机放到onStart中，无论你发的是粘性事件还是普通事件，在UI进入onStop状态后，接收到的事件也不会刷新UI，相当于浪费了事件的发送；如果发送事件是粘性事件，订阅方法是sticky=true，UI再次获取焦点时，会再次接收到事件，可以做行UI的更新，一般不建议这样使用。

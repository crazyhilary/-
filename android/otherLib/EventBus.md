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
   private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
        Class<?> eventType = subscriberMethod.eventType;
        Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
        CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
        if (subscriptions == null) {
            subscriptions = new CopyOnWriteArrayList<>();
            subscriptionsByEventType.put(eventType, subscriptions);
        } else {
            if (subscriptions.contains(newSubscription)) {
                throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                        + eventType);
            }
        }

        int size = subscriptions.size();
        for (int i = 0; i <= size; i++) {
            if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
                subscriptions.add(i, newSubscription);
                break;
            }
        }

        List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
        if (subscribedEvents == null) {
            subscribedEvents = new ArrayList<>();
            typesBySubscriber.put(subscriber, subscribedEvents);
        }
        subscribedEvents.add(eventType);

        if (subscriberMethod.sticky) {
            if (eventInheritance) {
                // Existing sticky events of all subclasses of eventType have to be considered.
                // Note: Iterating over all events may be inefficient with lots of sticky events,
                // thus data structure should be changed to allow a more efficient lookup
                // (e.g. an additional map storing sub classes of super classes: Class -> List<Class>).
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
  ...  
}
```


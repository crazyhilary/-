# Jepack：Lifecycle

## 一、Lifecyle与Actiivty的关系

Lifecycle是Jepack组件中重中之重的组件，是Jepack组件存在的灵魂。下面是官方对Lifecycle解释

```
生命周期感知型组件可执行操作来响应另一个组件（如 Activity 和 Fragment）的生命周期状态的变化。这些组件有助于您写出更有条理且往往更精简的代码，这样的代码更易于维护。
```

先看下Lifecycle引入的地方（以Androidx为例，support也是一样）

Android.app.Activity这是Android SDK中的Activity，扩展包中的Activity都是基于此类做的扩展

<img src="http://qike.hilary.top/uPic/Activity继承关系图%20%281%29.png" alt="Activity继承关系图 (1)"  />



Lifecycle生命周期是在FragmentActivity的生命周期方法中做的调用，所以说如果想使用Lifecycle生命周期功能，早于FragmentActivity类是不生效的。

## 二、Lifecycle相关类

###  1、LifecycleObsever

1. 空的观察者接口`LifecycleObsever`

2. 继续`LifecycleObserver`，添加观察生命周期方法

   ```java
   interface FullLifecycleObserver extends LifecycleObserver {
   
       void onCreate(LifecycleOwner owner);
   
       void onStart(LifecycleOwner owner);
   
       void onResume(LifecycleOwner owner);
   
       void onPause(LifecycleOwner owner);
   
       void onStop(LifecycleOwner owner);
   
       void onDestroy(LifecycleOwner owner);
   }
   ```

3. 用户使用的默认观察者 ：`DefaultLifecycleObserver`，继承于`FullLifecycleObserver`, 实现接口所有方法，我们使用时，可以直接使用此类。

### 2、LifecycleEventObserver

​	   此类是观察事件变化，只有一个方法：onStateChanged，不细分到那种事件，而FullLifecycleObserver是细分到事件的，可以理他们两者功能是一样，用途不同，如果同时实现这两个接口，则会先调用FullLifecycleObserver中的方法，然后调用本类的onStateChanged方法，FullLifecycleObserverAdapter就是实现这两个接口分发工作的适配器。

### 3、LifecycleOwner

LifecycleOwner，处理生命周期事件的类实现此接口，在Activity或Fragement中实现了此接口，也可以在自定义的类中实现。

唯一方法

```java
Lifecycle getLifecycle();
```



### 4、Lifecycle

用于存储有关组件（Activity/Fragment）的生命周期状态的信息，并允许其它对象观察此状态。

观察者容器：添加观察者、移出观察者，获取当前生命周期状态

此类中包含一个`Event`和`State`

**特别注意：事件和状态的区分，事件和我们理解的Acitivty周期是对应的。**

**状态，只有初始状态（INITIALIZED）、创建状态（CREATED）、开始状态（STARTED）、焦点状态（RESUMED）**

发送一个事件`ON_PAUSE`,状态会从状态`RESUMED`退回到`STARTED`

1. Event

   事件包含整个Activity生命周期事件：ON_CREATE、ON_START、ON_RESUME、ON_PAUSE、ON_STOP、ON_DESTROY、ON_ANY，前三个事件在Activity组件周期onCreate、onStart、onResume之后调用，后面三个事件在Actiivty周期组件前调用。
   
1. State

1. 生命周期的状态：

   * DESTROYED
   
     LifecycleOwner的销毁状态，例如会在Activity的onDestroyed方法之前调用事件ON_DESTROY，调用后LifecycleOwner不再派发任何事件
   
   * INITIALIZED
   
     LifecycleOwner被初始化状态，对于Activity来说，还不有接收任何状态
   
   * CREATED
   
     LifecycleOwner的创建状态，对于Activity来说，进入此状态有两种情况，一是调用onCreate后，另一种是要执行onStop之前
   
   * STARTED
   
     LifecycleOwner的开始状态，对于Activity来说，进入此状态有两种情况，一是调用onStart后，另一种是要执行onPause之前
   
   * RESUMED
   
     LifecycleOwner的恢复状态，对于Activity来说，调用onResumed之后。

### 5、LifecycleRegistry

LifecycleRegistry是Lifecycle的实现类，用于管理观察者的容器，他可以在Fragment或Acitivyt中用，也可以在自己定义的Lifecycle中直接使用。

## 三、流程

### 测试代码及结果

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        lifecycle.addObserver(object : LifecycleEventObserver{
            override fun onStateChanged(source: LifecycleOwner, event: Lifecycle.Event) {
                println("######onStateChanged: ${event.name}")
            }
        })
    }

}
```

打开`MainActivity`，息屏再打开，这个流程打印日志

```java
######onStateChanged: ON_CREATE
######onStateChanged: ON_START
######onStateChanged: ON_RESUME
######onStateChanged: ON_PAUSE
######onStateChanged: ON_STOP
######onStateChanged: ON_START
######onStateChanged: ON_RESUME
```

### 流程跟踪

在`FragmentActivity`中定义了LifecycleRegistry，父类的`ComponentActivity`中实现了LifecycleOwner，所以我们可以在当前Activity中就可以拿到LifecycleRegistry，把我们的观察者注册到这个容器中，下面代码是添加观察者操作

```java
 @Override
    public void addObserver(@NonNull LifecycleObserver observer) {
      //在创建对象的时候设置是否在主线程中操作，现在只有一个默认值，必须在主线程中操作，此处检查当前操作是否在主线程
        enforceMainThreadIfNeeded("addObserver");
      //mState为当前状态，如果为DESTROYED，则标记新添加的观察者状态也是DESTROYED，否则初始默认状态为INITIALIZED
        State initialState = mState == DESTROYED ? DESTROYED : INITIALIZED;
      //包装观察者，主要逻辑在Lifecycling类中，通过适配器做事件分发，在Lifecycling.lifecycleEventObserver(obsever)
      //中做不同观察者事件分发
        ObserverWithState statefulObserver = new ObserverWithState(observer, initialState);
      //把当前添加的观察者放入容器中，如果已存在则返回旧值
        ObserverWithState previous = mObserverMap.putIfAbsent(observer, statefulObserver);
			//如果已被添加过，则不做后需要事件分发
        if (previous != null) {
            return;
        }
      //新添加观察者，把当前容器的状态遍历分发给新添加的观察者，比如：当前Activity为onResume，这时添加观察者，观察者会依次收到
      //事件：CREATED、STARTED、RESUMED
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            // it is null we should be destroyed. Fallback quickly
            return;
        }
				//多线程操作时产生竞争才会产生mAddingObserverCounter >0 的情况，现在只会等于0
      	//mHandlingEvent也是只有会false
        boolean isReentrance = mAddingObserverCounter != 0 || mHandlingEvent;
      //计算当前添加的观察者的状态
        State targetState = calculateTargetState(observer);
        mAddingObserverCounter++;
      //如果当前添加的观察者状态小低于当前Activity的状态，则发送事件
        while ((statefulObserver.mState.compareTo(targetState) < 0
                && mObserverMap.contains(observer))) {
            pushParentState(statefulObserver.mState);
          	/**获取当前观察者的高一级状态: 
          	 * INITIALIZED 	-> ON_CREATE 
          	 * CREATED     	-> ON_START
          	 * STARTED			-> ON_RESUME
          	 * 其它状态为		 -> NULL
          	 **/
            final Event event = Event.upFrom(statefulObserver.mState);
            if (event == null) {
                throw new IllegalStateException("no event up from " + statefulObserver.mState);
            }
          	//分发事件
            statefulObserver.dispatchEvent(lifecycleOwner, event);
            popParentState();
            // 重新计算添加的观察者状态是否已达到当前Acitvity对应的状态，否则继续分发更高一级的事件
            targetState = calculateTargetState(observer);
        }

      	//此判断，一直都会进入同步状态
        if (!isReentrance) {
            //同步状态，此方法为核心方法，生命周期改变也会通过此方法进行事件处理
            sync();
        }
        mAddingObserverCounter--;
    }

		private void sync() {
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            throw new IllegalStateException("LifecycleOwner of this LifecycleRegistry is already"
                    + "garbage collected. It is too late to change lifecycle state.");
        }
        while (!isSynced()) {
            mNewEventOccurred = false;
            // no need to check eldest for nullability, because isSynced does it for us.
            if (mState.compareTo(mObserverMap.eldest().getValue().mState) < 0) {
                backwardPass(lifecycleOwner);
            }
            Map.Entry<LifecycleObserver, ObserverWithState> newest = mObserverMap.newest();
            if (!mNewEventOccurred && newest != null
                    && mState.compareTo(newest.getValue().mState) > 0) {
                forwardPass(lifecycleOwner);
            }
        }
        mNewEventOccurred = false;
    }
	
		//是否需要同步
		private boolean isSynced() {
      	//没有观察者时不需要同步，直接返回
        if (mObserverMap.size() == 0) {
            return true;
        }
      	//当前状态与已添加的所有观察者的状态一至，则不需要同步，否则同步状态。
        State eldestObserverState = mObserverMap.eldest().getValue().mState;
        State newestObserverState = mObserverMap.newest().getValue().mState;
        return eldestObserverState == newestObserverState && mState == newestObserverState;
    }

		//生命周期从INITIALIZED-->RESUMED
		private void forwardPass(LifecycleOwner lifecycleOwner) {
        Iterator<Map.Entry<LifecycleObserver, ObserverWithState>> ascendingIterator =
                mObserverMap.iteratorWithAdditions();
        while (ascendingIterator.hasNext() && !mNewEventOccurred) {
            Map.Entry<LifecycleObserver, ObserverWithState> entry = ascendingIterator.next();
            ObserverWithState observer = entry.getValue();
            while ((observer.mState.compareTo(mState) < 0 && !mNewEventOccurred
                    && mObserverMap.contains(entry.getKey()))) {
                pushParentState(observer.mState);
	              //重点理解这行代码
                final Event event = Event.upFrom(observer.mState);
                if (event == null) {
                    throw new IllegalStateException("no event up from " + observer.mState);
                }
                observer.dispatchEvent(lifecycleOwner, event);
                popParentState();
            }
        }
    }

		//生命周期从RESUMED-->DESTROYED
    private void backwardPass(LifecycleOwner lifecycleOwner) {
          Iterator<Map.Entry<LifecycleObserver, ObserverWithState>> descendingIterator =
                  mObserverMap.descendingIterator();
          while (descendingIterator.hasNext() && !mNewEventOccurred) {
              Map.Entry<LifecycleObserver, ObserverWithState> entry = descendingIterator.next();
              ObserverWithState observer = entry.getValue();
              while ((observer.mState.compareTo(mState) > 0 && !mNewEventOccurred
                      && mObserverMap.contains(entry.getKey()))) {
                  //重点理解这行代码
                	final Event event = Event.downFrom(observer.mState);
                  if (event == null) {
                      throw new IllegalStateException("no event down from " + observer.mState);
                  }
                  pushParentState(event.getTargetState());
                  observer.dispatchEvent(lifecycleOwner, event);
                  popParentState();
              }
          }
      }

```

## 扩展

### 1. 总结

就像官网上说的

```
		生命周期感知型组件可执行操作来响应另一个组件（如 Activity 和 Fragment）的生命周期状态的变化。这些组件有助于您编写出更有条理且往往更精简的代码，此类代码更易于维护。
		一种常见的模式是在 Activity 和 Fragment 的生命周期方法中实现依赖组件的操作。但是，这种模式会导致代码条理性很差而且会扩散错误。通过使用生命周期感知型组件，您可以将依赖组件的代码从生命周期方法移入组件本身中。
```

Lifecyle生命周期组件可以单独使用，也可以配合其它组件使用，核心是为了解决把依赖于Activity生命周期操作独立到相应的组件中，不必要再放到Activity中，组件更加独立，Activity可以很轻松集成这些组件，也不用再管理其依赖组件的生命周期。

### 2. 解决观察应用前后台的解决方案

​	ProcessLifecycleOwner通过实现LifecycleOwner，很好了解决了观察应用处于前后台的判断



### 3. 常用生命组件库

```kotlin
dependencies {
        def lifecycle_version = "2.5.0-alpha04"
        def arch_version = "2.1.0"

        // ViewModel
        implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
        // ViewModel utilities for Compose
        implementation "androidx.lifecycle:lifecycle-viewmodel-compose:$lifecycle_version"
        // LiveData
        implementation "androidx.lifecycle:lifecycle-livedata-ktx:$lifecycle_version"
        // Lifecycles only (without ViewModel or LiveData)
        implementation "androidx.lifecycle:lifecycle-runtime-ktx:$lifecycle_version"

        // Saved state module for ViewModel
        implementation "androidx.lifecycle:lifecycle-viewmodel-savedstate:$lifecycle_version"

        // Annotation processor
        kapt "androidx.lifecycle:lifecycle-compiler:$lifecycle_version"
        // alternately - if using Java8, use the following instead of lifecycle-compiler
        implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"

        // optional - helpers for implementing LifecycleOwner in a Service
        implementation "androidx.lifecycle:lifecycle-service:$lifecycle_version"

        // optional - ProcessLifecycleOwner provides a lifecycle for the whole application process
        implementation "androidx.lifecycle:lifecycle-process:$lifecycle_version"

        // optional - ReactiveStreams support for LiveData
        implementation "androidx.lifecycle:lifecycle-reactivestreams-ktx:$lifecycle_version"

        // optional - Test helpers for LiveData
        testImplementation "androidx.arch.core:core-testing:$arch_version"
    }
```


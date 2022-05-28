# LiveData

## 一、LiveData

```java
public abstract class LiveData<T> {
  	//记录所有注册的观察者，在注册、移除注册、生命同期有变化遍历的时候使用
    private SafeIterableMap<Observer<? super T>, ObserverWrapper> mObservers =
            new SafeIterableMap<>();
  	//记录有多少观察者处于活跃状态
  	int mActiveCount = 0;
		//状态改变后，处理事件时标识为true，说明在处理状态，当处理状态时，过虑掉新事件的处理
  	//当事件处理完成后置为false
  	private boolean mChangingActiveState;
  	//数据版本号，当有新值来时，版本号加1，处理数据的时候根据版本号来判断数据是否需要再次更新
  	private int mVersion;
  	...
    /**	数据分发  
     *	通过initiator参数来区分更新当前Oberser还是更新所有；
     *	如果是通过setValue(object)设置值时，这时更新所有Oberser；
     *	如果是通过Lifecycle状态变化更新的话，只更新当前Oberser，因为这里的状态变化会通知所有Oberser。
     *	在注册Observer时，区分了是否监听Lifecycle状态变化，监听与不监听的区别时，
     * 	当Lifecycle状态变化时，传入进来的initiator不为空，也就是说Lifecycle状态变化是会通知监听的观察者,
     *	所以通过observeForever注册的Observer不会通知变化，也不需要通知。
     **/
  	void dispatchingValue(@Nullable ObserverWrapper initiator) {
      	//是否正在处理数据
        if (mDispatchingValue) {
            mDispatchInvalidated = true;
            return;
        }
      	//标识正在处理数据，防止重复操作
        mDispatchingValue = true;
        do {
            mDispatchInvalidated = false;
          	//是否更新当前Observer对象，如果是则直接更新
          	//不是，则遍历所有观察者并更新；
            if (initiator != null) {
                considerNotify(initiator);
                initiator = null;
            } else {
                for (Iterator<Map.Entry<Observer<? super T>, ObserverWrapper>> iterator =
                        mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                    considerNotify(iterator.next().getValue());
                  	//当前正在分发数据时，又有新数据更新，则停止当前动作，重新分派新值
                    if (mDispatchInvalidated) {
                        break;
                    }
                }
            }
        } while (mDispatchInvalidated);
        mDispatchingValue = false;
    }
  
  	/**
  	 *	添加Observer到Owner中，这是我们必用的方法，通过观察当前页面是否为活跃状态更新数据，更新UI时必用方法
  	 *	如果不涉及UI更新，观察所有数据变化，可以使用另一个同名方法
  	 **/
  	@MainThread
    public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
        assertMainThread("observe");
      	//如果当前状态为DESTROYED，则抛弃不做处理
        if (owner.getLifecycle().getCurrentState() == DESTROYED) {
            // ignore
            return;
        }
      	//包装Observer，做数据更新判断，自动移除注册处理。
        LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
      	//如果observer已绑定了owner，且不是当前的owner，则抛异常
        if (existing != null && !existing.isAttachedTo(owner)) {
            throw new IllegalArgumentException("Cannot add the same observer"
                    + " with different lifecycles");
        }
      	//如果已重复添加则不做处理
        if (existing != null) {
            return;
        }
        owner.getLifecycle().addObserver(wrapper);
    }
  
  	/**
  	 *	添加无关状态的Observer
  	 *	目的实现无关UI更新操作，只更新数据时使用
  	 * 	当Lifecycle状态变化时也不会通知通过此方法注册的Observer
  	 **/
  	@MainThread
    public void observeForever(@NonNull Observer<? super T> observer) {
        assertMainThread("observeForever");
        AlwaysActiveObserver wrapper = new AlwaysActiveObserver(observer);
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
      	//由于此方法无关UI操作，所以可以绑定不同的Owner，但不可以通过observe(LifecycleOwner,Observer)添加绑定
      	//因为observe是基本UI更新操作的观察，与当前方法使用模式正好互斥
        if (existing instanceof LiveData.LifecycleBoundObserver) {
            throw new IllegalArgumentException("Cannot add the same observer"
                    + " with different lifecycles");
        }
        if (existing != null) {
            return;
        }
        wrapper.activeStateChanged(true);
    }
  	
  	/**
  	 *	判断当前LiveData是否有活跃的Observer
  	 *	这个值是通过LiveData中的Observer自己修改的，
  	 *  如果当前Lifecycle为不活跃状态则统计数为0，否则为当前已注册的观察者总数
  	 **/
  	public boolean hasActiveObservers() {
        return mActiveCount > 0;
    }
  
  	/**	是否需要更新数据
  	 *	1. 当前页面的状态是否符合更新条件；
     *	2. 数据版本号是否已更新过。
     **/
  	private void considerNotify(ObserverWrapper observer) {
        if (!observer.mActive) {
            return;
        }
        // Check latest state b4 dispatch. Maybe it changed state but we didn't get the event yet.
        //
        // we still first check observer.active to keep it as the entrance for events. So even if
        // the observer moved to an active state, if we've not received that event, we better not
        // notify for a more predictable notification order.
        if (!observer.shouldBeActive()) {
            observer.activeStateChanged(false);
            return;
        }
        if (observer.mLastVersion >= mVersion) {
            return;
        }
        observer.mLastVersion = mVersion;
        observer.mObserver.onChanged((T) mData);
    }
  
  	/**
  	 *	忽略Owner状态的Observer，有数据就更新，不可用于UI操作的观察者。
  	 **/
  	private class AlwaysActiveObserver extends ObserverWrapper {

        AlwaysActiveObserver(Observer<? super T> observer) {
            super(observer);
        }

        @Override
        boolean shouldBeActive() {
            return true;
        }
    }
  	
  	/**
  	 *	在子线程中调用，发送值到主线程更新数据
  	 *	postValue(“1”);
  	 *	setValue("2);
  	 *	会接连收到两个值，分别为2、1，由于postValue会通过Handler从新排队放到主线程操作
  	 *  setValue是在当前主线程中直接操作，但前者不会阻塞线程
  	 **/
  	protected void postValue(T value) {
        boolean postTask;
        synchronized (mDataLock) {
            postTask = mPendingData == NOT_SET;
            mPendingData = value;
        }
        if (!postTask) {
            return;
        }
        ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
    }
  
  	/**
  	 *	只能在主线程中调用更数据操作
  	 **/
  	@MainThread
    protected void setValue(T value) {
        assertMainThread("setValue");
        mVersion++;
        mData = value;
        dispatchingValue(null);
    }
  
}
```



LiveData有几个辅助类，来保证LiveData的生命周期管理

### ObserverWrapper

```java
//观察者的包装类，扩展Lifecycle包中的Objserver类，添加当前状态及版本号
//此包装类主要是提供判断生命周期状态及数据派发
private abstract class ObserverWrapper {
        final Observer<? super T> mObserver;
        boolean mActive;
        int mLastVersion = START_VERSION;

        ObserverWrapper(Observer<? super T> observer) {
            mObserver = observer;
        }

        abstract boolean shouldBeActive();

        boolean isAttachedTo(LifecycleOwner owner) {
            return false;
        }

        void detachObserver() {
        }

  			//每当页面的生命周期变化都会调用，此方法决定是否做数据下发
        void activeStateChanged(boolean newActive) {
          	//新状态与旧状态是否一致，避免重复发送数据
            if (newActive == mActive) {
                return;
            }
            mActive = newActive;
          	//根据自身的状态设置LiveData活跃的Observer数量，自身活跃+1，不活跃-1
            changeActiveCounter(mActive ? 1 : -1);
          	//判断当前页面的生命周期是否为活跃状态，如果不是则不分发数据
            if (mActive) {
                dispatchingValue(this);
            }
        }
}
```

### LifecycleBoundObserver

```java
//再次包装LifecycleOwner，实现生命周期处理，判断Owner的邦定逻辑
class LifecycleBoundObserver extends ObserverWrapper implements LifecycleEventObserver {
        @NonNull
        final LifecycleOwner mOwner;

        LifecycleBoundObserver(@NonNull LifecycleOwner owner, Observer<? super T> observer) {
            super(observer);
            mOwner = owner;
        }

  			//比较当前页面的生命周期是否处于活跃状态，判断是否可以发送数据
  			//如果我们想实现一个不受生命周期就派发数据的功能，可以写死为true。
        @Override
        boolean shouldBeActive() {
            return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
        }

  			//设置观察者的状态，如果处于DESTROYED，则把观察者移除
  			//观察者操作都需要注册与取消注册操作，但LiveData我们使用时，只需要添加注册，不需要处理取消注册，
  			//是因为这里实现了取消注册行为，当状态为DESTROYED时，说明当前页面不再需要处理数据了，这里自动取消了注册行为。
        @Override
        public void onStateChanged(@NonNull LifecycleOwner source,
                @NonNull Lifecycle.Event event) {
            Lifecycle.State currentState = mOwner.getLifecycle().getCurrentState();
            if (currentState == DESTROYED) {
                removeObserver(mObserver);
                return;
            }
          	//Lifecyle特点时不会丢失事件，当处理事件1时，事件2也发生了，当事件1处理完还会再继续处理事件2
            //直到事件处理到与当前页面同级别状态为止。
            Lifecycle.State prevState = null;
            while (prevState != currentState) {
                prevState = currentState;
                activeStateChanged(shouldBeActive());
                currentState = mOwner.getLifecycle().getCurrentState();
            }
        }

  			//注册与移除注册的时候使用，判断重复注册与重复移除
        @Override
        boolean isAttachedTo(LifecycleOwner owner) {
            return mOwner == owner;
        }

  			//移除注册
        @Override
        void detachObserver() {
            mOwner.getLifecycle().removeObserver(this);
        }
    }
```



## 二、MutableLiveData

MutableLiveData是LiveData的实现类，LiveData是不可以直接使用的，我们可以能过此类来实现没有定制化的功能

## 三、MediatorLiveData

我们可以理解为此类为了实现数据转换而存在的，参考 io.reactivex设计理念，单看此类可能会不理解具体的使用，可以参考`Transformations`工具类的使用。

我们先看下具体的代码实现

```java
public class MediatorLiveData<T> extends MutableLiveData<T> {
  	//排队Observer的多地方注册及管理生命周期变化活跃与非活跃状态之间的切换做注册与移除注册管理操作的数据源
  	private SafeIterableMap<LiveData<?>, Source<?>> mSources = new SafeIterableMap<>();
  	
  	/**
  	 *	添加LiveData数据源与数据的观察者
  	 *	判断添加的LiveData是否已经注册过，如果已注册过则判断当前注册与以前注册的Observer是否为同一个
  	 *	如果是则不处理，如果不是则抛出异常
  	 **/
  	public <S> void addSource(@NonNull LiveData<S> source, @NonNull Observer<? super S> onChanged) {
        Source<S> e = new Source<>(source, onChanged);
        Source<?> existing = mSources.putIfAbsent(source, e);
        if (existing != null && existing.mObserver != onChanged) {
            throw new IllegalArgumentException(
                    "This source was already added with the different observer");
        }
        if (existing != null) {
            return;
        }
      	//hasActiveObservers()是判断当前Lifecycle是否处理活跃状态，如果不活跃则Observer不添加到源数据观察者的行列中
      	//这个判断表明，如果我们不把MediatorLiveData注册到Lifecycle中，我们添加的源数据变化是不会被他绑定的观察者感觉到的
      	//这个功能要想实现我们想要的功能，把MediatorLiveData注册到Lifecycle中是必要的操作。
        if (hasActiveObservers()) {
            e.plug();
        }
    }
  
  	
  	/**
  	 *	当Lifecycle为活跃状态时，把源数据与它的Observer绑定到一起，观察源数据的变化
  	 *	
  	 **/
    @CallSuper
    @Override
    protected void onActive() {
        for (Map.Entry<LiveData<?>, Source<?>> source : mSources) {
            source.getValue().plug();
        }
    }
  
  	/**
  	 *	数据源包装类，添加的数据源都是通过此类做行包装后放到Map集合中
  	 *	此类也是一个Observer实现，先把自己注册到别的地方当作观察者，当有数据变化的时候，通过自己的onChanged(Object)
  	 *  再把变化传给真正的Observer，这里也做一次版本号判断，过虑掉重复触发事件。
  	 *	此类的plug()与unplug()注册与移除注册是通过外部类监听onActive()与onInactive()实现的，
  	 *	它本身不需要关心生命周期的状态，这里也是使用的observeForever()；
  	 **/
  	private static class Source<V> implements Observer<V> {
        final LiveData<V> mLiveData;
        final Observer<? super V> mObserver;
        int mVersion = START_VERSION;

        Source(LiveData<V> liveData, final Observer<? super V> observer) {
            mLiveData = liveData;
            mObserver = observer;
        }

        void plug() {
            mLiveData.observeForever(this);
        }

        void unplug() {
            mLiveData.removeObserver(this);
        }

        @Override
        public void onChanged(@Nullable V v) {
            if (mVersion != mLiveData.getVersion()) {
                mVersion = mLiveData.getVersion();
                mObserver.onChanged(v);
            }
        }
    }
}
```

代码其实很简单，但理解使用有些困难，下面我们分析下使用方式

从源码中，我们可以发现，要想使用`MediatorLiveData`功能，必须把`MediatorLiveData`对象注册到Lifecycle中去，才可能触发源数据的变化同步到相应的Observer中。

我们可以利用这一特性，来实现数据传送的开关，正常我们使用在Activity或Fragemtn中是没有问题的，我们任意使用Owner进行注册功能，但如果是放到ViewModel或其它没有Owner的场景，就比较困难了。我们以ViewModel举例，





## 四、Transformations



## 总结

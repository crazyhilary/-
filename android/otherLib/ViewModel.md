# ViewModel

ViewModel为Activity、Fragment准备和管理必要的数据，负责Activity、Fragment之间的数据通讯，做业务逻辑处理。

ViewModel使用一般伴随着一个Activity或Fragment的生命周期，随着伴随者生命周期终结而终结。

ViewModel有一个很重要的重用，就是当Activity配置改变，并不是真正的销毁时，ViewModel会保存数据，并为UI还原提供改变前的数据，为什么ViewModel可以在Activity被销毁后还可以恢复数据，请看下面`ViewModelProvider`分析。

注意：在使用ViewModel时，Viewmodel的职责就是帮UI管理数据，并不要持有UI控件及Activity或Fragment

# 类解析

## 一、ViewModel

ViewModel类很简单，提供一个存放tag的Map集合，我们重写时，如果对象被销毁时，会被调用一个方法`onCleared`在这里面释放我们的资源。tag集合中如果包含有`Closeable`功能的对象，ViewModel会自动帮我们调用`close`。

ViewModel有一个子类`AndroidViewModel`，添加了对Application对象的持有，其它的并没有变化。

## 二、ViewModelStore

ViewModelStore用于存储ViewModel，此类仅提供一个Map来存放ViewModel，通过`clear`清理存储的ViewModel对象。`cleaer`方法的调用是通过在Activity中观察Lifecycle的生命同期，当收到ON_DESTROY事件时，判断并不是改变配置的ON_DESTROY事件，则做清除数据处理

```java
getLifecycle().addObserver(new LifecycleEventObserver() {
            @Override
            public void onStateChanged(@NonNull LifecycleOwner source,
                    @NonNull Lifecycle.Event event) {
                if (event == Lifecycle.Event.ON_DESTROY) {
                    // Clear out the available context
                    mContextAwareHelper.clearAvailableContext();
                    // And clear the ViewModelStore
                    if (!isChangingConfigurations()) {
                        getViewModelStore().clear();
                    }
                }
            }
        });
```



ViewModelStoreOwner接口用于实现类获取ViewModelStore对象`ComponentActivity`类实现了此接口，

# 三、ViewModelProvider

ViewModelProvider是工具类，通过此工具类创建ViewModel对象，默认为Activity或Fragment

```java
public class ViewModelProvider {
  ...
  //ViewModel的工厂类
	public interface Factory {
        @NonNull
        <T extends ViewModel> T create(@NonNull Class<T> modelClass);
  }
  //由于Activity或Fragment实现了ViewModelStoreOwner接口，在使用的时候可以直接传Activity或Fragment对象就可以
  //创建ViewModel对象，我们可以在Actiivty或Fragment中实现一个自己的创建ViewModel工厂HasDefaultViewModelProviderFactory，
  //这样在创建ViewMoel时，会调用我们自己的工厂去创建ViewModel。
  //如果没有实现默认的工厂则由Provider的NewInstanceFactory工厂去创建ViewModel
  public ViewModelProvider(@NonNull ViewModelStoreOwner owner) {
        this(owner.getViewModelStore(), owner instanceof HasDefaultViewModelProviderFactory
                ? ((HasDefaultViewModelProviderFactory) owner).getDefaultViewModelProviderFactory()
                : NewInstanceFactory.getInstance());
  }
  
  //通过自定义的工厂去创建ViewModel或AndroidViewModel
  //创建AndroidViewModel可以传入AndroidViewModelFactory工厂类
  public ViewModelProvider(@NonNull ViewModelStoreOwner owner, @NonNull Factory factory) {
        this(owner.getViewModelStore(), factory);
  }
  
  public <T extends ViewModel> T get(@NonNull Class<T> modelClass) {
        String canonicalName = modelClass.getCanonicalName();
        if (canonicalName == null) {
            throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
        }
        return get(DEFAULT_KEY + ":" + canonicalName, modelClass);
  }
  
  /**
   *	通过指定的Key和要创建的ViewModel来获取ViewModel对象	
   *	如果创建过则读取Map中的缓存对象，否则通过相应的工厂创建ViewModel对象
   *	mViewModelStore是Activity或Fragment传过来的，当前Activity更改配置时，mViewModelStore会存储更改配置前的数据
   *	这样实现了更改配置后，ViewModel还能保存以前的数据。
   *	mViewModelStore的数据在切换横屏后还能保持是因为在Activity中onSaveInstanceState(bundle)方法保存了mViewModelStore中	     	  *  的数据，Activity重新创建时在onCreate中把数据恢复的。
   */
  public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
        ViewModel viewModel = mViewModelStore.get(key);

        if (modelClass.isInstance(viewModel)) {
            if (mFactory instanceof OnRequeryFactory) {
                ((OnRequeryFactory) mFactory).onRequery(viewModel);
            }
            return (T) viewModel;
        } else {
            //noinspection StatementWithEmptyBody
            if (viewModel != null) {
                // TODO: log a warning.
            }
        }
        if (mFactory instanceof KeyedFactory) {
            viewModel = ((KeyedFactory) mFactory).create(key, modelClass);
        } else {
            viewModel = mFactory.create(modelClass);
        }
        mViewModelStore.put(key, viewModel);
        return (T) viewModel;
  }
  
  
  //创建无参的ViewModel实例
  public static class NewInstanceFactory implements Factory {
    ...
    public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
            //noinspection TryWithIdenticalCatches
            try {
                return modelClass.newInstance();
            } catch (InstantiationException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            } catch (IllegalAccessException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            }
        }
    ...
  }
  
  //创建AndroidViewModel对象工厂
  public static class AndroidViewModelFactory extends ViewModelProvider.NewInstanceFactory {
    	...
  		public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
            if (AndroidViewModel.class.isAssignableFrom(modelClass)) {
                //noinspection TryWithIdenticalCatches
                try {
                    return modelClass.getConstructor(Application.class).newInstance(mApplication);
                } catch (NoSuchMethodException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                } catch (IllegalAccessException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                } catch (InstantiationException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                } catch (InvocationTargetException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                }
            }
            return super.create(modelClass);
        }
    ...
  }
  ...
}
```



# 总结

ViewModel有两个功能：

1. 持有Activity或Fragment数据，在横屏切换时，通过Activity的生命周期机制，保存和恢复数据，解决Activity中的变量在横屏后数据丢失，自动帮我们实现了onSaveInstanceState(Bundle)中保存数据，onCreate(Bundle)中恢复数据的操作。Activity和Fragment之间的数据共享（Fragment可以通过传入与之邦定的Activity对象创建ViewModel，这样实现了Fragment和Activity共用Activity中的ViewModelStore，解决了Actiivty与Fragment之间直接传值的操作）。
2. ViewModel实现逻辑功能，解决Activity臃肿问题，只做管理操作，也解决了逻辑可以与其它Activity或Fragment共用。特殊需要注意的是，逻辑功能共用需要做好功能的抽离，通过继承或其它方法解决ViewModel变的代码臃肿的后果，变成万能的ViewModel。

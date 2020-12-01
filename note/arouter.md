# arouter

arouter是一个装饰模式，对外开放ARouter，平常调用通过ARouter使用。

1. 先添加一个注解

```java
// 在支持路由的页面上添加注解(必选)
// 这里的路径需要注意的是至少需要有两级，/xx/xx
@Route(path = "/test/activity")
public class YourActivity extend Activity {
    ...
}
```

2. 初始化SDK

   ```java
   // 在支持路由的页面上添加注解(必选)
   // 这里的路径需要注意的是至少需要有两级，/xx/xx
   @Route(path = "/test/activity")
   public class YourActivity extend Activity {
       ...
   }
   ```

3. 初始化SDK

   ```java
   if (isDebug()) {           // 这两行必须写在init之前，否则这些配置在init过程中将无效
       ARouter.openLog();     // 打印日志
       ARouter.openDebug();   // 开启调试模式(如果在InstantRun模式下运行，必须开启调试模式！线上版本需要关闭,否则有安全风险)
   }
   ARouter.init(mApplication); // 尽可能早，推荐在Application中初始化
   ```

4. 发起路由操作

```java
// 1. 应用内简单的跳转(通过URL跳转在'进阶用法'中)
ARouter.getInstance().build("/test/activity").navigation();

// 2. 跳转并携带参数
ARouter.getInstance().build("/test/1")
            .withLong("key1", 666L)
            .withString("key3", "888")
            .withObject("key4", new Test("Jack", "Rose"))
            .navigation();
```



上面是ARouter的基本用法，根据上面的用法，看下源码是怎么实现的。

ARouter类是对_ARouter类的代理，类中包含：

1. 初始化库：

```java
public static void init(Application application)
```

2. 开始测试的时候用的日志类的方法

3. 让ARouter使用我们自己的线程池：

   ```java
   public static synchronized void setExecutor(ThreadPoolExecutor tpe)
   ```

4. 注入调用初始化数据

   ```java
   public void inject(Object thiz)
   ```

5. 构建参数

   ```java
   public Postcard build(String path)
   ```

   ```java
   public Postcard build(Uri url)
   ```

6. 启动导航

   ```java
   public <T> T navigation(Class<? extends T> service)
   ```

   ```java
   public Object navigation(Context mContext, Postcard postcard, int requestCode, NavigationCallback callback)
   ```



上面就是ARouter为我们提供的方法，我们根据一相启动新Activity的流程看下源码的流程，

首先我们看下

```java
/**
 * Build the roadmap, draw a postcard.
 *
 * @param path Where you go.
 */
public Postcard build(String path) {
    return _ARouter.getInstance().build(path);
}
```

通过参数path，读取到要跳转的路径信息。

```java
protected Postcard build(String path) {
    if (TextUtils.isEmpty(path)) {
        throw new HandlerException(Consts.TAG + "Parameter is invalid!");
    } else {
        PathReplaceService pService = ARouter.getInstance().navigation(PathReplaceService.class);
        if (null != pService) {
            path = pService.forString(path);
        }
        return build(path, extractGroup(path), true);
    }
}
```

简单的验证path的合法性，pService我们先忽略，通过extractGroup(path)，获取path中的路径信息。

```java
/**
 * 提取路径信息
 */
private String extractGroup(String path) {
    if (TextUtils.isEmpty(path) || !path.startsWith("/")) {
        throw new HandlerException(Consts.TAG + "Extract the default group failed, the path must be start with '/' and contain more than 2 '/'!");
    }

    try {
      //截取路径中的第一个目录为group，由此我们得出，无论path中有多少线目录，它只提取第一级最为Group
        String defaultGroup = path.substring(1, path.indexOf("/", 1));
        if (TextUtils.isEmpty(defaultGroup)) {
            throw new HandlerException(Consts.TAG + "Extract the default group failed! There's nothing between 2 '/'!");
        } else {
            return defaultGroup;
        }
    } catch (Exception e) {
        logger.warning(Consts.TAG, "Failed to extract default group! " + e.getMessage());
        return null;
    }
}
```

再向下看

```java
/**
 * 通过 path and group 构建postcard
 * postcard是存放参数的信息载体
 */
protected Postcard build(String path, String group, Boolean afterReplace) {
    if (TextUtils.isEmpty(path) || TextUtils.isEmpty(group)) {
        throw new HandlerException(Consts.TAG + "Parameter is invalid!");
    } else {
      	//这个方法是不会进来的，这是为了兼容以前版本。
        if (!afterReplace) {
            PathReplaceService pService = ARouter.getInstance().navigation(PathReplaceService.class);
            if (null != pService) {
                path = pService.forString(path);
            }
        }

        return new Postcard(path, group);      	//创建一个Postcard
    }
}
```



Postcard是包含路线图的容器。Postcard继承于RouteMeta，RouteMeta是路由信息的基础类，



```java
public class RouteMeta {
    private RouteType type;         // 路由的类型：ACTIVITY、SERVICE、PROVIDER等
    private Element rawType;        // 路由的真实类型
    private Class<?> destination;   // 跳转的目的地
    private String path;            // 路径
    private String group;           // 分组
    private int priority = -1;      // 优先线，越小越高
    private int extra;              // 扩展
    private Map<String, Integer> paramsType;  // 参数
    private String name;						

    private Map<String, Autowired> injectConfig;  // 注解缓存
```

```java
public final class Postcard extends RouteMeta {
    // Base
    private Uri uri;
    private Object tag;             // A tag prepare for some thing wrong.
    private Bundle mBundle;         // Data to transform
    private int flags = -1;         // flags是发广播的时候使用的，例：FLAG_ACTIVITY_*
    private int timeout = 300;      // Navigation timeout, TimeUnit.Second
    private IProvider provider;     // It will be set value, if this postcard was provider.
    private boolean greenChannel;
    private SerializationService serializationService;

    // Animation
    private Bundle optionsCompat;    // The transition animation of activity
    private int enterAnim = -1;
    private int exitAnim = -1;
```



上面这两个是整个ARouter的信息包含类了。由此可知，我们在使用传参的都是放在Postcard里面，还有动画效果。

这时我们所需要的路由数据构建完成，也就是我们平常所用的Intent的数据。

接下来就是我们通过这此信息来调用导航，在Postcard中有不同的navigation方法，调用此方法，会调用

```java
ARouter.getInstance().navigation(...)
```

接下下调用

```java
/**
     * Use router navigation.
     *
     * @param context     Activity or null.
     * @param postcard    Route metas
     * @param requestCode RequestCode
     * @param callback    cb
     */
    protected Object navigation(final Context context, final Postcard postcard, final int requestCode, final NavigationCallback callback) {
      	//数据预处理服务，可以通过PretreatmentService取消跳转或添加跳转统计、记录等。
        PretreatmentService pretreatmentService = ARouter.getInstance().navigation(PretreatmentService.class);
        if (null != pretreatmentService && !pretreatmentService.onPretreatment(context, postcard)) {
            // Pretreatment failed, navigation canceled.
            return null;
        }

        try {
          //完善postcard信息
            LogisticsCenter.completion(postcard);
        } catch (NoRouteFoundException ex) {
            logger.warning(Consts.TAG, ex.getMessage());

            if (debuggable()) {
                // Show friendly tips for user.
                runInMainThread(new Runnable() {
                    @Override
                    public void run() {
                        Toast.makeText(mContext, "There's no route matched!\n" +
                                " Path = [" + postcard.getPath() + "]\n" +
                                " Group = [" + postcard.getGroup() + "]", Toast.LENGTH_LONG).show();
                    }
                });
            }

            if (null != callback) {
                callback.onLost(postcard);
            } else {
                // No callback for this invoke, then we use the global degrade service.
                DegradeService degradeService = ARouter.getInstance().navigation(DegradeService.class);
                if (null != degradeService) {
                    degradeService.onLost(context, postcard);
                }
            }

            return null;
        }
				//需要回调
        if (null != callback) {
            callback.onFound(postcard);
        }
					//是否跳过所有拦截器
        if (!postcard.isGreenChannel()) {   // It must be run in async thread, maybe interceptor cost too mush time made ANR.
            interceptorService.doInterceptions(postcard, new InterceptorCallback() {
                /**
                 * Continue process
                 *
                 * @param postcard route meta
                 */
                @Override
                public void onContinue(Postcard postcard) {
                    _navigation(context, postcard, requestCode, callback);
                }

                /**
                 * Interrupt process, pipeline will be destory when this method called.
                 *
                 * @param exception Reson of interrupt.
                 */
                @Override
                public void onInterrupt(Throwable exception) {
                    if (null != callback) {
                        callback.onInterrupt(postcard);
                    }

                    logger.info(Consts.TAG, "Navigation failed, termination by interceptor : " + exception.getMessage());
                }
            });
        } else {
            return _navigation(context, postcard, requestCode, callback);
        }

        return null;
    }
```

下面就是我们常用的了。

```java
private Object _navigation(final Context context, final Postcard postcard, final int requestCode, final NavigationCallback callback) {
    final Context currentContext = null == context ? mContext : context;

    switch (postcard.getType()) {
        case ACTIVITY:
            // Build intent
            final Intent intent = new Intent(currentContext, postcard.getDestination());
            intent.putExtras(postcard.getExtras());

            // Set flags.
            int flags = postcard.getFlags();
            if (-1 != flags) {
                intent.setFlags(flags);
            } else if (!(currentContext instanceof Activity)) {    // Non activity, need less one flag.
                intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            }

            // Set Actions
            String action = postcard.getAction();
            if (!TextUtils.isEmpty(action)) {
                intent.setAction(action);
            }

            // Navigation in main looper.
            runInMainThread(new Runnable() {
                @Override
                public void run() {
                    startActivity(requestCode, currentContext, intent, postcard, callback);
                }
            });

            break;
        case PROVIDER:
            return postcard.getProvider();
        case BOARDCAST:
        case CONTENT_PROVIDER:
        case FRAGMENT:
            Class fragmentMeta = postcard.getDestination();
            try {
                Object instance = fragmentMeta.getConstructor().newInstance();
                if (instance instanceof Fragment) {
                    ((Fragment) instance).setArguments(postcard.getExtras());
                } else if (instance instanceof android.support.v4.app.Fragment) {
                    ((android.support.v4.app.Fragment) instance).setArguments(postcard.getExtras());
                }

                return instance;
            } catch (Exception ex) {
                logger.error(Consts.TAG, "Fetch fragment instance error, " + TextUtils.formatStackTrace(ex.getStackTrace()));
            }
        case METHOD:
        case SERVICE:
        default:
            return null;
    }

    return null;
}
```



以上就是我们正常整个流程。

我们添加的注解在编译后会成生相应的文件，会为分组信息分成一个分组类

```java
/**
 * DO NOT EDIT THIS FILE!!! IT WAS GENERATED BY AROUTER. */
public class ARouter$$Root$$app implements IRouteRoot {
  @Override
  public void loadInto(Map<String, Class<? extends IRouteGroup>> routes) {
    routes.put("home", ARouter$$Group$$home.class);
    routes.put("user", ARouter$$Group$$user.class);
  }
}
```

上面是自动生成的文件，其中`home` 、`user` 为分组名，后面的Value类，为对应的分组内的成员

```java
/**
 * DO NOT EDIT THIS FILE!!! IT WAS GENERATED BY AROUTER. */
public class ARouter$$Group$$home implements IRouteGroup {
  @Override
  public void loadInto(Map<String, RouteMeta> atlas) {
    atlas.put("/home/mine/activity", RouteMeta.build(RouteType.ACTIVITY, MineAct.class, "/home/mine/activity", "home", null, -1, -2147483648));
  }
}
```

RouteMeta为基本信息类，这里它根据信息帮我们自动成生了跳转信息。

当我们定义了Interceptor时，也同样会为我们生成相应的信息

```java
/**
 * DO NOT EDIT THIS FILE!!! IT WAS GENERATED BY AROUTER. */
public class ARouter$$Interceptors$$app implements IInterceptorGroup {
  @Override
  public void loadInto(Map<Integer, Class<? extends IInterceptor>> interceptors) {
    interceptors.put(1, UserInterceptorService.class);
    interceptors.put(2, VIPInterceptorService.class);
  }
}
```

**注意这里是按照拦截器的级别来存储的，如果级别重复会覆盖。**

```java
/**
 * DO NOT EDIT THIS FILE!!! IT WAS GENERATED BY AROUTER. */
public class ARouter$$Providers$$app implements IProviderGroup {
  @Override
  public void loadInto(Map<String, RouteMeta> providers) {
    providers.put("com.alibaba.android.arouter.facade.service.DegradeService", RouteMeta.build(RouteType.PROVIDER, DegradeServiceImpl.class, "/user/degrade", "user", null, -1, -2147483648));
  }
}
```



LogisticsCenter类，在初始化的时候会把我们路由所需要的Route注解信息加载到内存中。这里会有两种方式加载，一种是自动化插件注册，另一种是每次启动的时候注册，第二种方式会拖慢应用启动速度。推荐使用第一种。



拦截器在每次程序启动的初始化的时候会

#### 自定义全局降级策略(DegradeService)

在使用的时候初始化，在调用导航跳转的时候，如果未使用NavigationCallback会调用此服务。此服务只有自定义一个，多个会覆盖。

#### 处理跳转结果

NavigationCallback是跳转导航前的回调，如果没有设置NavigationCallback回调，路由跳转错误会由DegradeService处理。

```java
public interface NavigationCallback {

    /**
     * Callback when find the destination.
     *
     * @param postcard meta
     */
    void onFound(Postcard postcard);

    /**
     * Callback after lose your way.
     *
     * @param postcard meta
     */
    void onLost(Postcard postcard);

    /**
     * Callback after navigation.
     *
     * @param postcard meta
     */
    void onArrival(Postcard postcard);

    /**
     * Callback on interrupt.
     *	拦截器抛异常时调用
     * @param postcard meta
     */
    void onInterrupt(Postcard postcard);
}
```



#### 为目标页面声明更多信息

```java
// 我们经常需要在目标页面中配置一些属性，比方说"是否需要登陆"之类的
// 可以通过 Route 注解中的 extras 属性进行扩展，这个属性是一个 int值，换句话说，单个int有4字节，也就是32位，可以配置32个开关
// 剩下的可以自行发挥，通过字节操作可以标识32个开关，通过开关标记目标页面的一些属性，在拦截器中可以拿到这个标记进行业务逻辑判断
@Route(path = "/test/activity", extras = Consts.XXXX)
```


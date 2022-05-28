# DataBinding

使用DataBinding时，只需要在`build.gradle`中添加已下配置就可以自由使用了

```groovy
android {
        ...
        viewBinding {
            enabled = true
        }
    }
    
```

DataBinding分为两部分：视图邦定、数据绑定；

## 视图邦定

如果启动dataBinding功能，我们创建的 XML 布局文件会自动生成对应的绑定类，命名规则：将 XML 文件的名称转换为驼峰式大小写，并在末尾添加“Binding”一词。

自动生成的绑定类生成目录：

<img src="http://qike.hilary.top/uPic/image-20220523223355392.png" alt="image-20220523223355392" style="zoom:50%;" />

自动生成类代码

```java
public final class ActivityMainBinding implements ViewBinding {
  @NonNull
  private final ConstraintLayout rootView;
	//根据 XML 布局页面中的ID名称生成变量
  @NonNull
  public final TextView textView;

  private ActivityMainBinding(@NonNull ConstraintLayout rootView, @NonNull TextView textView) {
    this.rootView = rootView;
    this.textView = textView;
  }

  //返回 XML 根视图的引用
  @Override
  @NonNull
  public ConstraintLayout getRoot() {
    return rootView;
  }

  //创建Binding对象，一般用于Activity中，直接通过Activity中的layoutInflater()传参给此方法
  @NonNull
  public static ActivityMainBinding inflate(@NonNull LayoutInflater inflater) {
    return inflate(inflater, null, false);
  }

  //一般用于Fragment、Adapter等非Activity初始化Binding对象的地方
  @NonNull
  public static ActivityMainBinding inflate(@NonNull LayoutInflater inflater,
      @Nullable ViewGroup parent, boolean attachToParent) {
    View root = inflater.inflate(R.layout.activity_main, parent, false);
    if (attachToParent) {
      parent.addView(root);
    }
    return bind(root);
  }

  //对布局页面带有ID的控件进行初始化
  @NonNull
  public static ActivityMainBinding bind(@NonNull View rootView) {
    int id;
    missingId: {
      id = R.id.textView;
      TextView textView = ViewBindings.findChildViewById(rootView, id);
      if (textView == null) {
        break missingId;
      }

      return new ActivityMainBinding((ConstraintLayout) rootView, textView);
    }
    String missingId = rootView.getResources().getResourceName(id);
    throw new NullPointerException("Missing required view with ID: ".concat(missingId));
  }
}
```

Activity中初始化binding对象

```java
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        binding.textView.text = "1111"
    }
}
```

Fragment中初始化binding对象

```java
 private var _binding: ActivityMainBinding? = null
    // 此对象只能在 onCreateView 和 onDestroyView 中间使用
    // 
    private val binding get() = _binding!!

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        _binding = ActivityMainBinding.inflate(inflater, container, false)
        val view = binding.root
        return view
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
```

视图绑定是数据绑定的轻量级实现，视图绑定不需要修改 XML 文件，视图绑定帮我们解决控件的初始化，也在编译时检查代码与布局之间的兼容，如果有问题会编译失败。

如果不需要自动生成 Binding 对象，在布局根目录添加`tools:viewBindingIgnore="true"`

Binding更多的功能是在数据绑定库中，建议在项目开发中同时使用视图绑定和数据绑定功能。

## 数据绑定库

数据库绑定库借助布局中的绑定组件，可以把Activity中的许多框架调用移除，在布局文件中实现同样的功能。

要使用`dataBinding`，只需要像使用viewBinding一样，在`buidle.gradle`中添加配置

```groovy
android {
        ...
        dataBinding {
            enabled = true
        }
    }
    
```

布局文件修改，布局文件最外层使用`layout`包裹，文件分为两部分：`data`、`view`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
      	<!-- 定义变量 -->
        <variable
            name="user"
            type="com.hilary.binding.demo.User" />
    </data>
    <androidx.constraintlayout.widget.ConstraintLayout
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <!-- 通过设置user中的值，来改变textView显示 -->
        <TextView
            android:id="@+id/textView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.firstName}"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="200dp"
            android:background="#33ff0000"
            app:layout_constraintBottom_toBottomOf="parent"
            tools:viewBindingIgnore="true">
            <TextView
                android:id="@+id/subTextView"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@{user.lastName}"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintBottom_toBottomOf="parent"/>
        </androidx.constraintlayout.widget.ConstraintLayout>

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>

```

使用`User`为页面赋值

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        binding.user = User("hilary", "chen")
    }
}
```

<img src="http://qike.hilary.top/uPic/image-20220524094841696.png" alt="image-20220524094841696" style="zoom: 50%;" />


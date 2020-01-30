# Intent

#### # 显式Intent和隐式Intent的区别？

**显式Intent**

举一个例子：

```java
Intent intent = new Intent(FirstActivity.this,SecondActivity.class);
startActivity(intent);
```

像这种目的很明确，我就是想在`FirstActivity`中启动`SecondActivity`，这种启动`Activity`或者`Service`的方式称为`显示Intent`。

**隐式Intent**

相较于显式Intent，隐式Intent则含蓄了很多，它并不明确指定我们想要指定哪一个Activity，而是指定一系列的`action`或者`category`等信息，然后交由系统去分析这个Intent并帮我们找出合适的活动去启动。

举一个例子：

```java
Intent intent = new Intent("android.intent.action.MAIN");
startActivity(intent);
```

什么是`android.intent.action.MAIN`?

这个是我们在`AndroidManifest.xml`中配置的：

```xml
<application
    <!-- 省略 -->
    android:theme="@style/AppTheme">
  	<!-- 省略 -->
    <activity
        android:name=".MainActivity"
        android:launchMode="standard">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>
</application>
```

#### # Intent传递数据大小的限制？如何突破限制？

`Intent`传递数据大小的限制大概在1MB左右，因为Binder传递数据是1MB

**突破方法**

1. 使用FileProvider或者数据库
2. 使用静态类全局共享

#### # PendingIntent

放在RemoteView里面讲吧




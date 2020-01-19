# Android四大组件

## 一、Activity

#### # 解释一下Activity的生命周期？

- `onCreate`：Activity正在被创建
- `onRestart`：Activity正在被重新启动
- `onStart`：Activity正在被启动，Activity可见但未出现在前台
- `onResume`：Activity可见，并出现在前台开始活动
- `onPause`：Activity正在停止
- `onStop`：Activity将要停止
- `onDestroy`：Activity将要销毁

#### # 说说Activity的生命周期过程？

一个Activity的正常生命周期：

`onCreate` - `onStart` - `onResume` - `onPause` - `onStop` - `onDestroy`

**Activity跳转**

当Activity A跳转到Activity B，这个时候A和B的生命周期为：

`A-onCreate` - `A-onStart` - `A-onResume` - `A-onPause` - `B-onCreate` - `B-onStart` - `B-onResume` - `A-onStop` 

接着finish掉Activity B，这个时候的生命周期为：

`B-onPause` - `A-onRestart` - `A-onStart` - `A-onResume` - `B-onStop` - `onDestroy`

**App切回到后台或者回到主屏幕**

切换到后台：

`onCreate` - `onStart` - `onResume` - `onPause` 

再返回当前App：

`onRestart` - `onStart` - `onResume` 

#### # 一些特殊情况下的Activity的生命周期？

**横竖屏切换**

通常情况下，如果发生了横竖屏切换，当前的Activity会销毁并重新创建，这个时候的生命周期为：

`onCreate` - `onStart` - `onResume` - `onPause` - `onSaveInstanceState` - `onStop` - `onDestroy` - `onCreate` - `onStart` - `onRestoreInstanceState` - `OnResume`

其中，`#onSaveInstanceState`和`#onRestoreInstanceState`可以用来存储和恢复横竖屏切换前界面的信息。

**内存不足**

如果内存不足，会优先杀死优先级别较低的Activity，优先级别的定义，从高到低：

1. 前台Activity：当前与用户交互的Activity
2. 可见非前台Activity：当前Activity弹出对话框的时候
3. 后台Activity

这些Activity被杀死的时候，也需要走`#onSaveInstanceState`和`#onRestoreInstanceState`，也就是说其数据存储和恢复的过程跟上一个过程一样。

#### # Fragment的生命周期？

通常Fragment的生命周期是：

`onAttach` - `onCreate` - `onCreateView` - `onActivityCreate` - `onStart` - `onResume` - `onPause` - `onStop` - `onDestroyView` - `onDestroy` - `onDetach`

####  # Activity中包含Fragment时的生命周期？

假设Activity为A，Fragment为F，则启动时的生命周期为：

`A-onCreate`- `F-onAttach` - `F-onCreate` - `F-onViewCreate` - `F-onActivityCreate` - `F-onStart` - `A-onStart` - `A-onResume` - `F-onResume`

退出时的生命周期为：

`F-onPause` - `A-onPause` - `F-onStop` - `A-onStop` - `F-onDestroyView` - `F-onDestroy` - `F-onDetach` - `onDestroy`

#### # Activity的四大启动模式？

Activity的启动模式一般在AndroidManifest的Activity中的launchMode属性设置，常用的Activity启动模式有四种：

- Standard：标准模式。如果不设置，默认就是标准模式，每次都会重新创建一个新的实例
- SingleTop：栈顶复用。如果Activity处于活动栈的栈顶，那么再次生成一个新的Activity不会被重新创建，`#onNewIntent`方法也会被触发。
- SingleTask：站内复用，假设一个Activity栈为：ABCD，D为栈顶，这个时候再次跳转到Activity A，则不会创建新的Activity A，Activity A的`#onNewIntent`会被重新调用，Activity BCD则会出栈，所以此时的Activity栈为：A。
- SingleInstance：单实例模式，顾名思义，一个栈中只会有一个实例，如果打开SingleInstance的Activity，则会重新创建一个栈，并在该栈中只会创建一个新的实例。

推荐阅读Blog：[《一篇文章搞懂 Activity 启动模式》](https://juejin.im/post/5c6fce04f265da2d943f6641#comment)，图不错，有助于理解，不过文章的最后一部分不对，至少验证的不对~

## 二、Service

`Service`是实现程序后台运行的解决方案，它非常适合去执行那些不需要和用户交互而且还要求长期执行的任务。每个服务都只会存在一个实例。

#### # 讲一下Service的使用方式和生命周期？

使用`Service`的方式有两种：

**第一种**

使用步骤：

1. 继承`Service`
2. `AndroidManifest`注册

3. 分别使用`#startService`和`#stopService`启动和停止服务

第一种方式的生命周期：

`onCreate` - `onStartCommond` - `onDestroy`

**第二种**

使用步骤：

1. 继承`Service`
2. `AndroidManifest`注册
3. 实现`Binder`类
4. 在`Service`的`#onBind`方法中返回上一步中定义的`Binder`类
5. 创建`ServiceConnection`调用实现的`Binder`类中的方法
6. 分别使用`#bindService`和`#unBindService`启动和停止服务

第二种方式的生命周期：

`onCreate` - `onBind` - `onUnBind` - `onDestroy`

#### # 聊聊IntentService

虽说Service的使用方式并不复杂，但是总有程序员忘记开启线程，或者忘记调用`stopServie`方法，为了解决以上的两个尴尬问题，Android专门提供了`IntentService`类，从而提供一个异步的、自动停止的服务。

`IntentService`的特征是：

- 建立独立`Worker`线程处理所有的`Intent`请求
- 建立独显`Worker`线程处理`#onHandleIntent`的代码，不需要处理多线程问题
- 请求处理完毕，`IntentService`会自动停止，无需调用`#stopSelf`方法停止
- 为`Service`的`#onBInd`方法提供默认实现，返回为null
- 为`Service`的`#onStartCommond`提供默认实现，将`Intent`请求加入队列

## 三、BroadcastReceiver

#### # 解释一下Broadcast Receiver的生命周期？

广播接收器的生命周期其实很短：

调用接收对象 - `onReceive` - 结束

#### # 广播的类型有哪些？

Android中的广播主要可以分为两种：

1. **标准广播**：完全异步执行的广播，在广播放出后，所有的接收器几乎同时受到广播，效率虽高，但无法截断。
2. **有序广播**：同步执行的广播，同一时间，只有一个广播接收器能够收到这条广播消息，优先级别高的广播接收器会先收到广播，且前面的广播接收器截断广播，后面的广播接收器就无法收到广播。

#### # 注册的广播方式？

按照注册类型划分，可以分为动态注册和静态注册：

**动态注册**

1. 灵活性较高，`Activity`启动时注册，`Activity`注销时注销

**静态注册**

1. 在`AndroidManifest`中配置
2. 程序未启动时就能收到广播

同样情况下，动态广播接收器的优先级高于静态的广播接收器

#### # 全局广播和本地广播？

**全局广播**

发出的广播可以被任何程序接收到，并且自己也可以接受来自于其他应用程序的广播

**本地广播**

发出的广播只能够应用程序的内部进行传递

## 四、ContentProvider

`ContentProvider`主要用于在不同的应用程序之间实现数据共享功能，允许一个程序访问另外一个程序的同时，还能保证访问数据的安全性。`CotentProvider`的底层实现是`Binder`，所以它天生就适合于跨进程通信，对于同步问题，Android也提供了很好的封装。

#### # ContentProvider初始化时机？

介于`Application`的`#attachBaseContext`和`#onCreate`方法之间。

#### # ContentProvider的同步异步问题？

`ContentProvider`的`#onCreate`方法由系统回调运行在主线程中，而`#query`、`#update` 、`#insert`、`#delete`四大方法存在多线程情况的，因此该方法内部需要做好线程同步。
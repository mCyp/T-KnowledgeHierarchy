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

其中，`onSaveInstanceState`和`onRestoreInstanceState`可以用来存储和恢复横竖屏切换前界面的信息。

**内存不足**

如果内存不足，会优先杀死优先级别较低的Activity，优先级别的定义，从高到低：

1. 前台Activity：当前与用户交互的Activity
2. 可见非前台Activity：当前Activity弹出对话框的时候
3. 后台Activity

这些Activity被杀死的时候，也需要走`onSaveInstanceState`和`onRestoreInstanceState`，也就是说其数据存储和恢复的过程跟上一个过程一样。

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
- SingleTop：栈顶复用。如果Activity处于活动栈的栈顶，那么再次生成一个新的Activity不会被重新创建，`onNewIntent`方法也会被触发。
- SingleTask：站内复用，假设一个Activity栈为：ABCD，D为栈顶，这个时候再次跳转到Activity A，则不会创建新的Activity A，Activity A的`onNewIntent`会被重新调用，Activity BCD则会出栈，所以此时的Activity栈为：A。
- SingleInstance：单实例模式，顾名思义，一个栈中只会有一个实例，如果打开SingleInstance的Activity，则会重新创建一个栈，并在该栈中只会创建一个新的实例。

推荐阅读Blog：[《一篇文章搞懂 Activity 启动模式》](https://juejin.im/post/5c6fce04f265da2d943f6641#comment)，图不错，有助于理解，不过文章的最后一部分不对，至少验证的不对~


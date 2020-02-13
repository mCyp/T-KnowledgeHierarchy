# Window

Window表示一个窗口的概念。除此以外，我们还需了解：

- Window是一个抽象类，具体的实现类是PhotoWindow。
- Window是Android所有View的载体。
- Window的具体实现位于WMS(Window Manger Service)中，所以，Window Manger和WMS的交互属于IPC，基于Binder。

## Window的一些知识点

#### # Window分类？

Window可以分为三种类型：

- 应用Window：对应着Activity,
- 子Window：对应着Dialog
- 系统Window：状态栏、Toast

## Window的内部机制

笔记就简单一点，毕竟不是源码分析的文章，所以就不涉及源码了。

#### # WindowManger提供了哪些功能？

这个可以参见`ViewManger`这个接口，提供了：

-  添加Window
- 删除Window
- 更新Window

虽然从方法名字上看都是针对Window，但是实际上，都是操作的View。

Window是一个抽象的概念，每一个`Window`都对应着一个`View`和一个`ViewRootImpl`，`Window`和`View`通过`ViewRootImpl`建立联系，因此`Window`并不是实际存在的，它是以`View`的形式存在。

#### # 添加Window的过程？

Window的添加过程是需要借助`WindowManager`，`WindowManger`的实现类是`WindowMangerImpl`，但是进入`WindowManger`的源码，发现它使用了桥接模式，真实的处理类是`WindowManagerGlobal`，再深入一点，发现它的过程其实很简单：

1. 检查`Window`中的一些参数是否合法，比如`Window`中是否有`View`。
2. 创建`ViewRootImpl`，然后将`View`添加进`ViewRootImpl`。
3. 调用`ViewRootImpl`中的`setView`方法完成界面更新。
4. 通过`WindowSession`调用WMS(Window Manger Service)完成`Window`的添加。

#### # 删除Window的过程？

与之前的情况类似，删除`Window`需要借助`ViewRootImpl`来删除视图，这里的过程大概是：

1. 查找等待删除的`View`的索引。

2. 同步删除就是直接删除，一般不会用到，我们讨论异步删除，会将需要删除的`View`放入`ViewRootImpl`中的等待删除队列，再利用`Handler`发送消息，接收消息的时候再删除。

真正删除的过程会：

1. 进行垃圾回收的工作。
2. 通知WMS进行Window的删除。
3. 回调View中的一些方法。
4.  对`WindwoMangerGlobal`中的一些参数进行更新。

#### # 更新Window的过程？

我原以为Window的更新不会触发IPC，这个观点是错误的，除了视图的重新测量、定位和绘制外，`ViewRootImpl`还会通过`WindowSession`更新`Window`视图。

## 具体分析

#### # Acitivity\Dialog\Toast的Window创建的过程？

其实`Activity`和`Dialog`的`Window`的创建过程类似，`Toast`的不同，因为Activity、Dialog和Toast分别属于应用Window、子Window和系统Window。具体的下次有机会分析。

- [ ] 分析
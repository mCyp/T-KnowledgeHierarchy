# Activity\Window\ViewRoot\DecorView关系

## 整体介绍

在接收关系之前，先大概了解这四位的作用

#### # Activity?

Activity，Android四大组件，虽然它是四大组件中唯一可见的，但是我们不可以认为它就是一个视图容器，实际上它确实也不是的，它不负责控制视图，它只负责控制生命周期和事件处理。

#### # Window?

Window，表示一个窗口的概念，是Android中所有视图的载体。真正控制视图的类是Window，具体的实现类是`PhoneWindow`。

#### # DecorView?

DecorView，顶级视图，从结构上来看，你可以理解Android每一个`Window`下面都有一个视图树，即父视图下面有子视图，子视图下面可能有子子视图，当然也可能没有，而DecorView就是最上层的视图。它的上层是`Window`;

从内容上来看，它是一个`FrameLayout`，里面包含了标题栏和内容栏，所以开始一个新的项目，使用默认主题，尽管你的布局文件设置了`match_parent`，但是标题栏还会出现，就跟下图一样。

#### # ViewRoot?

ViewRoot，从名字上看，它看上去跟`DecorView`的一样，其实不然，`ViewRoot`对应的类是`ViewRootImpl`，它是连接`WindowManager`和`DecorView`的桥梁，`View`的三大流程都是通过`ViewRoot`完成的。

除此以外，Android中所有触屏事件、刷新事件和按键事件都是通过`ViewRoot`转发的。

所以以上四位的关系是：

![Activity等的关系](https://teaof-konwleadge-1255982134.cos.ap-shanghai.myqcloud.com/blog/Android/View/Window%E5%92%8CActivity%E7%AD%89%E7%9A%84%E5%85%B3%E7%B3%BB.png)

通过以上了解可以知道：**Activity就像个控制器，不负责视图部分。Window像个承载器，装着内部视图。DecorView就是个顶层视图，是所有View的最外层布局。ViewRoot像个连接器，负责沟通，通过硬件的感知来通知视图，进行用户之间的交互。**

来自【Ruheng】，文章地址：https://www.jianshu.com/p/8766babc40e0
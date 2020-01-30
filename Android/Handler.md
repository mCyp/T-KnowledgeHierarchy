# Handler

`Handler`是在Android中一个很有意思的东西，借助`Handler`可以轻松的将任务切换到创建它的线程中去，不过在很多人来看，它就是用来更新UI的。

想要弄懂`Handler`机制，阅读《Android开发艺术探索》是一个很好的选择。

#### # ThreadLocal是什么？

`ThreadLocal`是一个线程数据存储类，通过它可以在指定的线程中存储数据，数据存储成功后，仅可以在指定线程中取出数据。

查看了`JDK 8`，发现原理是每一个线程`Thread`中都有一个`ThreadLocalMap`，`ThreadLocal`里面获取当前线程，进而获取当前线程中的`ThreadLocalMap`，实质上`ThreadLocalMap`是一个类型是`ThreadLocalMap.Entry`数组，接着找到与当前`ThreadLocal`匹配的`ThreadLocal`成员，进而设置或者取值。

#### # MessageQueue是什么？

消息队列`MessageQueue`主要包含两个操作：

- `MessageQueue#enqueueMessage(Message msg)`：插入消息
- `MessageQueue#next()`：获取消息

实际上它是通过一个单链表的数据结构来维护消息列表。

#### # Looper是什么？

`Looper`在消息机制中扮演者循环的角色，具体来说就是不停地从`MessageQueue`读取新的消息，如果有新的消息就立刻处理，没有新的消息就阻塞在那里。

#### # Looper的原理？

`Looper`最关键的方法是`Looper#loop()`，这是一个静态方法，是`Looper`扮演循环角色的关键：

```java
public static void loop() {
	final Looper me = myLooper();
	if (me == null) {
		throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
	}
	final MessageQueue queue = me.mQueue;
	// ...
	for (;;) {
		Message msg = queue.next();
		// might block
		if (msg == null) {
			// No message indicates that the message queue is quitting.
			return;
		}
		// This must be in a local variable, in case a UI event sets the logger
		// ... 消费消息
		try {
			msg.target.dispatchMessage(msg);
			dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
		}
		finally {
			// ...
		}
		// ...
		msg.recycleUnchecked();
	}
}
```

主要的功能就是：

1. 循环消息队列
2. 取出消息，没有消息的情况下可能会阻塞
3. 得到消息的情况下就消费消息

可以看出，`Looper#loop()`工作在什么线程，消费消息就会发生在那个线程。

#### # Looper如何停止？

调用了`Looper.loop()`的以后，`Looper`也是可以停止的，主要分为两个方法：

- `Looper#quit()`：直接退出
- `Looper#quitSafely()`：当前消息处理完毕后安全退出

#### # 子线程如何使用Hanlder?

`Looper`源码中已经给了我们答案：

```java
class LooperThread extends Thread {
	public Handler mHandler;
	public void run() {
		Looper.prepare();
		mHandler = new Handler() {
			public void handleMessage(Message msg) {
				// process incoming messages here
			}
		};
		Looper.loop();
	}
}
```

再深入一下，如果不使用`Looper.prepare()`会发生什么？

因为子线程中不会默认创建`Looper`，所以需要`Looper.prepare()`创建当前子线程的`Looper`，在`Handler`的构造函数中，如果当前线程没有`Looper`，会抛出异常：

```java
public Handler(Callback callback, Boolean async) {
	// ...
	mLooper = Looper.myLooper();
	if (mLooper == null) {
		throw new RuntimeException(
		                "Can't create handler inside thread " + Thread.currentThread()
		                        + " that has not called Looper.prepare()");
	}
	mQueue = mLooper.mQueue;
	mCallback = callback;
	mAsynchronous = async;
}
```

这也解释了为什么没有`Looper`的子线程创建`Handler`会发生异常。

最后一点，即使创建了`Looper`，也要在调用`Looper.loop()`才会开始工作。

#### # Handler原理是什么？

**第一步 创建发送消息**

利用`Handler`发送消息的入口通常有两种：

1. 创建消息`Message`发送
2. 利用`Handler#post(Runnable r)`

其实这两个消息的本质都是一样的，都是创建一个`Message`。

```java
public final Boolean post(Runnable r)
{
	return  sendMessageDelayed(getPostMessage(r), 0);
}
```

看一眼`Handler#getPostMessage`方法你就明白了：

```java
private static Message getPostMessage(Runnable r) {
	Message m = Message.obtain();
	m.callback = r;
	return m;
}
```

既然都是使用的`Message`，在发送消息的方法里，按照顺序，**依次**调用：

- `Handler#sendMessageDelayed`
- `Handler#sendMessageAtTime`：这个方法会获取当前的消息队列`MessageQueue`，接着讲这个`MessageQueue`作为参数传递到下一个方法`Handler#enqueueMessage`

从方法名称你就可以猜到，这是调用了消息队列的插入消息的方法`MessageQueue#enqueueMessage`：

```java
private Boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
	msg.target = this;
	if (mAsynchronous) {
		msg.setAsynchronous(true);
	}
	return queue.enqueueMessage(msg, uptimeMillis);
}
```

**第二步 Looper循环获取消息**

在讲解**Looper的原理是什么**？中我已经给出了源码，这里就不再介绍源码，我觉得第二步是`Handler`能够切换线程的原因。

因为在线程中，我们一般会调用`Looper.prepare()`创建当前线程的`Looper`，接着差不多就可以调用`Looper.loop()`，因为消费消息的方法发生在该方法中，所以该方法发生在哪个线程，消费信息就发生在哪个线程，这也就是`Handler`可以切换线程的原因。

**第三步 Handler处理消息**

在上面的`Looper`的介绍中，我们知道在`Looper#loop()`中调用了

```java
msg.target.dispatchMessage(msg);
```

`msg.target`就是发送消息的`Handler`，来看看`Handler#dispatchMessage(Message msg)`发生了什么？

```java
public void dispatchMessage(Message msg) {
	if (msg.callback != null) {
      	// msg.callback由Handler#getPostMessage(Runnable r)创建
      	// 入口是Handler#postRunnable这类方法
		handleCallback(msg);
	} else {
      	// 如果不通过Handler#postRunnable方法
		if (mCallback != null) {
          	// mCallback通过Handler的构造函数设置
			if (mCallback.handleMessage(msg)) {
				return;
			}
		}
      	// 如果构造函数没有设置mCallback
      	// 可以通过创建Handler对象的时候复写Handler#handleMessage方法
		handleMessage(msg);
	}
}
```

#### 总结

整个思路再理一下：

1. 在使用`Handler`前，如果是子线程，需要在当前线程创建`Looper`，初始化`Looper`的时候，`Looper`内部也会初始化一个消息队列`MessageQueue`，将`Looper`存进`ThreadLocal`当中，保证不同线程的`Looper`不同，并且保证一个线程只会有一个`Looper`。
2. 调用`Looper.loop()`,`Looper`开始工作。
3. 创建`Handler`，`Handler`内部会将当前线程中的`Looper`中的消息队列`MessageQueue`赋值给`Handler`中的消息队列。
4. 调用`Handler`发送消息。`Handler`会将消息添加到消息队列`MessageQueue`。
5. 因为`Looper`处于工作状态，不断询问消息队列`MessageQueue`是否有新的消息道来，一但发现有新的消息，就开始处理消息。处理的线程就是调用`Looper.prepare()`的线程，也就是创建`Handler`的线程。

总的来说，线程和`Handler`之间的对应关系是：

1个`Thread` - 1个`Looper` - 1个消息队列`MessageQueue` - 多个`Handler`




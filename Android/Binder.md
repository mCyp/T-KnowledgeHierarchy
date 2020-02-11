# Binder

Binder是安卓中特有的IPC方式，在讲Binder之前，我们看看安卓还可以使用哪些IPC方式以及它们的优缺点：

| 名称     | 拷贝次数 | 特点                                                         |
| -------- | -------- | ------------------------------------------------------------ |
| Socket   | 2        | 通用接口，传输效率低，开销大，广泛用于网络传输和低速地跨进城传输中 |
| 管道     | 2        | 限制因素多，缓冲区有限，使用不当易出错                       |
| 共享内存 | 0        | 控制复杂，不好使用                                           |

与此同时，进程间的安全问题也比较重要，Binder就很好的解决了这一方面的问题：

- 性能方面：完成了内核到用户进程的内存映射，使通常的两次降为一次。
- 安全方面：为发送进程添加PID和UID，从而鉴别对方身份。

基于此，Binder在Android中得到了很好的应用，Messenger、AIDL和ContentProvider都是基于Binder。

## 一、整体把握

#### # 简单介绍一下Binder?

从上面的内容我们得知，Binder是一种IPC方式，性能高且安全。除此以外，Binder很好的应用了C/S架构，从组件角度来看：

![Binder通信](https://teaof-konwleadge-1255982134.cos.ap-shanghai.myqcloud.com/blog/Android/%E8%BF%9B%E7%A8%8B%E9%80%9A%E4%BF%A1/Binder/Binder%E7%BB%84%E4%BB%B6.png)

从上图课题看到四个主要部分：

- **Client**：请求服务的进程。
- **Server**：提供服务的进程。
- **Service Manger**：Service Manager是整个Binder机制的保护进程，用来管理开发者创建的各种Server，其中它主要提供了两个功能，一、供Server注册Binder 二、让Client查询Server的Binder，进而与Server进程通信。
- **Binder驱动**：驱动负责Binder通信的建立。

除了图上的四个部分，我们也可以看到整个过程的三大步骤：

1. Server在Service Manager中注册：Server进程在创建的时候，也会创建对应的Binder实体，如果要提供服务给Client，就必须为Binder实体注册一个名字。
2. Client通过Service Manager获取服务：Client知道服务中Binder实体的名字后，通过名字从Service Manager获取Binder实体的引用。
3. Client使用服务与Server进行通信：Client通过调用Binder实体与Server进行通信。

看了上图你可能会觉得Client和Server是直接进行通行的，其实并不是这样的，可以从下图再感知一下：

![Binder流程](https://teaof-konwleadge-1255982134.cos.ap-shanghai.myqcloud.com/blog/Android/%E8%BF%9B%E7%A8%8B%E9%80%9A%E4%BF%A1/Binder/Binder%E6%B5%81%E7%A8%8B.jpeg)

## 二、重点问题

在简单了解以后，挑几个重要的问题了解一下：

- Service Manager是如何出现的？
- 数据进行传递是如何进行映射的？

#### # Service Manager是如何出现的？

为什么会问这个问题？其实注意整体把握的第一张图，你会发现Service Manager、Server和Client的处在用户空间中，也就是说Service Manager、Server和Client其实都是进程，这就涉及了进程间通信，Service Manager就是用来解决进程间通信的，现在Service Manager和其他Client、Server之间也是进程间通信，听着就像鸡和蛋谁先出现的问题。不过，Service Manager和Client、Server之间都是使用Binder进行通信的，Service Manager就是先出现的那只鸡，在每个进程生成以后，都会有一个0号引用指向Service Manager。

![Binder机制](https://teaof-konwleadge-1255982134.cos.ap-shanghai.myqcloud.com/blog/Android/%E8%BF%9B%E7%A8%8B%E9%80%9A%E4%BF%A1/Binder/Binder%E6%9C%BA%E5%88%B6.jpeg)

明白了解Service Manger的重要性以后，

```c++
int main( int argc, char** argv )
{
	struct binder_state	*bs;
	union selinux_callback	cb;
	char *driver;

	if ( argc > 1 )
	{
		driver = argv[1];
	} else {
    // Binder设备文件
		driver = "/dev/binder";
	}

  // 打开Binder文件
	bs = binder_open( driver, 128 * 1024 );
	// ... 省略
  // 通知当前进程称为Service Manager
	if ( binder_become_context_manager( bs ) )
	{
		ALOGE( "cannot become context manager (%s)\n", strerror( errno ) );
		return(-1);
	}

	// ...
  // ... Service Manger进入工作状态，等待Server、CLient
	binder_loop( bs, svcmgr_handler );

	return(0);
}
```

大概过程就是：

1. **打开/dev/binder文件**：`binder_open(driver, 128 * 1024)`

2. **建立128k内存映射**：`mmap(Null,mapsize,PROT_READ,MAP_PRIVATE,bs->fdd,0)`，这个方法其实来自方法`binder_open(driver, 128 * 1024)`内部。

3. **通知Binder驱动程序它是Service Manger**：`binder_become_context_manager( bs )`
4. **进入工作状态，循环等待请求的到来**：`binder_loop( bs, svcmgr_handler )`

具体的代码就不探究了，了解过程即可。

#### # 内存映射是如何进行的？

将Binder的进程通信之前，我们先看看传统的IPC方式是如何进行数据跨进程传输的，首先，数据会从发送进程的缓存区拷贝到内核缓存区，之后，数据再从内核缓冲区拷贝到接收进程的缓存区，这种存储-转发必定经过两此拷贝。

那么Binder是如何做的呢？Binder将数据接收缓存交给了Binder驱动，Binder驱动通过方法`mmap`创建数据接收的缓存空间：

```c++
mmap(Null,mapsize,PROT_READ,MAP_PRIVATE,bs->fdd,0);
```

这样的Binder的接收方就有一片大小为mapsize的接收缓存区，`mmap()`的返回值内存映射在用户空间的地址，不过这段空间是由驱动管理，用户不必也不能直接访问。

## 具体实现

#### # 如何理解AIDL的原理？

这个过程其实挺简单，可以分为两个部分：

1. 如何获取Binder
2. 获取Binder后的使用

上面的图依旧有用：

![Binder流程](https://teaof-konwleadge-1255982134.cos.ap-shanghai.myqcloud.com/blog/Android/%E8%BF%9B%E7%A8%8B%E9%80%9A%E4%BF%A1/Binder/Binder%E6%B5%81%E7%A8%8B.jpeg)

- [ ]  TODO：具体的以后分析 


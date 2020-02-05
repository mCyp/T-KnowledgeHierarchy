# OKHttp源码分析

网络请求一直是Android中的重中之重，不管你使用的是内网还是外网。而在各个网络库之中，**OkHttp**则占了一席之地。我们平时只知使用**Retrofit**，殊不知**Retrofit**的核心就是**OKHttp**。

既然是源码分析，就不讲OkHttp的使用了，使用可查文章[《OkHttp使用教程》](https://www.jianshu.com/p/ca8a982a116b)

## 一、OkHttp总览

我使用的**OkHttp**的版本是4.2.1，查看源码的发现是用Kotlin写的，有趣~

### 第一步 创建OkHttpClient对象

`OkHttpClient`是一个核心类，为什么说它是核心类呢？因为它负责发送HTTP请求和读取请求结果。点进源码发现很多重要的属性在里面：

```kotlin
// 调度器
@get:JvmName("dispatcher") val dispatcher: Dispatcher = builder.dispatcher
// 连接池
@get:JvmName("connectionPool") val connectionPool: ConnectionPool = builder.connectionPool
// 过滤器
@get:JvmName("interceptors") val interceptors: List<Interceptor> =
      builder.interceptors.toImmutableList()
/**
 * Returns an immutable list of interceptors that observe a single network request and response.
 * These interceptors must call [Interceptor.Chain.proceed] exactly once: it is an error for
 * a network interceptor to short-circuit or repeat a network request.
 */
@get:JvmName("networkInterceptors") val networkInterceptors: List<Interceptor> =
    builder.networkInterceptors.toImmutableList()
// 事件监听器工厂类
@get:JvmName("eventListenerFactory") val eventListenerFactory: EventListener.Factory =
    builder.eventListenerFactory
// ...
// 重定向
@get:JvmName("followRedirects") val followRedirects: Boolean = builder.followRedirects
@get:JvmName("followSslRedirects") val followSslRedirects: Boolean = builder.followSslRedirects
// cookie
@get:JvmName("cookieJar") val cookieJar: CookieJar = builder.cookieJar
// 缓存
@get:JvmName("cache") val cache: Cache? = builder.cache
// 域名
@get:JvmName("dns") val dns: Dns = builder.dns
// 代理
@get:JvmName("proxy") val proxy: Proxy? = builder.proxy
// ...
// socket
@get:JvmName("socketFactory") val socketFactory: SocketFactory = builder.socketFactory
@get:JvmName("sslSocketFactory") val sslSocketFactory: SSLSocketFactory
  get() = sslSocketFactoryOrNull ?: throw IllegalStateException("CLEARTEXT-only client")
// ...
// 协议？
@get:JvmName("protocols") val protocols: List<Protocol> = builder.protocols
// 证书相关
@get:JvmName("hostnameVerifier") val hostnameVerifier: HostnameVerifier = builder.hostnameVerifier
@get:JvmName("certificatePinner") val certificatePinner: CertificatePinner
@get:JvmName("certificateChainCleaner") val certificateChainCleaner: CertificateChainCleaner?
// ... 时间属性
@get:JvmName("callTimeoutMillis") val callTimeoutMillis: Int = builder.callTimeout
@get:JvmName("connectTimeoutMillis") val connectTimeoutMillis: Int = builder.connectTimeout
@get:JvmName("readTimeoutMillis") val readTimeoutMillis: Int = builder.readTimeout
@get:JvmName("writeTimeoutMillis") val writeTimeoutMillis: Int = builder.writeTimeout
@get:JvmName("pingIntervalMillis") val pingIntervalMillis: Int = builder.pingInterval
```

### 第二步 构建Request

既然要构建Http请求，总得知道干什么吧？`Request`的任务就是如此，查看`Request`的构造函数：

```kotlin
/**
 * An HTTP request. Instances of this class are immutable if their [body] is null or itself
 * immutable.
 */
class Request internal constructor(
  @get:JvmName("url") val url: HttpUrl,
  @get:JvmName("method") val method: String,
  @get:JvmName("headers") val headers: Headers,
  @get:JvmName("body") val body: RequestBody?,
  internal val tags: Map<Class<*>, Any>
){
  //...
}
```

即设置Http请求的请求行(路径、方法)、请求头和请求体。

### 第三步 同步Or异步？

同步和异步的请求过程大致是一样的，这里就先分析异步过程，再分析同步过程。

异步代码：`client.newCall(request).enqueue(Callback callback)`

尽管代码简单，但实际过程却很复杂

#### 1. client.newCall(request)

`client.newCall(request)`这个过程返回的是一个`Call`，`Call`是什么？它其实代表一个等待被执行的请求，这个可以从它的注释中可以看出：

```kotlin
// 代表等待被执行的请求，可以被取消
interface Call : Cloneable {
  // 获取请求
  fun request(): Request
  // 同步执行
  @Throws(IOException::class)
  fun execute(): Response
  // 异步执行
  fun enqueue(responseCallback: Callback)
  // 取消请求
  fun cancel()
  // 检查是否被执行
  fun isExecuted(): Boolean
  // 检查是否被取消
  fun isCanceled(): Boolean
  // 超时
  fun timeout(): Timeout
  // OkHttpClient实现这个接口
  interface Factory {
    fun newCall(request: Request): Call
  }
}
```

`Call`是一个接口，具体的实现类是`RealCall`，它才是真实的处理类，那么真实的异步调用方法`enqueue(responseCallback: Callback)`是如何实现的呢？

#### 2. enqueue(responseCallback: Callback)

```kotlin
override fun enqueue(responseCallback: Callback) {
  synchronized(this) {
    check(!executed) { "Already Executed" }
    executed = true
  }
  transmitter.callStart()
  client.dispatcher.enqueue(AsyncCall(responseCallback))
}
```

介绍一下这个里面的三个元素：

- transmitter -- `Transmitter`，它是OkHttp的应用层和网络层的桥梁，这个类显示了应用层的几个重要元素：连接(`connections`)、请求(`requests`)、结果(`responses`)和流(`streams`)，不过这个类在OkHttp的前几个版本没有出现。
- dispatcher -- `Dispatcher`，异步执行的调度器
- `AsyncCall`：异步执行的网络请求的的任务，后面解释。

#### 3. dispatcher.enqueue(AsyncCall(responseCallback))

可以发现，调度器里面放了一个`AsyncCall`，它的构造方法里面放了一个参数 - 请求结果的回调`responseCallback`，这个就不用介绍了，那么`AsyncCall`是什么？它就是一个异步执行的调度任务，实现了`Runnable`接口：

```kotlin
internal inner class AsyncCall(
    private val responseCallback: Callback
  ) : Runnable {
  //...
}
```

已知`AsyncCall`，再看调度器的如何执行：

```kotlin
internal fun enqueue(call: AsyncCall) {
  synchronized(this) {
    // 将异步任务添加进准备队列中
    readyAsyncCalls.add(call)
    if (!call.get().forWebSocket) {
      val existingCall = findExistingCallWithHost(call.host())
      if (existingCall != null) call.reuseCallsPerHostFrom(existingCall)
    }
  }
  promoteAndExecute()
}

private fun promoteAndExecute(): Boolean {
  assert(!Thread.holdsLock(this))

  val executableCalls = mutableListOf<AsyncCall>()
  val isRunning: Boolean
  synchronized(this) {
    val i = readyAsyncCalls.iterator()
    while (i.hasNext()) {
      val asyncCall = i.next()
	  
      // 请求数量或者Host数量大于最大值会取消
      if (runningAsyncCalls.size >= this.maxRequests) break // Max capacity.
      if (asyncCall.callsPerHost().get() >= this.maxRequestsPerHost) continue // Host max capacity.
      
      i.remove()
      asyncCall.callsPerHost().incrementAndGet()
      // 加入执行列表
      executableCalls.add(asyncCall)
      runningAsyncCalls.add(asyncCall)
    }
    isRunning = runningCallsCount() > 0
  }

  for (i in 0 until executableCalls.size) {
   	// 取出任务，放入线程池中执行
    val asyncCall = executableCalls[i]
    asyncCall.executeOn(executorService)
  }

  return isRunning
}
```

可以看到，最后的结果就是将任务`AsyncCall`放入线程池中`executorService`中执行，你可能会说`asyncCall.executeOn(executorService)`这跟平常线程池的调度不一样，别急，看具体实现：

```kotlin
fun executeOn(executorService: ExecutorService) {
	assert(!Thread.holdsLock(client.dispatcher))
	var success = false
	try {
      	// 放入线程池executorService this是AsyncCall
	    executorService.execute(this)
	    success = true
	} catch (e: RejectedExecutionException) {
		// ...
	}
	finally {
		if (!success) {
			client.dispatcher.finished(this) // This call is no longer running!
		}
	}
}
```

显然，这是我们预料到的结果，下面，核心知识来了，且看异步调度任务的执行方法`AsyncCall#run()`：

```kotlin
override fun run() {
	threadName("OkHttp ${redactedUrl()}") {
		// ...
		try {
			val response = getResponseWithInterceptorChain()
			// ...
			responseCallback.onResponse(this@RealCall, response)
		}
		catch (e: IOException) {
			if (signalledCallback) {
				// ...
			} else {
				responseCallback.onFailure(this@RealCall, e)
			}
		}
		finally {
			client.dispatcher.finished(this)
		}
	}
}
```

原来我们想要的**请求结果**`Response`是在这里获取的：

```kotlin
@Throws(IOException::class)
fun getResponseWithInterceptorChain(): Response {
  // 一系列过滤器
  val interceptors = mutableListOf<Interceptor>()
  // OkhttpClient自己设置的过滤器
  interceptors += client.interceptors
  // 重新连接和重定向过滤器
  interceptors += RetryAndFollowUpInterceptor(client)
  // 用于应用层和网络层的过滤器
  interceptors += BridgeInterceptor(client.cookieJar)
  // 缓存过滤器
  interceptors += CacheInterceptor(client.cache)
  // 连接过滤器
  interceptors += ConnectInterceptor
  if (!forWebSocket) {
    // 这个也是自己设置的过滤器
    interceptors += client.networkInterceptors
  }
  interceptors += CallServerInterceptor(forWebSocket)

  // 责任链模式调度
  val chain = RealInterceptorChain(interceptors, transmitter, null, 0, originalRequest, this,
      client.connectTimeoutMillis, client.readTimeoutMillis, client.writeTimeoutMillis)

  try {
    val response = chain.proceed(originalRequest)
    // ...
    return response
  } catch (e: IOException) {
    calledNoMoreExchanges = true
    throw transmitter.noMoreExchanges(e) as Throwable
  } finally {
    if (!calledNoMoreExchanges) {
      transmitter.noMoreExchanges(null)
    }
  }
}
```

一大段一大段的代码看起来确实难受，不过这一段代码也确实是**OKHttp的精髓**，不可不看，以下是两个关注点：

- 从代码开始，就使用了过滤器的集合，包括了系统必须使用的过滤器如重连和重定向过滤器、连接过滤器和缓存过滤器等，以及自行添加的过滤器如打印日志的过滤器。
- 确定了过滤器，下面就是挨个使用，方法就是使用[责任链模式](https://www.runoob.com/design-pattern/chain-of-responsibility-pattern.html)，主要就是`RealInterceptorChain`和过滤器`interceptors`之间的调度。

以上就是异步方法的调度了，同步的方法类似，用一张网友的图片简述一下整个过程：

![调度](https://teaof-konwleadge-1255982134.cos.ap-shanghai.myqcloud.com/blog/Android/Code%20Analysis/7866586-db6c625e9fa03ae0.png)

## 二、过滤器

虽说，从上面的过程中已经可以了解OkHttp的大致过程了，但我们肯定不满足于此，我就挑几个我关心的问题，进行深入探索：

1. `BridgeInterceptor`起的作用是什么？
2. **OkHttp**是如何进行缓存管理的？
3. **OKHttp**是如何进行网络连接的？

#### # BridgeInterceptor是什么？又起了什么作用？

简单看了一下`BridgeInterceptor`的介绍，似乎就想通了：

> Bridges from application code to network code.

简而言之，就是应用层代码和网络层代码之间的桥梁，只要搞懂什么是**应用层代码**和是什么**网络层代码，**我想这个问题就好解决了。

以下是我的个人理解：

- 应用层代码：还没有组装好的网络请求对象，这个时候已知的信息都是零件。比如，我们用`Post`方法请求数据，已经有了请求对象(例如：请求的一些参数日期、序号等)，还知道请求地址和请求规则。
- 网络层代码：知道上述内容显然是不能够进行网络请求的，这个时候我们需要将上述这些内容封装成HTTP请求，就是`Request`对象，封装好了就是一个一目了然的网络层代码了，能够被网络传输时组装成网络报文。

再看一下`BridgeInterceptor#intercept(chain: Interceptor.Chain)`的发送部分：

```kotlin
@Throws(IOException::class)
override fun intercept(chain: Interceptor.Chain): Response {
  val userRequest = chain.request()
  val requestBuilder = userRequest.newBuilder()

  val body = userRequest.body
  if (body != null) {
    val contentType = body.contentType()
    if (contentType != null) {
      requestBuilder.header("Content-Type", contentType.toString())
    }

    val contentLength = body.contentLength()
    if (contentLength != -1L) {
      requestBuilder.header("Content-Length", contentLength.toString())
      requestBuilder.removeHeader("Transfer-Encoding")
    } else {
      requestBuilder.header("Transfer-Encoding", "chunked")
      requestBuilder.removeHeader("Content-Length")
    }
  }

  if (userRequest.header("Host") == null) {
    requestBuilder.header("Host", userRequest.url.toHostHeader())
  }

  if (userRequest.header("Connection") == null) {
    requestBuilder.header("Connection", "Keep-Alive")
  }

  //... 设置编码 cookie 用户信息 等等

  val networkResponse = chain.proceed(requestBuilder.build())
}
```

#### # OkHttp是如何进行缓存管理的？

从名字就可以看出，`CacheInterceptor`是负责缓存管理的，所以秘密应该在`CacheInterceptor`中。

代码见注释：

```kotlin
@Throws(IOException::class)
override fun intercept(chain: Interceptor.Chain): Response {
  	// 查看是否有缓存
	val cacheCandidate = cache?.get(chain.request())
	val now = System.currentTimeMillis()
    
    // 获取缓存策略
	val strategy = CacheStrategy.Factory(now, chain.request(), cacheCandidate).compute()
	val networkRequest = strategy.networkRequest
	val cacheResponse = strategy.cacheResponse
	cache?.trackResponse(strategy)
	if (cacheCandidate != null && cacheResponse == null) {
		// The cache candidate wasn't applicable. Close it.
		cacheCandidate.body?.closeQuietly()
	}
    
	// 1. 禁用网络并且本地缓存也为空
	if (networkRequest == null && cacheResponse == null) {
		return Response.Builder()
		          .request(chain.request())
		          .protocol(Protocol.HTTP_1_1)
		          .code(HTTP_GATEWAY_TIMEOUT)
		          .message("Unsatisfiable Request (only-if-cached)")
		          .body(EMPTY_RESPONSE)
		          .sentRequestAtMillis(-1L)
		          .receivedResponseAtMillis(System.currentTimeMillis())
		          .build()
	}
    
	// 2. 禁用网络，可以用本地缓存
	if (networkRequest == null) {
		return cacheResponse!!.newBuilder()
		          .cacheResponse(stripBody(cacheResponse))
		          .build()
	}
    
	var networkResponse: Response? = null
	try {
	    networkResponse = chain.proceed(networkRequest)
	}
	finally {
		// If we're crashing on I/O or otherwise, don't leak the cache body.
		if (networkResponse == null && cacheCandidate != null) {
			cacheCandidate.body?.closeQuietly()
		}
	}
	// 3. 有网络的情况下也有本地缓存.
	if (cacheResponse != null) {
		if (networkResponse?.code == HTTP_NOT_MODIFIED) {
			val response = cacheResponse.newBuilder()
		            .headers(combine(cacheResponse.headers, networkResponse.headers))
		            .sentRequestAtMillis(networkResponse.sentRequestAtMillis)
		            .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis)
		            .cacheResponse(stripBody(cacheResponse))
		            .networkResponse(stripBody(networkResponse))
		            .build()
		        networkResponse.body!!.close()
		        // Update the cache after combining headers but before stripping the
			// Content-Encoding header (as performed by initContentStream()).
			cache!!.trackConditionalCacheHit()
		    cache.update(cacheResponse, response)
		    return response
		} else {
			cacheResponse.body?.closeQuietly()
		}
	}
    
	val response = networkResponse!!.newBuilder()
        			.cacheResponse(stripBody(cacheResponse))
        			.networkResponse(stripBody(networkResponse))
        			.build()
    // 4. cache不为null，写入缓存
    if (cache != null) {
		if (response.promisesBody() && CacheStrategy.isCacheable(response, networkRequest)) {
			// Offer this request to the cache.
			val cacheRequest = cache.put(response)
		    return cacheWritingResponse(cacheRequest, response)
		}
		if (HttpMethod.invalidatesCache(networkRequest.method)) {
			try {
				cache.remove(networkRequest)
			}
			catch (_: IOException) {
				// The cache cannot be written.
			}
		}
	}
	return response
}
```

#### # 如何进行网络连接？

看完上面两个答案，你可能会轻松的回答，答案在`ConnectInterceptor`，看完它的代码，确实如此！`ConnectInterceptor`中显示的代码，其实非常少：

```kotlin
/** Opens a connection to the target server and proceeds to the next interceptor. */
object ConnectInterceptor : Interceptor {

  @Throws(IOException::class)
  override fun intercept(chain: Interceptor.Chain): Response {
    val realChain = chain as RealInterceptorChain
    val request = realChain.request()
    val transmitter = realChain.transmitter()

    val doExtensiveHealthChecks = request.method != "GET"
    val exchange = transmitter.newExchange(chain, doExtensiveHealthChecks)

    return realChain.proceed(request, transmitter, exchange)
  }
}
```

层层嵌套的代码扒的太难受了，直接奔重点，看建立连接的代码：

```kotlin
fun find(
    client: OkHttpClient,
    chain: Interceptor.Chain,
    doExtensiveHealthChecks: Boolean
  ): ExchangeCodec {
	val connectTimeout = chain.connectTimeoutMillis()
	// ... 省略参数
	try {
      	  // 寻找一个连接
	      val resultConnection = findHealthyConnection(
	      	  connectTimeout = connectTimeout,
	      	  readTimeout = readTimeout,
	      	  writeTimeout = writeTimeout,
	      	  pingIntervalMillis = pingIntervalMillis,
	      	  connectionRetryEnabled = connectionRetryEnabled,
	      	  doExtensiveHealthChecks = doExtensiveHealthChecks
		  )
	      return resultConnection.newCodec(client, chain)
	}
	catch (e: RouteException) {
		  trackFailure()
	      throw e
	}
	catch (e: IOException) {
		  trackFailure()
	      throw RouteException(e)
	}
}
```

上面的代码意思是找一个健康的连接？从参数看是要找一个读取响应时间都正常的连接，点进`ExchangeFinder#findHealthyConnection()`：

```kotlin
@Throws(IOException::class)
private fun findHealthyConnection(
  //... 参数省略
): RealConnection {
	while (true) {
		val candidate = findConnection(
		      connectTimeout = connectTimeout,
		      readTimeout = readTimeout,
		      writeTimeout = writeTimeout,
		      pingIntervalMillis = pingIntervalMillis,
		      connectionRetryEnabled = connectionRetryEnabled
		 )
		      
      	//...
		
      	// 检验
		if (!candidate.isHealthy(doExtensiveHealthChecks)) {
			candidate.noNewExchanges()
			        continue
		}
		return candidate
	}
}
```

如方法名分为两个过程：

1. 寻找连接
2. 检验连接是否健康，其实是检测`Socket`是否关闭，与上面我的猜测不相同~

肯定是继续点进寻找连接的方法，该方法就是寻找连接的逻辑核心了`ExchangeFinder#findConnection(...)`：

```kotlin
/**
 * 获取一个连接去存放流，获取的优先级是 已经存在的连接、连接池、建立一个新的连接
 */
@Throws(IOException::class)
private fun findConnection(
    connectTimeout: int,
    readTimeout: int,
    writeTimeout: int,
    pingIntervalMillis: int,
    connectionRetryEnabled: Boolean
): RealConnection {
  	// 是否从连接池中找到连接
	var foundPooledConnection = false
    // 获取的连接
	var result: RealConnection? = null
    // 选择的路由，这个我不是特别清楚 ...
	var selectedRoute: Route? = null
    // 释放的连接
	var releasedConnection: RealConnection?
	val toClose: Socket?
	synchronized(connectionPool) {
    	if (transmitter.isCanceled) throw IOException("Canceled")
    
		hasStreamFailure = false // This is a fresh attempt.
    	releasedConnection = transmitter.connection
    	toClose = if (transmitter.connection != null && transmitter.connection!!.noNewExchanges) {
			transmitter.releaseConnectionNoEvents()
		} else {
			null
		}
    
    	// 1. 当前已经存在的连接
		if (transmitter.connection != null) {
			// We had an already-allocated connection and it's good.
			result = transmitter.connection
	    	releasedConnection = null
		}
    
		if (result == null) {
			// 2. 尝试从连接池获取一条连接
			if (connectionPool.transmitterAcquirePooledConnection(address, transmitter, null, false)) {
				foundPooledConnection = true
				result = transmitter.connection
			} else if (nextRouteToTry != null) {
				selectedRoute = nextRouteToTry
				nextRouteToTry = null
			} else if (retryCurrentRoute()) {
				selectedRoute = transmitter.connection!!.route()
			}
		}
	}
	toClose?.closeQuietly()
	if (releasedConnection != null) {
		eventListener.connectionReleased(call, releasedConnection!!)
	}
	if (foundPooledConnection) {
		eventListener.connectionAcquired(call, result!!)
	}
	if (result != null) {
		// If we found an already-allocated or pooled connection, we're done.
		return result!!
	}
	// If we need a route selection, make one. This is a blocking operation.
	var newRouteSelection = false
	if (selectedRoute == null && (routeSelection == null || !routeSelection!!.hasNext())) {
		newRouteSelection = true
		routeSelection = routeSelector.next()
	}
	var routes: List<Route>? = null
	synchronized(connectionPool) {
		if (transmitter.isCanceled) throw IOException("Canceled")
		if (newRouteSelection) {
			// Now that we have a set of IP addresses, make another attempt at getting a connection from
			// the pool. This could match due to connection coalescing.
          	// HTTP 2 是可以在一个连接中进行多个HTTP请求的
			routes = routeSelection!!.routes
			if (connectionPool.transmitterAcquirePooledConnection(
					address, transmitter, routes, false)) {
				foundPooledConnection = true
				result = transmitter.connection
			}
		}
		if (!foundPooledConnection) {
			if (selectedRoute == null) {
				selectedRoute = routeSelection!!.next()
			}
		
      		// 创建连接
			result = RealConnection(connectionPool, selectedRoute!!)
			connectingConnection = result
		}
	}
                                    
	// If we found a pooled connection on the 2nd time around, we're done.
	if (foundPooledConnection) {
		eventListener.connectionAcquired(call, result!!)
		return result!!
	}
                                        
	// Do TCP + TLS handshakes. This is a blocking operation.
    // 进行TCP和TLS的握手 TCP跟HTTP连接有关 TLS跟HTTPS相关
	result!!.connect(
		connectTimeout,
		readTimeout,
		writeTimeout,
		pingIntervalMillis,
		connectionRetryEnabled,
		call,
		eventListener
	)
	connectionPool.routeDatabase.connected(result!!.route())
	var socket: Socket? = null
	synchronized(connectionPool) {
		connectingConnection = null
		// 同上次连接Host相同
		if (connectionPool.transmitterAcquirePooledConnection(address, transmitter, routes, true)) {
			result!!.noNewExchanges = true
			socket = result!!.socket()
			result = transmitter.connection
			nextRouteToTry = selectedRoute
		} else {
			connectionPool.put(result!!)
			transmitter.acquireConnectionNoEvents(result!!)
		}
	}
	socket?.closeQuietly()
	eventListener.connectionAcquired(call, result!!)
	return result!!
}
```

上面的代码也不需要一次都看下来，主要明白上面的代码作用是获取连接，逻辑是：

1. 优先从未关闭的连接从获取
2. 从连接池中获取
3. 最后实在没辙，就重新创建一个连接，并且需要经历TCP和TLS的握手

到这里也就差不多了，再深入就是`Socket`相关的知识了。


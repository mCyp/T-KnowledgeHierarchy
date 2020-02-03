# AsyncTask

#### # AsyncTask是什么？

`AsyncTask`是一个轻量级的的异步任务类，它可以在线程池中执行后台任务，然后把执行任务的进度和结果传递给主线程。

关于`AsyncTask`的使用，这里就不再多介绍了，**《第一行代码》**和**《Android开发艺术探索》**都有介绍，感兴趣可以查阅一下，其中，关键的就是四个方法：

1. `onPreExecute()`：主线程执行，任务执行前被调度
2. `doInBackground(Params... params)`：线程池中执行，在该方法中调用`publisProgress(int num)`更新任务进度
3. `onProgressUpdate(Progress... values)`：在主线程中执行任务，后台调用`publisProgress(int num)`会调用该方法
4. `onPostExecute(Result result)`：主线程执行，得到任务结果

#### # AsyncTask使用注意事项？

在日常使用`AsyncTask`需要注意：

- `AsyncTask`必须在主线程中创建
- 一个`AsyncTask`对象只能执行一次，即只能调用一次`execute()`方法
- Android 1.6之前，`AsyncTask`串行执行任务，Android 1.6以后开发并行执行任务，但是到了Android 3.0，为了避免`AsyncTask`所带来的的并发错误，`AsyncTask`又采用一个线程串行执行任务，尽管如此，我们任然可以调用`executeOnExecutor()`方法来并行地执行任务

#### # 简单分析AsyncTask原理？

源码可以查看：

[Android Pie：AsyncTask源码](https://www.androidos.net.cn/android/9.0.0_r8/xref/frameworks/base/core/java/android/os/AsyncTask.java)

`AsyncTask`的源码其实不复杂，主要是借助了线程池和`Handler`，线程池用来添加任务，`Handler`用来更新主线程，线程池的能够存放的线程跟Cpu有关。


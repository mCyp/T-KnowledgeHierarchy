# Android性能优化

## # 崩溃优化

### 1. 常见的崩溃

常见的崩溃分为两种，**Java崩溃**和**Native崩溃**：

- Java崩溃：出现在Java代码中，出现了未捕获的异常，导致程序异常退出
- Native崩溃：一般都是Native代码中访问了非法地址，也可能是地址对齐出现了问题，或者发生了程序主动的abort，这些都产生了响应的signal信号，导致程序异常退出

### 2. 崩溃产生的流程

**Java崩溃**

**Native崩溃**

Nativie崩溃知识扫盲：[《Android 平台 Native 代码的崩溃捕获机制及实现》](https://mp.weixin.qq.com/s/g-WzYF3wWAljok1XjPoo7w)

### 3. 崩溃搜集工具

常见的崩溃搜集工具有腾讯Bugly，阿里的啄木鸟平台等

##### Java崩溃

##### Native崩溃

常见的Nativie崩溃的搜集工具有[BreakPad](https://github.com/google/breakpad)

### 4. 崩溃搜集技巧

##### 崩溃现场需要注意

- 崩溃的基本信息：线程名，进程名、崩溃堆栈和类型
- 系统信息：Logcat、机型、系统、厂商、CPU、设备状态信息
- 内存信息
- 资源信息：文件句柄、线程数和JNI
- 应用信息

##### 崩溃分析

1. 确定重点
2. 查找共性
3. 尝试复现




### android是否会断掉进程
- android线程中异常会断掉进程
- java线程中异常 会断掉 线程

### app崩溃的主要原因
- 未捕获异常的崩溃：java中未捕捉异常的处理流程，native未捕获异常的处理流程
- anr崩溃：ANR的处理方案，Service处理的ANR过程
- 内存引起的崩溃：抖动，益出。实际上是前面两者的崩溃

### android未捕捉异常处理机制
- 未捕捉异常出现会交给JVM处理，JVM收集对应的堆栈信息，然后调用dispatchUncaughtException，开始执行异常的分发处理流程
- dispatchUncaughtException中，先判断有无异常预处理器，即uncaughtExceptionPreHandler，有的话，将会先调用异常预处理器的uncaughtException方法
- 接下来获取异常处理器，并调用其uncaughtException方法。至此，整个异常分发处理流程完毕
- App进程启动时，系统已经在RuntimeInit.java类中注册了一个默认的异常预处理器loggingHandler
- 在App进程启动时，系统会自动注入loggingHandler对象，作为异常预处理器。当有未捕获异常发生时，以此会自动调用loggingHandler对象的uncaughtException方法，以完成默认的日志输出。
- App启动时，系统会默认为其设置一个线程默认的异常处理器，当未捕获异常发生时，默认情况下的闪退就是这个线程默认的异常处理器，即KillApplicationHandler去具体触发的。

### anr崩溃参考卡顿优化
- [卡顿优化](./android/性能/卡顿优化.md)

### 内存引起的崩溃参考内存优化
- [内存优化](./android/性能/内存优化.md)

### 如何优化Crash
- 首先讲卡顿崩溃的原因
- 然后讲怎么去定位问题，等位代码
- 最后解决解决
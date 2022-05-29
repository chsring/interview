### 使用Lifcycle解耦页面与组件
```java
//自定义view 需要实现LifecycleOberver
//添加生命周期的注解
@OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
private void start(){}
@OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
private void stop(){}
```
```java
//在activity 中注册生命周期的方法
getLifecycle().addObserver(chronometer);
```


### 使用LifecycleService解耦Service与组件
- 启动一个service在后台，获取GPS信息



### 使用ProcessLifecycleOwner监听程序生命周期
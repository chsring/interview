### 使用
```java
liveData.observe(this,Observer<String>()){
    public void onChanged(String data){
        
    }
}
```

- Livedata是一个观察者，可以观察到Activity和Fragment生命周期的变化，通过lifecycle完成观察者被观察者的绑定
- 同时也是被观察者，例如数据变化时，会发送一个通知告诉观察者。
- Livedata有两个方法，setValue():主线程发消息，postValue():从子线程发消息 


### Livedata使用与实现原理


### Livedata粘性事件源码分析


### Jetpack中状态机如何管理生命周期


### hook实现Livedata非粘性功能


### Livedata递归调用源码中如何做容错


### 企业级LivedataBus封装实现
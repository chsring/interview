### Android线程交互
#### 子线程与主线程的Handler通信
- 主线程自带Looper机制，所以不需要创建Looper

#### 子线程之间的Handler通信
- 子线程在处理消息时的handler需要创建Looper，并在Handler之后需要Looper.loop()；
- 发消息时，必须保证MyThread的run方法执行，因为只有looper准备好之后，才会开启消息队列的循环
```java
　　class MyThread extends Thread {
　　　　@Override
　　　　public void run() {
　　　　　　Looper.prepare();
　　　　　　mSubHandler = new Handler() {
　　　　　　　　@Override
　　　　　　　　public void handleMessage(Message msg) {
　　　　　　　　　　switch (msg.what) {
　　　　　　　　　　　　case 1:
　　　　　　　　　　　　Log.i(TAG, (String) msg.obj);
　　　　　　　　　　　　break;
　　　　　　　　　　}
　　　　　　　　}
　　　　　　};
　　　　　　Looper.loop();
　　　　}
　　}
}
```
#### HandlerThread
- HandlerThread是一个包含Looper的Thread，我们可以直接使用这个 Looper 创建 Handler。
- HandlerThread适用于单线程+异步队列模型场景，相对Handler + Thread简洁。
```java
Handler mThreadHandler = new Handler(mHandlerThread.getLooper()) {
    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case 1:
            Log.i(TAG, "threadName--" + Thread.currentThread().getName() + (String) msg.obj);
            break;
        }
    }
};   

// 发送消息至HanderThread
mThreadHandler.sendMessage(msg);
```

#### runOnuiThread 子线程更新主线程ui
```java
new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            Thread.sleep( 1000 );
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
 
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                // 执行UI操作 
                Toast.makeText(MainActivity.this, "Test", Toast.LENGTH_SHORT).show();
            }
        });
    }
}).start();
```
#### view.post(Runnable r);view.postDelay(Runnable r)

#### AsyncTask
- AsyncTask是一个抽象类，它是由Android封装的一个轻量级异步类。它可以在线程池中执行后台任务，然后把执行的进度和最终结果传递给主线程并在主线程中更新UI

- https://blog.csdn.net/weixin_30254435/article/details/98309798

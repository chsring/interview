### 缓存机制加载流程
- Glide.with(this).load(url).into(imageView);//默认是开启内存缓存和磁盘缓存的。
- 运行时缓存：活动缓存，内存缓存。持久性缓存：磁盘缓存
- 加载图像时先找活动缓存（非LRU算法），找到直接加载显示
- 如果找不到活动缓存，则找LRU内存缓存，找到后，从内存缓存中移除，然后添加进活动缓存中，加载显示
- 如果找不到LRU内存缓存，则找LRU磁盘缓存，找到后添加进活动缓存中，然后加载显示
- 如果找不到 则访问Http，请求成功后会保存到磁盘缓存，添加进活动缓存中，然后加载显示

### 缓存回收流程
- 动缓存里面的图片不再使用的时候会将图片移动到内存缓存，当内存缓存数据超过限制时候就进行移除。
- 这里注意到是一个Bitmap可能被多个地方使用，所以要对其引用进行计算，没有引用时候才移动到内存缓存。

### LRU算法
- 磁盘缓存和内存缓存均使用
- 内部是一个链表，在put的时候 当链表元素大于最大值之后，就把最近最少使用的元素移除掉，也就是链表首元素移除掉
- 如果这时使用了链表首元素，则把首元素拿出来放到链表最后，链表第二个元素变成了首元素
- LRUCache类，直接封装了LinkedHashMap。如果LinkedHashMap(0,0.75,true) 最后一个参数设置为true就开启了LRU算法。

### 活动缓存的由来，为什么有活动缓存？
- 内存缓存中使用的是LRU算法，如果我正在加载一张图片，该图片是链表首元素就会放到上面，此时又put进多张图片，达到了maxSize
- 如果没有活动缓存，就会直接把正在显示的图片回收掉导致崩溃

### Glide生命周期(Glide的生命周期方法有三个onStart、onStop、onDestory)会与Activity、Fragment保持一致
- Glide.with(this)绑定了Activity的生命周期。在Activity内新建了一个无UI的Fragment
- 这个Fragment持有一个Lifecycle，通过Lifecycle在Fragment关键生命周期通知RequestManager进行相关从操作
- 在生命周期onStart时继续加载，onStop时暂停加载，onDestory时停止加载任务和清除操作。

#### 源码层 以context为FragmentActivity为例
- with方法可以接受Context，Activity，FragmentActivity，Fragment和View不同的类型。with函数的执行, 会构造glide单例，而RequestManagerRetriever在initializeGlide中会进行构造
- with方法会构造完equestManagerRetriever通过get返回一个 RequestManager
```java
public class Glide implements ComponentCallbacks2 {
   /**使用上下文启动的任何请求将应用Application级别的options，不会根据生命周期时间启动和停止
     *这个方法适用于在普通的Fragment、Activity之外的使用场景，比如在Service中、notification
     *中的缩略图
     */
  public static RequestManager with(@NonNull Context context) {
    return getRetriever(context).get(context);
  }
  /**启动一个加载绑定在Activity的生命周期上，并使用Activity的默认options
   */
  public static RequestManager with(@NonNull Activity activity) {
    return getRetriever(activity).get(activity);
  }
  /**启动一个加载绑定在FragmentActivity的生命周期上，并使用FragmentActivity的默认options
   */
  public static RequestManager with(@NonNull FragmentActivity activity) {
    return getRetriever(activity).get(activity);
  }
  /**启动一个加载绑定在Fragment的生命周期上，并使用Fragment的默认options
   */
  public static RequestManager with(@NonNull Fragment fragment) {
    return getRetriever(fragment.getContext()).get(fragment);
  }
}
```
- RequestManager 用来管理Glide的生命周期，实现了 LifecycleListener方法，onStart，onStop，onDestroy 三个方法具体处理逻辑便是写在这个类中
```java
  RequestManager(Glide glide,Lifecycle lifecycle,RequestManagerTreeNode treeNode,RequestTracker requestTracker,ConnectivityMonitorFactory factory,Context context) {
    this.glide = glide;
    this.lifecycle = lifecycle;
    this.treeNode = treeNode;
    this.requestTracker = requestTracker;
    this.context = context;

    connectivityMonitor =factory.build(context.getApplicationContext(),new RequestManagerConnectivityListener(requestTracker));
    if (Util.isOnBackgroundThread()) {
      Util.postOnUiThread(addSelfToLifecycle);
    } else {
        // Lifecycle lifecycle 在此注册监听，获取的是 SupportRequestManagerFragment 中的 ActivityFragmentLifecycle
        // 当activity触发onStart 会执行 SupportRequestManagerFragment的onStart，进而会触发 ActivityFragmentLifecycle lifecycle.onStart
      lifecycle.addListener(this);
    }
    lifecycle.addListener(connectivityMonitor);

    defaultRequestListeners =
        new CopyOnWriteArrayList<>(glide.getGlideContext().getDefaultRequestListeners());
    setRequestOptions(glide.getGlideContext().getDefaultRequestOptions());

    glide.registerRequestManager(this);
  }
```
- 以传入Activity为例，get方法最终调用的是 supportFragmentGet 方法
```java
public class RequestManagerRetriever implements Handler.Callback {
    @NonNull
    public RequestManager get(@NonNull FragmentActivity activity) {
        if (Util.isOnBackgroundThread()) {
            return get(activity.getApplicationContext());
        } else {
            assertNotDestroyed(activity);
            frameWaiter.registerSelf(activity);
            FragmentManager fm = activity.getSupportFragmentManager();
            return supportFragmentGet(activity, fm, /*parentHint=*/ null, isActivityVisible(activity));
        }
    }
}
```
- supportFragmentGet 方法主要做了两件事：
- 通过 getSupportRequestManagerFragment() 方法创建 SupportRequestManagerFragment，完成Activity与Fragment绑定
- 创建 RequestManager，通过调用 setRequestManager() 完成RequestManager 和 RequestManagerFragment 的绑定。
```java
public class RequestManagerRetriever implements Handler.Callback {
    private RequestManager supportFragmentGet(Context context, FragmentManager fm, android.app.Fragment parentHint, boolean isParentVisible) {
        //①在当前Activity添加一个RequestManagerFragment用于管理请求的生命周期（完成当前） 
        SupportRequestManagerFragment current = getSupportRequestManagerFragment(fm, parentHint, isParentVisible);
        //获取RequestManager 
        RequestManager requestManager = current.getRequestManager();
        //如果不存在RequestManager，则创建 
        if (requestManager == null) {
            Glide glide = Glide.get(context);
            //②构建RequestManager   
            //current.getGlideLifecycle()就是ActivityFragmentLifecycle，也就是构建RequestManager时会传入fragment中的ActivityFragmentLifecycle 
            requestManager = factory.build(glide, current.getGlideLifecycle(), current.getRequestManagerTreeNode(), context);
            //这里实际就是isActivityVisable
            if (isParentVisible) {
                //requestManger调用onStart
                requestManager.onStart();
            }
            //将构建出来的RequestManager绑定到fragment中 
            current.setRequestManager(requestManager);
        }
        //返回当前请求的管理者 
        return requestManager;
    }
}
```
- 在 SupportRequestManagerFragment 中先查看当前Activity中有没有FRAGMENT_TAG这个标签对应的Fragment，如果有就直接返回
- 如果没有，会判断pendingRequestManagerFragments中有没有，如果有就返回
- 如果没有，就会重写new一个，然后放入到pendingRequestManagerFragments中，然后添加到当前Activity，再给Handler发送一条移除的消息
```java
public class RequestManagerRetriever implements Handler.Callback {
    @NonNull
    private SupportRequestManagerFragment getSupportRequestManagerFragment(
            @NonNull final FragmentManager fm, @Nullable Fragment parentHint) {
        //根据tag从fragmentManger中获取
        SupportRequestManagerFragment current =
                (SupportRequestManagerFragment) fm.findFragmentByTag(FRAGMENT_TAG);
        if (current == null) {
            //没有获取达到从内存的一个map集合中获取,可以避免重复创建。
            current = pendingSupportRequestManagerFragments.get(fm);
            if (current == null) {
                //还没有获取到，则直接new一个fragment，在上段源码注释中已经说明，此是一个无view的frag
                current = new SupportRequestManagerFragment();
                current.setParentFragmentHint(parentHint);
                //添加到内存map集合中，待用。相同的Activity中加载图片的时候，就可以获取到了，主要依赖于
                //fragmentManger对象是否相同，查询条件传入的参数就是fm实例
                pendingSupportRequestManagerFragments.put(fm, current);
                //添加、提交
                fm.beginTransaction().add(current, FRAGMENT_TAG).commitAllowingStateLoss();
                //删除临时临时映射关系，不用管这块
                handler.obtainMessage(ID_REMOVE_SUPPORT_FRAGMENT_MANAGER, fm).sendToTarget();
            }
        }
        return current;
    }
}

```
- Glide就是通过它作为生命周期的分发入口，SupportRequestManagerFragment 的默认构造函数会实例化一个ActivityFragmentLifecycle，在生命周期onStart/onStop/onDestroy中会调用ActivityFragmentLifecycle的相应方法：
```java
// SupportRequestManagerFragment.java
/**一个无view的fragment用于安全的持有requestManager用于开始、停止和管理glide的请求
 *这个fragment作为子组件被添加到Activity或是Fragment上*/
public class SupportRequestManagerFragment extends Fragment {
    //Glide包自带的lifecycle组件(和androidx中相关lifecycle组件区分开)
    private final ActivityFragmentLifecycle lifecycle;
  ....
    @Nullable private RequestManager requestManager;
    //new对象进行创建的时候，构建了一个lifecycle对象
    public SupportRequestManagerFragment() {
        this(new ActivityFragmentLifecycle());
    }
    //成员变量lifecycle被赋值
    public SupportRequestManagerFragment(@NonNull ActivityFragmentLifecycle lifecycle) {
        this.lifecycle = lifecycle;
    }
    //提供设置注入requestManger方法
    public void setRequestManager(@Nullable RequestManager requestManager) {
        this.requestManager = requestManager;
    }
    //提供获取当前fragment的lifecycle
    ActivityFragmentLifecycle getGlideLifecycle() {
        return lifecycle;
    }
    //获取requestManager方法
    public RequestManager getRequestManager() {
        return requestManager;
    }
  ......非生命周期相关代码省略....
    //以下fragment本身生命周期方法的回调(这个不用多说，系统级别的api已经帮我们实现了这个回调)
    @Override
    public void onStart() {
        super.onStart();
        //回调glide的生命周期组件lifecycle.onStart
        lifecycle.onStart();
    }
    @Override
    public void onStop() {
        super.onStop();
        //回调glide的生命周期组件lifecycle.onStop
        lifecycle.onStop();
    }
    @Override
    public void onDestroy() {
        super.onDestroy();
        lifecycle.onDestroy();
        //回调glide的生命周期组件lifecycle.onDestroy
        unregisterFragmentWithRoot();
    }
  .....
}

```
- ActivityFragmentLifecycle 是从  supportFragmentGet() 方法中 构建 RequestManager 时传入的 RequestManagerFragment#getGlideLifecycle()
- ActivityFragmentLifecycle 实现了Lifecycle接口，并提供了注册LifecycleListener的方法
```java
class ActivityFragmentLifecycle implements Lifecycle {
  private final Set<LifecycleListener> lifecycleListeners =
      Collections.newSetFromMap(new WeakHashMap<LifecycleListener, Boolean>());
  ...
  @Override
  public void addListener(@NonNull LifecycleListener listener) {
    ....
  }
  @Override
  public void removeListener(@NonNull LifecycleListener listener) {
    ....
  }
  // lifecycle.onStart()方法的调用在这
  void onStart() {
    isStarted = true;
    for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {     //回调了Glide中lifecycleListener的onStart
      lifecycleListener.onStart();
    }
  }
  ....onStop、onDestory...
}
```
- LifecycelListener 真正监听生命周期事件接口，触发回调
```java
/** 一个监听接口，监听Activity和fragment的生命周期事件 */
public interface Lifecycle {
    void addListener(@NonNull LifecycleListener listener);
    void removeListener(@NonNull LifecycleListener listener);
}

/**
 * 真正的监听生命周期事件接口，有onStart、onStop、onDestroy
 */
public interface LifecycleListener {
  void onStart();
  void onStop();
  void onDestroy();
}

```

### 假如让你自己写个图片加载框架，你会考虑哪些问题？
#### 1.异步加载：线程池；由于网络会阻塞，所以读内存和硬盘可以放在一个线程池，网络需要另外一个线程池，网络也可以采用Okhttp内置的线程池。
#### 2.切换线程：Handler，无论是RxJava、EventBus，还是Glide，只要是想从子线程切换到Android主线程，都离不开Handler。
#### 3.缓存
- LruCache、DiskLruCache 内存缓存：LruCache(最新数据始终在LinkedHashMap最后一个)
- LruCache中维护了一个集合LinkedHashMap，该LinkedHashMap是以访问顺序排序的。
- 当调用get()方法访问缓存对象时，就会调用LinkedHashMap的get()方法获得对应集合元素，同时会更新该元素到队尾。
- 当调用put()方法时，就会在结合中添加元素，并调用trimToSize()判断缓存是否已满，如果满了就用LinkedHashMap的迭代器删除队首元素，即近期最少访问的元素。
- 磁盘缓存 DiskLruCache(DiskLruCache会自动生成journal文件，这个文件是日志文件，主要记录的是缓存的操作)
- DiskLruCache和LruCache内部都是使用了LinkedHashMap去实现缓存算法的，只不过前者针对的是将缓存存在本地，而后者是直接将缓存存在内存。
#### 4.防止OOM
- 软引用、LruCache、图片压缩、Bitmap像素存储位置
- 软引用：LruCache里存的是软引用对象，那么当内存不足的时候，Bitmap会被回收，也就是说通过SoftReference修饰的Bitmap就不会导致OOM。
- Bitmap被回收的时候，LruCache剩余的大小应该重新计算，可以写个方法，当Bitmap取出来是空的时候，LruCache清理一下，重新计算剩余内存；
- onLowMemory：当内存不足的时候，Activity、Fragment会调用onLowMemory方法，可以在这个方法里去清除缓存，Glide使用的就是这一种方式来防止OOM。
- Bitmap 像素存储位置考虑：Android 3.0到8.0 之间Bitmap像素数据存在Java堆，而8.0之后像素数据存到native堆中
- 内存泄露：注意ImageView的正确引用，生命周期管理，通过给Activity添加自定义Fragment方式，监听生命周期，在Activity/fragment 销毁的时候，取消图片加载任务

### 参考致谢
- https://blog.csdn.net/hymking/article/details/122134041
- https://blog.csdn.net/a130226123/article/details/119325449#load_677
### 深入理解glide缓存机制
- https://blog.csdn.net/u010142437/article/details/80434503




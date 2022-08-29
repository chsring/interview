## 简单介绍
- SharedPreferences(以下统称为sp)是Android提供的数据持久化的一种手段，适合单进程、小批量的数据存储与访问
- 由于sharedPreferences是基于xml文件实现的，所有持久化数据都是一次性加载，如果数据过大是不适合采用SP存放。
- 实际上是用xml文件存放数据，文件存保存放在/data/data//shared_prefs/

### 实现原理
#### 基本使用
```java
   mSharedPreferences = context.getSharedPreferences("test", Context.MODE_PRIVATE);
    SharedPreferences.Editor editor = mSharedPreferences.edit();
    editor.putString(key, value);
    editor.commit(); // 线程安全，性能慢，同步操作，在当前线程完成写文件操作
    editor.apply();  // 线程不安全的，性能高，是异步处理IO操作
```
#### context.getSharedPreferences其实就是简单的调用ContextImpl的getSharedPreferences，具体实现如下：
````java
@Override
public SharedPreferences getSharedPreferences(String name, int mode) {
    SharedPreferencesImpl sp;
    synchronized (ContextImpl.class) {
        if (sSharedPrefs == null) {
            sSharedPrefs = new ArrayMap<String, ArrayMap<String, SharedPreferencesImpl>>();
        }

        final String packageName = getPackageName();
        ArrayMap<String, SharedPreferencesImpl> packagePrefs = sSharedPrefs.get(packageName);
        if (packagePrefs == null) {
            packagePrefs = new ArrayMap<String, SharedPreferencesImpl>();
            sSharedPrefs.put(packageName, packagePrefs);
        }
        sp = packagePrefs.get(name);
        if (sp == null) {
        <!--读取文件-->
            File prefsFile = getSharedPrefsFile(name);
            sp = new SharedPreferencesImpl(prefsFile, mode);
            <!--缓存sp对象-->
            packagePrefs.put(name, sp);
            return sp;
        }
    }
    <!--跨进程同步问题-->
    if ((mode & Context.MODE_MULTI_PROCESS) != 0 ||
        getApplicationInfo().targetSdkVersion < android.os.Build.VERSION_CODES.HONEYCOMB) {
        sp.startReloadIfChangedUnexpectedly();
    }
    return sp;
}
````
- SharePreference 先去内存中查询与xml对应的SharePreferences是否已经被创建加载，如果没有那么该创建就创建，然后加载，在加载之后，要将所有的key-value保存到内存中。
#### 数据加载是new SharedPreferencesImpl 对象时候开始的
```java
 SharedPreferencesImpl(File file, int mode) {
    mFile = file;
    mBackupFile = makeBackupFile(file);
    mMode = mode;
    mLoaded = false;
    mMap = null;
    startLoadFromDisk();
}
```
#### startLoadFromDisk很简单，就是读取xml配置，如果其他线程想要在读取之前就是用的话，就会被阻塞，一直wait等待，直到数据读取完成。
````java
 private void loadFromDiskLocked() {
   ...
    Map map = null;
    StructStat stat = null;
    try {
        stat = Os.stat(mFile.getPath());
        if (mFile.canRead()) {
            BufferedInputStream str = null;
            try {
    <!--读取xml中配置-->
                str = new BufferedInputStream(
                        new FileInputStream(mFile), 16*1024);
                map = XmlUtils.readMapXml(str);
            }...
    mLoaded = true;
    ...
    <!--唤起其他等待线程-->
    notifyAll();
}
````
#### 使用xml解析工具XmlUtils，直接在当前线程读取xml文件，所以，如果xml文件稍大，尽量不要在主线程读取，读取完成之后，xml中的配置项都会被加载到内存，再次访问的时候，其实访问的是内存缓存。
### 数据的更新
- Editor是一个接口，这里的实现是一个EditorImpl对象，它首先批量预处理更新操作，之后再提交更新，在提交事务的时候有两种方式，一种是apply，另一种commit，两者的区别在于：何时将数据持久化到xml文件，前者是异步的，后者是同步的。
- Google推荐使用前一种，因为，就单进程而言，只要保证内存缓存正确就能保证运行时数据的正确性，而持久化，不必太及时，这种手段在Android中使用还是很常见的，比如权限的更新也是这样，况且，Google并不希望SharePreferences用于多进程，因为不安全
#### 两者源码
```java
public void apply() {
    <!--添加到内存-->
        final MemoryCommitResult mcr = commitToMemory();
        final Runnable awaitCommit = new Runnable() {
                public void run() {
                    try {
                        mcr.writtenToDiskLatch.await();
                    } catch (InterruptedException ignored) {
                    }
                }
            };

        QueuedWork.add(awaitCommit);
        Runnable postWriteRunnable = new Runnable() {
                public void run() {
                    awaitCommit.run();
                    QueuedWork.remove(awaitCommit);
                }
            };
        <!--延迟写入到xml文件-->
        SharedPreferencesImpl.this.enqueueDiskWrite(mcr, postWriteRunnable);
        <!--通知数据变化-->
        notifyListeners(mcr);
    }
 
 public boolean commit() {
        MemoryCommitResult mcr = commitToMemory();
        SharedPreferencesImpl.this.enqueueDiskWrite(
            mcr, null /* sync write on this thread okay */);
        try {
            mcr.writtenToDiskLatch.await();
        } catch (InterruptedException e) {
            return false;
        }
        notifyListeners(mcr);
        return mcr.writeToDiskResult;
    }     
```
#### 从上面可以看出两者最后都是先调用commitToMemory，将更改提交到内存，在这一点上两者是一致的，之后又都调用了enqueueDiskWrite进行数据持久化任务，不过commit函数一般会在当前线程直接写文件，而apply则提交一个事务到已给线程池，之后直接返回，实现如下：
```java
private void enqueueDiskWrite(final MemoryCommitResult mcr,
                              final Runnable postWriteRunnable) {
    final Runnable writeToDiskRunnable = new Runnable() {
            public void run() {
                synchronized (mWritingToDiskLock) {
                    writeToFile(mcr);
                }
                synchronized (SharedPreferencesImpl.this) {
                    mDiskWritesInFlight--;
                }
                if (postWriteRunnable != null) {
                    postWriteRunnable.run();
                }
            }
        };
   final boolean isFromSyncCommit = (postWriteRunnable == null);
    if (isFromSyncCommit) {
        boolean wasEmpty = false;
        synchronized (SharedPreferencesImpl.this) {
            wasEmpty = mDiskWritesInFlight == 1;
        }
        <!--如果没有其他线程在写文件，直接在当前线程执行-->
        if (wasEmpty) {
            writeToDiskRunnable.run();
            return;
        }
    }
   QueuedWork.singleThreadExecutor().execute(writeToDiskRunnable);
}
```
#### 其实简单来说，如果硬要说两者的区别的话，直接说commit同步，apply异步也不是没有问题，坦白来说，sp不适用于多线程
### SP的多线程
- 其实在sp创建的时候可以指定的加载模式中有个MODE_MULTI_PROCESS，它是Google提供的一个在多线程模式。但是这种模式并不是我们说的支持多进程同步更新等，它的作用只会在getSharedPreferences的时候，才会重新从xml重加载，如果我们在一个进程中更新xml，但是没有通知另一个进程，那么另一个进程的SharePreferences是不会自动更新的
```java
@Override
public SharedPreferences getSharedPreferences(String name, int mode) {
    SharedPreferencesImpl sp;
    ...
    if ((mode & Context.MODE_MULTI_PROCESS) != 0 ||
        getApplicationInfo().targetSdkVersion < android.os.Build.VERSION_CODES.HONEYCOMB) {
        // If somebody else (some other process) changed the prefs
        // file behind our back, we reload it.  This has been the
        // historical (if undocumented) behavior.
        sp.startReloadIfChangedUnexpectedly();
    }
    return sp;
}
```
#### MODE_MULTI_PROCESS只是个花瓶，对于多线程的支持几乎为0，简单来说，最好不需要用


### 总结
- 1.SharePreferences是Android基于xml实现的一种数据持久化手段
- 2.SharePreferences不支持多进程
- 3.SharePreferences的commit与apply一个是同步一个是异步（大部分场景下）
- 4.不要使用SharePreferences存储太大的数据
- 5.不要使用SharePreferences进行使用多线程调用

### 参考致谢
- 作者：RainyJiang
- https://juejin.cn/post/6926825172066369544
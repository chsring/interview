## JVM
- 编译流程：编译器 将java文件 编译成class文件，交给jvm运行
### 类加载机制
- 虚拟机把class文件加载到内存中，并对数据进行校验，转换解析和初始化，形成可以虚拟机直接使用的java类型即.class
![img.png](resouse/类加载机制.png)
### 类加载过程
#### 装载
- 通过一个类的限定名获取这个类的二进制字节流；
- 将这个字节流所代表的静态存储结构转换为方法区运行时数据结构；
- 在java堆中生成一个代表这个类的class对象，这个对象作为我们访问方法区的数据访问入口。
#### 链接
- 1.验证：保证我们加载类的正确性（文件格式，源数据，字节码，符号引用）；
- 2.准备：为类的静态变量分配内存，并将其初始化为当前类型的默认值；static int a = 1；那么它在这个准备阶段 a=0；
- 3.解析：把类中的符号引用转换成直接引用。
#### 初始化
- 为静态变量赋值，初始化静态代码块，初始化当前类的父类
### 类加载器层级
![img.png](resouse/类加载器.png)
### 双亲委派机制（父类委派机制）
![img.png](resouse/双亲委派机制.png)
- 在源码中已经存在了java.lang.String;但是我有自己写了个java.lang.String，这是应该加载哪个？优先加载源码中的String
```java
    // ClassLoader中的loadClass方法
    protected Class<?> loadClass(String name, boolean resolve)throws ClassNotFoundException{
        synchronized (getClassLoadingLock(name)) {
            // 判断是否加载过，如果加载过,直接拿过来用；否则走下面
            Class<?> c = findLoadedClass(name); //类加载机制中的缓存机制
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) { // 递归调用父类的 loadClass。双亲委派 本质依赖于本层递归。
                        c = parent.loadClass(name, false);
                    } else { // 如果递归到
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```
### 打破双亲委派机制
- 继承classload类，重写loadclass方法，把  c = parent.loadClass(name, false) 去掉
- SPI机制：Service Provider Interface 服务提供接口；通过接口，将这些装配的控制权移到外部，可以随时替换实现。
- OSGI：它支持热部署，热更新的。

### 运行时数据区
![img.png](resouse/运行时数据区.png)
- 运行时数据区是运行时的一块内存区域，划分为上图五块内容。
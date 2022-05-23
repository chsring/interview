### final
- final是用于修饰类、成员变量和成员方法，类不可被继承，成员变量不可变，成员方法不可被重写

### finally
- finally与try…catch…共同使用，确保无论是否出现异常都能被调用到；

### finalize
- finalize是类的方法，垃圾回收前会调用此方法，子类可以重写finalize方法实现对资源的回收

### Serlizable
- 是java序列化接口，在硬盘上读写，读写的过程中有大量临时变量产生，内部执行大量的I/O操作，效率很低；

### Parcelable
- 是Android序列化接口，效率高，在内存中读写，但使用麻烦，对象不能保存到磁盘中。
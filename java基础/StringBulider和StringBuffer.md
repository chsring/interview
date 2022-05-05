### String
- String是final修饰的，不可变，每次操作都会产生新的String对象

#### StringBuffer和StringBuilder
- StringBuffer和StringBuilder都是在原对象上操作
- StringBuffer是线程安全的，StringBuilder线程不安全的
- StringBuffer方法都是synchronized修饰的
- 性能：StringBuilder > StringBuffer > String
- 在不考虑多线程的情况下，优先使用StringBuilder，多线程使用共享变量时使用StringBuffer
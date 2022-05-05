### == 
- == 对比的是栈中的值，基本数据类型是变量值，引用类型是堆中内存对象的地址。

### Object中equals()方法
- equals：object类中默认也是采用==比较，通常会重写（为了比较成员变量中的值）。如果不作处理，跟 == 没什么差别，比较内存地址是否一样。

```java
public boolean equals(Object obj) {
        return (this == obj);
    }
```

### hashCode()
- Object中的方法，将对象的内存地址转化成整数之后返回。
- 约定：如果两个对象相等，必须具备相同的哈希码；如果两个对象哈希码想等，则两个对象可能相等，可能不想等。
 
### String 中重写了equals()方法
- 字符串的equals()是对值进行对比，两个不同地址存放的相同的字符串，equals()方法返回true

### 为什么重写equals()的时候还要重写hashCode()
- 由于 hashCode() 中的约定
- 规定：如果两个对象调用equals方法返回true，那么两个对象的hashCode必须返回相同的整数。
```java
String s1 = new String("hello");
String s2 = new String("hello");
    s1.euqals(s2); //返回true
    //但是 s1.hashCode() 不一定等于 s2.hashCode() 就违背了规定。
// 所以要对hashCode方法进行重写。因此String类中不仅重写了equals 还重写了 hashCode
```
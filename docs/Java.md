### 框架合集

##### 为什么HashMap要重写`hashcode()` 和 `equals()`方法？

ps：不同的jvm可能是不同的，这里分析的是jdk8 hotspot，并且文中的对象相同都是指的对象内容相同。

hash的作用，快速判断一个值相等的可能性。

根据value产生的hash值：

- 如果value值相同，hash值一定相同
- 如果value值不相同，hash值一定不相同
- 如果hash值相同，value值不一定相同
- 如果hash值不同，value值一定不同

首先，Java提供的hashcode是jvm根据内存地址计算的产生的`int`类型整数，并不是一个内存地址，通常情况下是可以看作唯一值的。

```java
public static native int identityHashCode(Object x);
```

HashMap中比较的是自己计算的hash值，而不是jvm提供的hashCode()，要保证对象具有相同内容的值具有相同的hash值，而jvm提供的hashCode是不具备的。

```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

HashMap中`Node<K,V>`的`equals()`方法实质上是调用的是`Map<K,V>`中对象K的equals方法，如果有重写，没有重写则是调用Object中的equals方法

```java
//`Node<K,V>`的`equals()`方法,用于比较两个对象是否相等，具体比较要看对象K中重写的equals方法的比较逻辑
public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
}  

//Objects.java中的方法，主要原因是HashMap中的key可能是null，避免有空指针异常
//主要做两个比较：
//1、是否为null 2、两个对象的地址是否相等
public static boolean equals(Object a, Object b) {
    return (a == b) || (a != null && a.equals(b));
}
```



HashMap的put方法中，会对存入的对象key的hash码、内存地址和对象equals方法进行多重比较，判断是否具有相同的对象key。

```Java
//利用hash码不同值一定不同的特点，快速判断key相等的可能性，因为对象的equals比较方法可能会比较复杂
//如果hash码相同，判断内存地址是否相同，如果不相同则继续调用equals()方法比较两个对象
if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
```

- 如果对象key的equals方法不重写，那么`(k = p.key) == key || (key != null && key.equals(k))`比较的都是对象的地址，结果是false，会有多个相同的对象作为key
- 如果对象key不重写hashCode，`p.hash == hash `比较的是jvm返回的hashCode，结果是false，同样会有多个相同的对象作为key

注意：**

>HashMap中的key一定是不可变的


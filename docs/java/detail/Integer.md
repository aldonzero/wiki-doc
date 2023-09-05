Integer 是基本类型int 的包装对象。

### Integer.MAX_VALUE + 1是多少

int 是4字节32位，最高位表示正负数。

```java
// A constant holding the minimum value an int can have, -2^31.    
@Native public static final int   MIN_VALUE = 0x80000000;
// A constant holding the maximum value an {@code int} can have, 2^31 - 1.
@Native public static final int   MAX_VALUE = 0x7fffffff;

//二进制表示
MAX_VALUE = 01111111111111111111111111111111
MIN_VALUE = 10000000000000000000000000000000
```

因此，MAX_VALUE + 1 = MIN_VALUE
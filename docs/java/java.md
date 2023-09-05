## 1 基础

## 1.1 关键字

#### 1.1.1  访问控制修饰符

- **default** (即**默认，什么也不写**）: 在**同一包内可见**，不使用任何修饰符。使用对象：类、接口、变量、方法。
- **private** : 在**同一类内可见**。使用对象：变量、方法。 **注意：不能修饰类（外部类）**
- **public** : 对所有类可见。使用对象：类、接口、变量、方法
- **protected** : 对**同一包内的类和所有子类**可见。使用对象：变量、方法。 **注意：能修饰类（内部类除外）**。

#### 1.1.2 default关键字

default关键字的三种用法：

1. `switch`中case没有匹配执行的默认方法；

   ```java
   switch (a){
       case 1:
           System.out.println("a is :" + 1);
           break;
       default:
           System.out.println("The parameter does not conform");
           break;
   }

   ```

2. 接口中使用`default`关键字修饰方法，方法必须添加方法体；

   ```java
   interface IBase {
       default void fun4(){} 
   }
   ```

3. 在注解中指定元素的默认值；

   ```java
   @Target(ElementType.METHOD)
   @Retention(RetentionPolicy.RUNTIME)
   public @interface TestAnnotation {
     public int id();
     public String description() default "No description";
   }
   ```

   ​

#### 1.1.3 static关键字

static 修饰符，用来修饰**类方法和类变量**。

类的加载过程有几个阶段：加载 -> 验证 -> 准备 ->  解析 -> 初始化

其中在准备阶段会为类变量分配内存和初始值，其中如果变量是static变量是final修饰的基本类型和字符串常量，会在准备阶段显示赋值。

在初始化阶段会执行类构造器方法`<clinit>`方法，为静态类变量显示赋值并执行静态代码块。

#### 1.1.4  final关键字

**final 变量：**

final 表示"最后的、最终的"含义，变量一旦赋值后，不能被重新赋值。被 final 修饰的实例变量必须显式指定初始值。

final 修饰符通常和 static 修饰符一起使用来创建类常量。

**final 方法**

父类中的 final 方法可以被子类继承，但是不能被子类重写。

声明 final 方法的主要目的是防止该方法的内容被修改。

**final 类**

final 类不能被继承，没有类能够继承 final 类的任何特性。



#### 1.1.5 this

this 关键字的作用：

* this 关键字代表了当前对象的引用
* this 出现在方法中：**哪个对象调用这个方法 this 就代表谁**
* this 可以出现在构造器中：代表构造器正在初始化的那个对象
* this 可以区分变量是访问的成员变量还是局部变量



#### 1.1.6 instanceof

instanceof：判断左边的对象是否是右边的类的实例，或者是其直接或间接子类，或者是其接口的实现类



### 1.2 抽象类和接口

JDK1.8比较。

#### 1.2.1 抽象类

抽象类是对一类具有相同特性的事务进行抽象，包括成员和方法；可以作为很多子类的父类，是一种模板模式设计。

**语法：**

1. 抽象类需要使用`abstract`关键字，可以用来修饰类、方法，不能用来修饰变量、代码块、构造器；

2. `abstract`修饰的方法不能有方法体；

3. `abstract`不能与`private、static、final、native`关键字联用，因为`abstract`关键字修饰的方法子类需要重写，而`private、static、final、native`关键字修饰的方法是不能被重写的；

   被 static 修饰的方法属于类，是类自己的东西，不是给子类来继承的，而抽象方法本身没有实现，就是用来给子类继承

4. 抽象类可以有构造函数，但是**不能被实例化**；从设计的角度看，抽象类本身是对一类事物的抽象，而实例化是一个具体事物对象，那么抽象类实例化在设计上是不合理的。

5. 子类只能继承一个父类。

#### 1.2.2 接口

接口是对类的局部行为抽象；在设计层面上是对一类事务行为的规范，是一种辐射设计。

**语法：**

1. 接口使用类使用`interface`修饰；
2. 接口中除了`static、final`变量，不能包含成员变量，并且变量只能使用不能被修改，因为接口中的变量会被隐式指定为`public static final `变量；
3. 接口中的方法只能使用`public、default`关键字或者不用关键字修饰方法，使用`default`关键字修饰的方法必须有方法体；
4. 不可以有构造函数，也不可以实例化；
5. 实现类可以实现多个接口；



**实现多个接口的使用注意事项：**

1. 当一个类实现多个接口时，多个接口中存在同名的静态方法并不会冲突，只能通过各自接口名访问静态方法
2. 当一个类实现多个接口时，多个接口中存在同名的默认方法，实现类必须重写这个方法
3. 当一个类既继承一个父类，又实现若干个接口时，父类中成员方法与接口中默认方法重名，子类**就近选择执行父类**的成员方法
4. 接口中，没有构造器，**不能创建对象**，接口是更彻底的抽象，连构造器都没有，自然不能创建对象

新增功能

jdk1.8 以后新增的功能：

* 默认方法（就是普通实例方法），必须用 default 修饰，必须有默认实现，必须用接口的实现类的对象来调用；
* 静态方法，默认会 public 修饰，必须有默认实现，接口的静态方法必须用接口的类名本身来调用；

#### 对比抽象类

| **参数**      | **抽象类**                                  | **接口**                                   |
| ----------- | ---------------------------------------- | ---------------------------------------- |
| 默认的方法实现     | 可以有默认的方法实现                               | 接口完全是抽象的，jdk8 以后有默认的实现                   |
| 实现          | 子类使用 **extends** 关键字来继承抽象类。如果子类不是抽象类的话，它需要提供抽象类中所有声明的方法的实现。 | 子类使用关键字 **implements** 来实现接口。它需要提供接口中所有声明的方法的实现 |
| 构造器         | 抽象类可以有构造器                                | 接口不能有构造器                                 |
| 与正常Java类的区别 | 除了不能实例化抽象类之外，和普通 Java 类没有任何区别            | 接口是完全不同的类型                               |
| 访问修饰符       | 抽象方法有 **public**、**protected** 和 **default** 这些修饰符 | 接口默认修饰符是 **public**，别的修饰符需要有方法体          |
| main方法      | 抽象方法可以有 main 方法并且我们可以运行它                 | jdk8 以前接口没有 main 方法，不能运行；jdk8 以后接口可以有 default 和 static 方法，可以运行 main 方法 |
| 多继承         | 抽象方法可以继承一个类和实现多个接口                       | 接口可以继承一个或多个其它接口，接口不可继承类                  |
| 速度          | 比接口速度要快                                  | 接口是稍微有点慢的，因为它需要时间去寻找在类中实现的方法             |
| 添加新方法       | 如果往抽象类中添加新的方法，可以给它提供默认的实现，因此不需要改变现在的代码   | 如果往接口中添加方法，那么必须改变实现该接口的类                 |



### 1.3  上下转型

  Java 不能隐式执行**向下转型**，因为这会使得精度降低，但是可以向上转型

* float 与 double：

  ```java
  //1.1字面量属于double类型，不能直接将1.1直接赋值给 float 变量，因为这是向下转型
  float f = 1.1;//报错
  //1.1f 字面量才是 float 类型
  float f = 1.1f;
  ```

  ```java
  float f1 = 1.234f;
  double d1 = f1;

  double d2 = 1.23;
  float f2 = (float) d2;//向下转型需要强转
  ```

  ```java
  int i1 = 1245;
  long l1 = i1;

  long l2 = 1234;
  int i2 = (int) l2;
  ```

* 隐式类型转换：

  字面量 1 是 int 类型，比 short 类型精度要高，因此不能隐式地将 int 类型向下转型为 short 类型

  **使用 += 或者 ++ 运算符会执行类型转换**：

  ```java
  short s1 = 1;
  s1 += 1;	//s1++;
  //上面的语句相当于将 s1 + 1 的计算结果进行了向下转型
  s1 = (short) (s1 + 1);
  ```



### 1.4 类型对比

* 有了基本数据类型，为什么还要引用数据类型？

  > 引用数据类型封装了数据和处理该数据的方法，比如 Integer.parseInt(String) 就是将 String 字符类型数据转换为 Integer 整型
  >
  > Java 中大部分类和方法都是针对引用数据类型，包括泛型和集合

* 引用数据类型那么好，为什么还用基本数据类型？

  > 引用类型的对象要多储存对象头，对基本数据类型来说空间浪费率太高。逻辑上来讲，Java 只有包装类就够了，为了运行速度，需要用到基本数据类型；优先考虑运行效率的问题，所以二者同时存在是合乎情理的

* Java 集合不能存放基本数据类型，只存放对象的引用？

  > 不能放基本数据类型是因为不是 Object 的子类。泛型思想，**如果不用泛型要写很多参数类型不同的但功能相同的函数（方法重载）**

* ==

  > == 比较基本数据类型：比较的是具体的值
  > == 比较引用数据类型：比较的是对象地址值


***

### 1.5 运算

#### 1.5.1 i++ 与 ++i 的区别？

i++ 表示先将 i 放在表达式中运算，然后再加 1，++i 表示先将 i 加 1，然后再放在表达式中运算

#### 1.5.2 位运算

- 异或 ^：两位相异为 1，相同为 0，又叫不进位加法
- 同或：两位相同为 1，相异为 0

#### 1.5.3 移位运算

- 计算机里一般用**补码表示数字**，正数、负数的表示区别就是最高位是 0 还是 1

  - 正数的原码反码补码相同，最高位为 0

    ```java
    100:	00000000  00000000  00000000  01100100
    ```

  - 负数：
    原码：最高位为 1，其余位置和正数相同
    反码：保证符号位不变，其余位置取反
    补码：保证符号位不变，其余位置取反后加 1，即反码 +1

    ```java
    -100 原码:	10000000  00000000  00000000  01100100	//32位
    -100 反码:	11111111  11111111  11111111  10011011
    -100 补码:	11111111  11111111  11111111  10011100
    ```

    补码 → 原码：符号位不变，其余位置取反加 1

  运算符：

  - `>>` 运算符：将二进制位进行右移操作，相当于除 2
  - `<<` 运算符：将二进制位进行左移操作，相当于乘 2
  - `>>>` 运算符：无符号右移，忽略符号位，空位都以 0 补齐

  运算规则：

  - 正数的左移与右移，空位补 0

  - 负数原码的左移与右移，空位补 0

    负数反码的左移与右移，空位补 1

    负数补码，左移低位补 0（会导致负数变为正数的问题，因为移动了符号位），右移高位补 1

  - 无符号移位，空位补 0



#### 1.5.4 switch

从 Java 7 开始，可以在 switch 条件判断语句中使用 String 对象

switch 不支持 long、float、double，switch 的设计初衷是对那些只有少数几个值的类型进行等值判断，如果值过于复杂，那么用 if 比较合适



## 2 枚举

枚举是 Java 中的一种特殊类型，为了做信息的标志和信息的分类

定义枚举的格式：

```java
修饰符 enum 枚举名称{
	第一行都是罗列枚举实例的名称。
}
```

枚举的特点：

* 枚举类是用 final 修饰的，枚举类不能被继承
* 枚举类默认继承了 java.lang.Enum 枚举类
* 枚举类的第一行都是常量，必须是罗列枚举类的实例名称
* 枚举类相当于是多例设计模式
* 每个枚举项都是一个实例，是一个静态成员变量

| 方法名                                      | 说明                 |
| ---------------------------------------- | ------------------ |
| String name()                            | 获取枚举项的名称           |
| int ordinal()                            | 返回枚举项在枚举类中的索引值     |
| int compareTo(E  o)                      | 比较两个枚举项，返回的是索引值的差值 |
| String toString()                        | 返回枚举常量的名称          |
| static <T> T  valueOf(Class<T> type,String  name) | 获取指定枚举类中的指定名称的枚举值  |
| values()                                 | 获得所有的枚举项           |

* 源码分析：

  ```java
  enum Season {
      SPRING , SUMMER , AUTUMN , WINTER;
  }
  // 枚举类的编译以后源代码：
  public final class Season extends java.lang.Enum<Season> {
  	public static final Season SPRING = new Season();
  	public static final Season SUMMER = new Season();
  	public static final Season AUTUMN = new Season();
  	public static final Season WINTER = new Season();

  	public static Season[] values();
  	public static Season valueOf(java.lang.String);
  }
  ```





## 3 数组

### 2.1 初始化

数组就是存储数据长度固定的容器，存储多个数据的数据类型要一致，**数组也是一个对象**

#### 2.1.1 创建数组：

* 数据类型[] 数组名：`int[] arr`  （常用）
* 数据类型 数组名[]：`int arr[]`

#### 2.1.2 静态初始化：

* 数据类型[] 数组名 = new 数据类型[]{元素1,元素2,...}：`int[] arr = new int[]{11,22,33}`
* 数据类型[] 数组名 = {元素1,元素2,...}：`int[] arr = {44,55,66}`

#### 2.1.3 动态初始化

* 数据类型[] 数组名 = new 数据类型[数组长度]：`int[] arr = new int[3]`




### 2.2 元素访问

* **索引**：每一个存储到数组的元素，都会自动的拥有一个编号，从 **0** 开始。这个自动编号称为数组索引（index），可以通过数组的索引访问到数组中的元素
* **访问格式**：数组名[索引]，`arr[0]`
* **赋值：**`arr[0] = 10`




### 2.3 内存分配

内存是计算机中的重要器件，临时存储区域，作用是运行程序。编写的程序是存放在硬盘中，在硬盘中的程序是不会运行的，必须放进内存中才能运行，运行完毕后会清空内存，Java 虚拟机要运行程序，必须要对内存进行空间的分配和管理

| 区域名称  | 作用                               |
| ----- | -------------------------------- |
| 寄存器   | 给 CPU 使用                         |
| 本地方法栈 | JVM 在使用操作系统功能的时候使用               |
| 方法区   | 存储可以运行的 class 文件                 |
| 堆内存   | 存储对象或者数组，new 来创建的，都存储在堆内存        |
| 方法栈   | 方法运行时使用的内存，比如 main 方法运行，进入方法栈中执行 |



### 2.4 数组异常

* 索引越界异常：ArrayIndexOutOfBoundsException 

* 空指针异常：NullPointerException 

  ```java
  public class ArrayDemo {
      public static void main(String[] args) {
          int[] arr = new int[3];
          //把null赋值给数组
          arr = null;
          System.out.println(arr[0]);
      }
  }
  ```

  arr = null，表示变量 arr 将不再保存数组的内存地址，也就不允许再操作数组，因此运行的时候会抛出空指针异常。在开发中，空指针异常是不能出现的，一旦出现了，就必须要修改编写的代码

  解决方案：给数组一个真正的堆内存空间引用即可

  ​

### 2.5 二维数组

二维数组也是一种容器，不同于一维数组，该容器存储的都是一维数组容器

#### 2.5.1 初始化

* 动态初始化：数据类型[][] 变量名 = new 数据类`[m][n]`，`int[][] arr = new int[3][3]`

  * m 表示这个二维数组，可以存放多少个一维数组，行
  * n 表示每一个一维数组，可以存放多少个元素，列
* 静态初始化
  * 数据类型[][] 变量名 = new 数据类型 [][]{{元素1, 元素2...} , {元素1, 元素2...} 
  * 数据类型[][] 变量名 = {{元素1, 元素2...}, {元素1, 元素2...}...}
  * `int[][] arr = {{11,22,33}, {44,55,66}}`


## 4 集合

### 4.1 集合概述

集合是一个大小可变的容器，容器中的每个数据称为一个元素

集合特点：类型可以不确定，大小不固定；集合有很多，不同的集合特点和使用场景不同

数组：类型和长度一旦定义出来就都固定

作用：

* 在开发中，很多时候元素的个数是不确定的
* 而且经常要进行元素的增删该查操作，集合都是非常合适的，开发中集合用的更多


***



### 4.2 存储结构

数据结构指的是数据以什么方式组织在一起，不同的数据结构，增删查的性能是不一样的

数据存储的常用结构有：栈、队列、数组、链表和红黑树

* 队列（queue）：先进先出，后进后出。(FIFO first in first out)

* 栈（stack）：后进先出，先进后出 （LIFO）

* 数组：数组是内存中的连续存储区域，分成若干等分的小区域（每个区域大小是一样的）元素存在索引

  特点：**查询元素快**（根据索引快速计算出元素的地址，然后立即去定位），**增删元素慢**（创建新数组，迁移元素）

* 链表：元素不是内存中的连续区域存储，元素是游离存储的，每个元素会记录下个元素的地址
  特点：**查询元素慢，增删元素快**（针对于首尾元素，速度极快，一般是双链表）

* 树：

  * 二叉树：binary tree 永远只有一个根节点，是每个结点不超过2个节点的树（tree） 

    特点：二叉排序树：小的左边，大的右边，但是可能树很高，性能变差，为了做排序和搜索会进行左旋和右旋实现平衡查找二叉树，让树的高度差不大于1

  * 红黑树（基于红黑规则实现自平衡的排序二叉树）：树保证到了很矮小，但是又排好序，性能最高的

    特点：**红黑树的增删查改性能都好**



### 4.3 Collection

#### 4.3.1 概述

Java 中集合的代表是 Collection，Collection 集合是 Java 中集合的祖宗类

Collection 集合底层为数组：`[value1, value2, ....]`

```java
Collection集合的体系:
                      Collection<E>(接口)
                 /                         \
          Set<E>(接口)                    List<E>(接口)
      /               \                  /             \
 HashSet<E>(实现类) TreeSet<>(实现类)  ArrayList<E>(实现类)  LinekdList<>(实现类)
 /
LinkedHashSet<>(实现类)
```

**集合的特点：**

* Set 系列集合：添加的元素是无序，不重复，无索引的
  * HashSet：添加的元素是无序，不重复，无索引的
  * LinkedHashSet：添加的元素是有序，不重复，无索引的
  * TreeSet：不重复，无索引，按照大小默认升序排序
* List 系列集合：添加的元素是有序，可重复，有索引
  * ArrayList：添加的元素是有序，可重复，有索引
  * LinekdList：添加的元素是有序，可重复，有索引



#### 4.3.2 遍历

Collection 集合的遍历方式有三种:

集合可以直接输出内容，因为底层重写了 toString() 方法

1. 迭代器

   * `public Iterator iterator()`：获取集合对应的迭代器，用来遍历集合中的元素的
   * `E next()`：获取下一个元素值
   * `boolean hasNext()`：判断是否有下一个元素，有返回 true ，反之返回 false
   * `default void remove()`：从底层集合中删除此迭代器返回的最后一个元素，这种方法只能在每次调用 next() 时调用一次

2. 增强 for 循环：可以遍历集合或者数组，遍历集合实际上是迭代器遍历的简化写法

   ```java
   for(被遍历集合或者数组中元素的类型 变量名称 : 被遍历集合或者数组){
   }
   ```

   缺点：遍历无法知道遍历到了哪个元素了，因为没有索引

3. JDK 1.8 开始之后的新技术 Lambda 表达式

   ```java
   //lambda表达式
    lists.forEach(s -> {
        System.out.println(s);
    });
   ```




### 4.4 List


####  4.4.1  ArrayList

##### 4.4.4.1 介绍

ArrayList 添加的元素，是有序，可重复，有索引的



##### 4.4.4.2 底层实现

ArrayList 实现类集合底层**基于数组存储数据**的，查询快，增删慢，支持快速随机访问

**初始化：**以无参数构造方法创建 ArrayList 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量（**惰性初始化**），即向数组中添加第一个元素时，**数组容量扩为 10**

**扩容：**新容量的大小为 `oldCapacity + (oldCapacity >> 1)`，`oldCapacity >> 1` 需要取整，所以**新容量大约是旧容量的 1.5 倍**左右，即 oldCapacity+oldCapacity/2

  扩容操作需要调用 `Arrays.copyOf()`（底层 `System.arraycopy()`）把原数组整个复制到**新数组**中，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数

**删除元素：**需要调用 System.arraycopy() 将 index+1 后面的元素都复制到 index 位置上，在旧数组上操作，该操作的时间复杂度为 O(N)，可以看到 ArrayList 删除元素的代价是非常高的

**序列化：**ArrayList 基于数组并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，就没必要全部进行序列化。保存元素的数组 elementData 使用 transient 修饰，该关键字声明数组默认不会被序列化

**Fail-Fast**：快速失败，modCount 用来记录 ArrayList **结构发生变化**的次数，结构发生变化是指添加或者删除至少一个元素的操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化

  在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，改变了抛出 ConcurrentModificationException 异常



#### 4.4.2 LinkedList

##### 4.4.2.1 介绍

LinkedList 也是 List 的实现类：基于**双向链表**实现，使用 Node 存储链表节点信息，增删比较快，查询慢



##### 4.4.2.2 底层实现

LinkedList 是一个实现了 List 接口的**双端链表**，支持高效的插入和删除操作，另外也实现了 Deque 接口，使得 LinkedList 类也具有队列的特性

使 LinkedList 变成线程安全的，可以调用静态类 Collections 类中的 synchronizedList 方法：

  ```java
  List list = Collections.synchronizedList(new LinkedList(...));
  ```


##### 4.4.2.3 对比 ArrayList

1. 是否保证线程安全：ArrayList 和 LinkedList 都是不同步的，也就是不保证线程安全
2. 底层数据结构： 
   * Arraylist 底层使用的是 `Object` 数组
   * LinkedList 底层使用的是双向链表数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环）
3. 插入和删除是否受元素位置的影响：
   * ArrayList 采用数组存储，所以插入和删除元素受元素位置的影响
   * LinkedList采 用链表存储，所以对于`add(E e)`方法的插入，删除元素不受元素位置的影响
4. 是否支持快速随机访问：
   * LinkedList 不支持高效的随机元素访问，ArrayList 支持
   * 快速随机访问就是通过元素的序号快速获取元素对象(对应于 `get(int index)` 方法)
5. 内存空间占用：
   * ArrayList 的空间浪费主要体现在在 list 列表的结尾会预留一定的容量空间
   * LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList更多的空间（因为要存放直接后继和直接前驱以及数据）




### 4.5 Set


#### 4.5.1  HashSet

哈希值：

- 哈希值：JDK 根据对象的地址或者字符串或者数字计算出来的数值

- 获取哈希值：Object 类中的 public int hashCode()

- 哈希值的特点

  - 同一个对象多次调用 hashCode() 方法返回的哈希值是相同的
  - 默认情况下，不同对象的哈希值是不同的，而重写 hashCode() 方法，可以实现让不同对象的哈希值相同

**HashSet 底层就是基于 HashMap 实现，值是  PRESENT = new Object()**

Set 集合添加的元素是无序，不重复的。**是如何去重复的？**

  1.对于有值特性的，Set集合可以直接判断进行去重复。
  2.对于引用数据类型的类对象，Set集合是按照如下流程进行是否重复的判断。

  Set集合会让两两对象，先调用自己的hashCode()方法得到彼此的**哈希值**（所谓的内存地址），  然后比较两个对象的哈希值是否相同，如果不相同则直接认为两个对象不重复。
  如果哈希值相同，会继续让两个对象进行**equals比较**内容是否相同，如果相同认为真的重复了如果不相同认为不重复。

  ```java
    
              Set集合会先让对象调用hashCode()方法获取两个对象的哈希值比较
                 /                     \
              false                    true
              /                          \
          不重复                        继续让两个对象进行equals比较
                                         /          \
                                       false        true
                                        /             \
                                      不重复          重复了

  ```

* Set 系列集合元素无序的根本原因

  Set 系列集合添加元素无序的根本原因是因为**底层采用了哈希表存储元素**。

  * JDK 1.8 之前：哈希表 = 数组（初始容量16) + 链表  + （哈希算法）
  * JDK 1.8 之后：哈希表 = 数组（初始容量16) + 链表 + 红黑树  + （哈希算法）
    * 当链表长度**超过阈值 8 且当前数组的长度 > 64**时，将**链表转换为红黑树**，减少了查找时间
    * 当链表长度超过阈值 8 且当前数组的长度 < 64时，扩容




#### 4.5.2 LinkedHashSet 

LinkedHashSet 为什么是有序的？

LinkedHashSet 底层依然是使用哈希表存储元素的，但是每个元素都额外带一个链来维护添加顺序，不光增删查快，还有顺序，缺点是多了一个存储顺序的链会**占内存空间**，而且不允许重复，无索引



#### 4.5.3 TreeSet

TreeSet 集合自排序的方式：

1. 有值特性的元素直接可以升序排序（浮点型，整型）
2. 字符串类型的元素会按照首字符的编号排序
3. 对于自定义的引用数据类型，TreeSet 默认无法排序，执行的时候报错，因为不知道排序规则

自定义的引用数据类型，TreeSet 默认无法排序，需要定制排序的规则，方案有 2 种：

   * 直接为**对象的类**实现比较器规则接口 Comparable，重写比较方法；
   * 直接为**集合**设置比较器 Comparator 对象，重写比较方法；

注意：如果类和集合都带有比较规则，优先使用集合自带的比较规则



### 4.6 Queue

Queue：队列，先进先出的特性

PriorityQueue 是优先级队列，底层存储结构为 Object[]，默认实现为小顶堆，每次出队最小的元素





### 4.7 Map

#### 4.7.1 概述

Collection 是单值集合体系，Map集合是一种双列集合，每个元素包含两个值。

Map集合的每个元素的格式：key=value（键值对元素），Map集合也被称为键值对集合

Map集合的完整格式：`{key1=value1, key2=value2, key3=value3, ...}`

```
Map集合的体系：
        Map<K , V>(接口,Map集合的祖宗类)
       /                      \
      TreeMap<K , V>           HashMap<K , V>(实现类,经典的，用的最多)
                                 \
                                  LinkedHashMap<K, V>(实现类)
```

Map 集合的特点：

1. Map 集合的特点都是由键决定的
2. Map 集合的键是无序，不重复的，无索引的（Set）
3. Map 集合的值无要求（List）
4. Map 集合的键值对都可以为 null
5. Map 集合后面重复的键对应元素会覆盖前面的元素

HashMap：元素按照键是无序，不重复，无索引，值不做要求

LinkedHashMap：元素按照键是有序，不重复，无索引，值不做要求



#### 4.7.2 遍历方式

Map集合的遍历方式有：3种。

1. “键找值”的方式遍历：先获取 Map 集合全部的键，再根据遍历键找值。
2. “键值对”的方式遍历：难度较大，采用增强 for 或者迭代器
3. JDK 1.8 开始之后的新技术：foreach，采用 Lambda 表达式



### 4.8 HashMap

#### 4.8.1 基本介绍

HashMap 基于哈希表的 Map 接口实现，是以 key-value 存储形式存在，主要用来存放键值对

特点：

* HashMap 的实现不是同步的，这意味着它不是线程安全的
* key 是唯一不重复的，底层的哈希表结构，依赖 hashCode 方法和 equals 方法保证键的唯一
* key、value 都可以为null，但是 key 位置只能是一个null
* HashMap 中的映射不是有序的，即存取是无序的
* **key 要存储的是自定义对象，需要重写 hashCode 和 equals 方法，防止出现地址不同内容相同的 key**

JDK7 对比 JDK8：

* 7 = 数组 + 链表，8 = 数组 + 链表 + 红黑树
* 7 中是头插法，多线程容易造成环，8 中是尾插法
* 7 的扩容是全部数据重新定位，8 中是位置不变或者当前位置 + 旧 size 大小来实现
* 7 是先判断是否要扩容再插入，8 中是先插入再看是否要扩容

底层数据结构：

* 哈希表（Hash table，也叫散列表），根据关键码值而直接访问的数据结构。通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度，这个映射函数叫做散列函数，存放记录的数组叫做散列表

* JDK1.8 之前 HashMap 由数组+链表组成

  * 数组是 HashMap 的主体
  * 链表则是为了解决哈希冲突而存在的（**拉链法解决冲突**），拉链法就是头插法，两个对象调用的 hashCode 方法计算的哈希码值（键的哈希）一致导致计算的数组索引值相同

* JDK1.8 以后 HashMap 由**数组+链表 +红黑树**数据结构组成

  * 解决哈希冲突时有了较大的变化
  * 当链表长度**超过（大于）阈值**（或者红黑树的边界值，默认为 8）并且当前数组的**长度大于等于 64 时**，此索引位置上的所有数据改为红黑树存储
  * 即使哈希函数取得再好，也很难达到元素百分百均匀分布。当 HashMap 中有大量的元素都存放到同一个桶中时，就相当于一个长的单链表，假如单链表有 n 个元素，遍历的**时间复杂度是 O(n)**，所以 JDK1.8 中引入了 红黑树（查找**时间复杂度为 O(logn)**）来优化这个问题，使得查找效率更高



#### 4.8.2 成员属性

1. 序列化版本号

   ```java
   private static final long serialVersionUID = 362498820763181265L;
   ```

2. 集合的初始化容量（**必须是二的 n 次幂** ）

   ```java
   // 默认的初始容量是16 -- 1<<4相当于1*2的4次方---1*16
   static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
   ```

   HashMap 构造方法指定集合的初始化容量大小：

   ```java
   HashMap(int initialCapacity)// 构造一个带指定初始容量和默认加载因子 (0.75) 的空 HashMap
   ```

   * **为什么必须是 2 的 n 次幂**？用位运算替代取余计算，减少 rehash 的代价（移动的节点少）

     HashMap 中添加元素时，需要根据 key 的 hash 值确定在数组中的具体位置。为了减少碰撞，把数据分配均匀，每个链表长度大致相同，实现该方法就是取模 `hash%length`，**计算机中直接求余效率不如位移运算**， **`hash % length == hash & (length-1)` 的前提是 length 是 2 的 n 次幂**

     散列平均分布：2 的 n 次方是 1 后面 n 个 0，2 的 n 次方 -1 是 n 个 1，可以**保证散列的均匀性**，减少碰撞

     ```java
     例如长度为8时候，3&(8-1)=3  2&(8-1)=2 ，不同位置上，不碰撞；
     例如长度为9时候，3&(9-1)=0  2&(9-1)=0 ，都在0上，碰撞了；
     ```

   * 如果输入值不是 2 的幂会怎么样？

     创建 HashMap 对象时，HashMap 通过位移运算和或运算得到的肯定是 2 的幂次数，并且是大于那个数的最近的数字，底层采用 tableSizeFor() 方法

3. 默认的负载因子，默认值是 0.75 

    ```java
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    ```

4. 集合最大容量 

    ```java
    // 集合最大容量的上限是：2的30次幂
    static final int MAXIMUM_CAPACITY = 1 << 30;// 0100 0000 0000 0000 0000 0000 0000 0000 = 2 ^ 30
    ```

5. 当链表的值超过 8 则会转红黑树（JDK1.8 新增）

    ```java
    // 当桶(bucket)上的结点数大于这个值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8;
    ```

    为什么 Map 桶中节点个数大于 8 才转为红黑树？

    * 在 HashMap 中有一段注释说明：**空间和时间的权衡**

      ```java
      TreeNodes占用空间大约是普通节点的两倍，所以我们只在箱子包含足够的节点时才使用树节点。当节点变少(由于删除或调整大小)时，就会被转换回普通的桶。在使用分布良好的用户hashcode时，很少使用树箱。理想情况下，在随机哈希码下，箱子中节点的频率服从"泊松分布"，默认调整阈值为0.75，平均参数约为0.5，尽管由于调整粒度的差异很大。忽略方差，列表大小k的预期出现次数是(exp(-0.5)*pow(0.5, k)/factorial(k))
      0:    0.60653066
      1:    0.30326533
      2:    0.07581633
      3:    0.01263606
      4:    0.00157952
      5:    0.00015795
      6:    0.00001316
      7:    0.00000094
      8:    0.00000006
      more: less than 1 in ten million
      一个bin中链表长度达到8个元素的概率为0.00000006，几乎是不可能事件，所以我们选择8这个数字
      ```

    * 其他说法
      红黑树的平均查找长度是 log(n)，如果长度为 8，平均查找长度为 log(8)=3，链表的平均查找长度为 n/2，当长度为 8 时，平均查找长度为 8/2=4，这才有转换成树的必要；链表长度如果是小于等于 6，6/2=3，而 log(6)=2.6，虽然速度也很快的，但转化为树结构和生成树的时间并不短

6. 当链表的值小于 6 则会从红黑树转回链表

    ```java
    // 当桶(bucket)上的结点数小于这个值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    ```

7. 当 Map 里面的数量**大于等于**这个阈值时，表中的桶才能进行树形化 ，否则桶内元素超过 8 时会扩容，而不是树形化。为了避免进行扩容、树形化选择的冲突，这个值不能小于 4 * TREEIFY_THRESHOLD (8)

    ```java
    // 桶中结构转化为红黑树对应的数组长度最小的值 
    static final int MIN_TREEIFY_CAPACITY = 64;
    ```

    原因：数组比较小的情况下变为红黑树结构，反而会降低效率，红黑树需要进行左旋，右旋，变色这些操作来保持平衡

8. table 用来初始化（必须是二的 n 次幂）

    ```java
    // 存储元素的数组 
    transient Node<K,V>[] table;
    ```

 9. HashMap 中**存放元素的个数**

    ```java
    // 存放元素的个数，HashMap中K-V的实时数量，不是table数组的长度
    transient int size;
    ```

10. 记录 HashMap 的修改次数 

   ```java
   // 每次扩容和更改map结构的计数器
    transient int modCount;  
   ```

11. 调整大小下一个容量的值计算方式为：容量 * 负载因子，容量是数组的长度

     ```java
     // 临界值，当实际大小(容量*负载因子)超过临界值时，会进行扩容
     int threshold;
     ```

12. **哈希表的加载因子**

    ```java
     final float loadFactor;
    ```

    * 加载因子的概述

      loadFactor 加载因子，是用来衡量 HashMap 满的程度，表示 HashMap 的疏密程度，影响 hash 操作到同一个数组位置的概率，计算 HashMap 的实时加载因子的方法为 **size/capacity**，而不是占用桶的数量去除以 capacity，capacity 是桶的数量，也就是 table 的长度 length

      当 HashMap 容纳的元素已经达到数组长度的 75% 时，表示 HashMap 拥挤需要扩容，而扩容这个过程涉及到 rehash、复制数据等操作，非常消耗性能，所以开发中尽量减少扩容的次数，通过创建 HashMap 集合对象时指定初始容量来避免

      ```java
      HashMap(int initialCapacity, float loadFactor)//构造指定初始容量和加载因子的空HashMap
      ```

    * 为什么加载因子设置为 0.75，初始化临界值是 12？

      loadFactor 太大导致查找元素效率低，存放的数据拥挤，太小导致数组的利用率低，存放的数据会很分散。loadFactor 的默认值为 **0.75f 是官方给出的一个比较好的临界值**

    * threshold 计算公式：capacity（数组长度默认16） * loadFactor（默认 0.75）。当 size >= threshold 的时候，那么就要考虑对数组的 resize（扩容），这就是衡量数组是否需要扩增的一个标准， 扩容后的 HashMap 容量是之前容量的**两倍**



## 2 反射

***

### 2.1 什么是反射？

反射是在**运行状态中**，对**任意一个类**都能够知道这个类的**所有方法和属性**；对于**任意一个对象**，都能调用它**任意方法和属性**；这种动态获取信息以及动态调用对象方法的功能称为Java的反射机制。

### 2.2 反射机制的优缺点？

**优点：**能动态获取对象的实例，提高灵活性；可于动态编译集合。

如，`Class.forName('com.mysql.jdbc.Driver.class');` ，加载MySQL的驱动类。

**缺点：**使用反射性**能较低**，需要解析字节码，将内存中的对象进行解析。

其解决方案是：

- 通过`setAccessible(true)`关闭JDK的安全检查来提升反射速度；
- 多次创建一个类的实例时，有缓存会快很多；
- ReflflectASM工具类，通过字节码生成的方式加快反射速度。

### 2.3 如何获取反射中的Class对象？

1. `Class.forName("类的路径")；` 当知道该类的全路径名时，你可以使用该方法获取 Class 类对象。

   ```java
   Class clz = Class.forName("java.lang.String");
   ```

2. `类名.class`这种方法只适合在编译前就知道操作的 Class。

   ```java
   Class clz = String.class
   ```

3. `对象名.getClass()`。

   ```java
   String str = new String("Hello");
   Class clz = str.getClass();
   ```

4. 如果是基本类型的`包装类`，可以调用包装类的`Type属性`来获得该包装类的Class对象。

   ```java
   Class clz = Integer.TYPE;
   ```

   ​

### 2.4 Java反射API有几类？

反射 API 用来生成 JVM 中的类、接口或则对象的信息。

- Class 类：反射的核心类，可以获取**类的属性，方法**等信息；
- Field 类：Java.lang.reflec 包中的类，表示类的**成员变量**，可以用来**获取和设置**类之中的属性值；
- Method 类：Java.lang.reflec 包中的类，表示**类的方法**，它可以用来**获取**类中的方法信息或者**执行**方法；
- Constructor 类：Java.lang.reflec 包中的类，表示类的**构造方法**。

### 2.5 反射使用步骤

1. 获取想要操作的类的Class对象，这是反射的核心，通过Class对象我们可以任意调用类的方法。
2. 调用 Class 类中的方法，既就是反射的使用阶段。
3. 使用反射 API 来操作这些信息。

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;

public class Main {
    public void print(String manner) {
        System.out.println(manner);
    }

    public static void main(String[] args) throws Exception {
        //正常调用
        Main main = new Main();
        main.print("正常调用");
        //反射调用
        //1、获取class对象
        Class clz = Class.forName("Main");
        //2、获取方法
        Method mainPrint = clz.getMethod("print", String.class);
        //3、获取构造方法
        Constructor constructorMain = clz.getConstructor();
        //4、创建对象实例
        Object newMain = constructorMain.newInstance();
        //5、方法调用
        mainPrint.invoke(newMain, "反射调用");
    }
}
```

### 2.6 为什么引入反射概念？反射机制的应用有哪些？

反射主要应用在以下几方面：

- 反射让开发人员可以通过外部类的全路径名**创建对象**，并使用这些类，实现一些扩展的功能。
- 反射让开发人员可以**枚举出类的全部成员**，包括构造函数、属性、方法。以帮助开发者写出正确的代码。
- 测试时可以利用反射 API **访问类的私有成员**，以保证测试代码覆盖率

第一种：JDBC 的数据库的连接。

在JDBC 的操作中，如果要想进行数据库的连接，则必须按照以上的几步完成

1. 通过Class.forName()加载数据库的驱动程序 （通过反射加载，前提是引入相关了Jar包）；
2. 通过 DriverManager 类进行数据库的连接，连接的时候要输入数据库的连接地址、用户名、密码；
3. 通过Connection 接口接收连接。

第二种：Spring 框架的使用，最经典的就是xml的配置模式。

Spring 通过 XML 配置模式装载 Bean 的过程：

1. 将程序内所有 XML 或 Properties 配置文件加载入内存中；
2. Java类里面解析xml或properties里面的内容，得到对应实体类的字节码字符串以及相关的属性信
   息；
3. 使用反射机制，根据这个字符串获得某个类的Class实例；
4. 动态配置实例的属性。

Spring这样做的好处是：

- 不用每一次都要在代码里面去new或者做其他的事情；
- 以后要改的话直接改配置文件，代码维护起来就很方便了；
- 有时为了适应某些需求，Java类里面不一定能直接调用另外的方法，可以通过反射机制来实现。

## 3 泛型

***

### 3.1 什么是泛型

在Java中，泛型是**将类型参数化，在编译时才确定具体的参数**，允许你在定义**类、接口或方法**时使用参数来表示类型。使用泛型可以使代码更加灵活、类型安全，并提高代码的重用性。

泛型的核心思想是类型参数化，它允许你在使用一个类或接口时指定具体的类型，从而在编译时期进行类型检查，**避免在运行时出现类型转换错误或类型安全问题**。

使用泛型的主要优点包括：

1. 类型安全：泛型提供了编译时的类型检查，可以在编译时捕获类型错误，避免在运行时出现类型转换异常。
2. 代码重用：泛型使得代码更加通用，可以在多种类型上进行重用，而不需要编写多个相似的方法或类。
3. 减少类型转换：使用泛型可以避免手动进行类型转换，提高代码的可读性和可维护性。

### 3.2 什么是类型擦除？

在使用泛型时加上类型参数，编译器在编译时去掉类型参数，即**泛型类型参数会被替换为它们的上限或Object类型**。

### 3.3 泛型中限定通配符和非限定通配符

限定通配符对类型进行了限制：

1. `<? extends T>` 通过确保类型必须是T的子类来限定类型的上界；
2. `<? super T>` 通过确保类型必须是T的父类来限定类型的上界。

非限定通配符`?` ，可以使用任意类型。例如，`List<?>`可以支持任意类型，如`List<A>、List<Integer>、List<B>`等。

泛型类型必须用限定内的类型初始化，否则会导致编译错误。



## 4 序列化与反序列化

***

### 4.1 Java序列化与反序列化是什么？

- 序列化（Serialization）：是将**对象转换为字节序列**的过程，以便对象以字节序列形式在网络上传输或保存到持久存储介质（如磁盘）中；
- 反序列化（Deserialization）：是将**字节序列恢复为对象**的过程。

序列化的主要目的是实现对象的持久化和传输。当对象被序列化后，它的状态信息（实例变量）会被转换成字节序列，包括对象的类型、属性值等。这样，该字节序列就可以被写入到文件、数据库或通过网络发送出去。

1. 对象序列化可以实现分布式对象，如，RMI（远程调用Remote Method Invocation）利用对象序列化运行远程主机上的服务，就像在本地机器上运行一样。
2. Java对象序列化不仅是一个对象的数据，而且递归保存对象引用的每个对象数据；
   - 可以将整个对象写入字节流，保存在文件中或者网络传输；
   - 对象深clone，复制对象本身以及引用，可以得到整个对象的序列（包括对象中属性是对象的序列）。
3. 对象、文件、数据，有许多不同格式，很难统一传输和保存，序列化成字节流，可以进行通用的格式传输和保存。

### 4.2 序列化实现方式

实现`Serializable`接口或`Externalizable`接口。

 **Serializable** 接口

类通过实现`java.io.Serializable`接口以启用序列化功能，`Serializable`接口没有方法或字段，仅用于标识可序列化。

**Externalizable** 接口

`Externalizable`继承自`Serializable`,在使用时需要重写`writeExternal`和`readExternal`方法，否则所有的变量会变成默认值。

可序列化类的所有子类都是可序列化的。

区别：

- `Serializable`只需要实现接口即可，无需实现方法，序列化所有信息，性能略差；
- `Externalizable`需要实现接口中的两个方法，可以自己决定序列化哪些信息，性能略好。

User类实现`Serializable`接口：

```java
class User implements Serializable {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
//output
//User{name='aldon', age=18}
```



**Externalizable** 接口

```java
public interface Externalizable extends java.io.Serializable {
  void writeExternal(ObjectOutput out) throws IOException;
  void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
}
```

User类实现`Externalizable`接口：

```java
class User implements Externalizable {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

  	//当自定义参数序列化时，需要一个没有参数的构造器
    public User() {
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(name);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        name = (String) in.readObject();
    }
}

//output
//User{name='aldon', age=0}
```



序列化与反序列：

```java
public class Main {
    public static void main(String[] args) {
        User user = new User("aldon");

        // write obj to file
        try (FileOutputStream fos = new FileOutputStream("user.txt");
             ObjectOutputStream oos = new ObjectOutputStream(fos)) {
            oos.writeObject(user);
        } catch (IOException e) {
            e.printStackTrace();
        }

        //read obj from file
        try (FileInputStream fis = new FileInputStream(new File("user.txt"));
             ObjectInputStream ois = new ObjectInputStream(fis)) {
            User nUser = (User) ois.readObject();
            System.out.println(nUser);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }

    }
}

```



### 4.3 什么是serialVersionUID？

`serialVersionUID`用来表明类的版本，校验版本的一致性。

在进行反序列化时，会将字节流中的`serialVersionUID`与本地类的`serialVersionUID`进行比较，如果一致，则可以进行反序列化，否则会出现版本不一致的异常。



### 4.4 为什么要显示指定serialVersionUID

在反序列化时会自动生成一个`serialVersionUID` ，如果显示指定，则生成的`serialVersionUID`是我们显示指定的值，这样在进行版本比较时就是预想的结果。

如果不显示指定，会出现什么问题？

- 如果类写完不再修改，那不会出现问题。
- 实际开发中代码会不断迭代，一旦修改就对象反序列化就会报错，抛出序列化运行时异常，因此实际开发中要指定一个不变的`serialVersionUID`值。



### 4.5 如果有些字段不想序列化，该怎么办？

使用`transient`关键字修饰，只能修饰变量，不能修饰类和方法。

阻止变量序列化成字节流序列，在反序列化时用`transient`关键字修饰的变量设置为初始值。



### 4.6 静态变量会被修饰吗？

不会。

序列化是针对对象实例而言，而静态变量是随着类加载而加载。

虽然`serialVersionUID`是`static`修饰的，但JVM在序列化是会自动生成一个`serialVersionUID`，然后将显示指定的`serialVersionUID`值赋值给自动生成的`serialVersionUID`.



## 5 异常

### 5.1 Error 和 Exception 区别是什么？

Java 中，所有的异常都有一个共同的祖先 java.lang 包中的 Throwable 类。 Throwable 类有两个重要的子类 `Exception （异常）`和 `Error （错误）`。

Exception 和 Error 二者都是 Java 异常处理的重要子类，各自都包含大量子类。

- Exception :**程序本身可以处理的异常**，可以通过 catch 来进行捕获，通常遇到这种错误，应对其进行处理，使应用程序可以继续正常运行。 Exception 又可以分为`运行时异常(RuntimeException, 又叫非受检查异常)`和`非运行时异常(又叫受检查异常)` 。
- Error ： Error 属于**程序无法处理的错误** ，我们没办法通过 catch 来进行捕获 。例如，系统崩溃，内存不足，堆栈溢出等，编译器不会对这类错误进行检测，一旦这类错误发生，通常应用程序会被终止，仅靠应用程序本身无法恢复。

### 5.2 非受检查异常(运行时异常)和受检查异常(非运行时异常)区别是什么？

非受检查异常：包括 RuntimeException 类及其子类，表示 **JVM 在运行期间可能出现的异常**。 **Java 编译器不会检查运行时异常**。例如： NullPointException(空指针) 、 NumberFormatException（字符串转换为数字） 、 IndexOutOfBoundsException(数组越界) 、 ClassCastException(类转换异常) 、ArrayStoreException(数据存储异常，操作数组时类型不一致) 等。

受检查异常：是Exception 中除 RuntimeException 及其子类之外的异常。 J**ava 编译器会检查受检查异常**。常见的受检查异常有： IO 相关的异常、 ClassNotFoundException 、 SQLException 等。

非受检查异常和受检查异常之间的区别：是否强制要求调用者必须处理此异常，如果强制要求调用者必须进行处理，那么就使用受检查异常，否则就选择非受检查异常。



### 5.3 throw 和 throws 的区别是什么?

- throw 关键字用在**方法内部**，只能用于**抛出一种异常**，用来抛出方法或代码块中的异常，受检查异常和非受检查异常都可以被抛出。
- throws 关键字用在**方法声明上**，可以**抛出多个异常**，用来标识该方法可能抛出的异常列表。一个方法用 throws 标识了可能抛出的异常列表，调用该方法的方法中必须包含可处理异常的代码，否则也要在方法签名中用 throws 关键字声明相应的异常。



### 5.4  try-catch-finally 中哪个部分可以省略？

可以省略`catch`或者`finally` ，二者不可以同时省略，可以是`try-catch、try-finally、try-catch-finally` 。

普通异常如果选择捕获，则必须用catch显示声明以便进一步处理，运行时异常在编译时
没有如此规定。但是，如果有运行时异常并且没有catch捕获处理，会出现程序中断。



### 5.5 try-catch-finally 中，如果 catch 中 return 了，finally 还会执行吗？

不管什么情况，finally代码块都会执行，但是如果try或者catch代码块中有return语句，并且return能执行，那么返回的代码是return中结果，finally无法修改其变量的值；如果finally代码块中有return语句，那么返回的是finally代码块的值。

```java
public static void main(String[] args) throws Exception {
    System.out.println(fun());
}

private static int fun(){
    int a = 10;
    try {
        System.out.println(a / 0);
        a = 20;
        return a;
    } catch (ArithmeticException e) {
        a = 30;
        return a;
    } finally {
        a = 40;
        System.out.println("execute finally");
    }
}
//执行结果
execute finally
30
```

```java
public static void main(String[] args) throws Exception {
    System.out.println(fun());
}

private static int fun(){
    int a = 10;
    try {
        System.out.println(a / 0);
        a = 20;
        return a;
    } catch (ArithmeticException e) {
        a = 30;
        return a;
    } finally {
        a = 40;
        System.out.println("execute finally");
        return a;
    }
}
//执行结果
execute finally
40
```

### 5.6 JVM 是如何处理异常的？

在一个方法中如果发生异常，这个方法会创建一个异常对象，并转交给 JVM，该异常对象包含异常名称，异常描述以及异常发生时应用程序的状态。创建异常对象并转交给 JVM 的过程称为抛出异常。可能有一系列的方法调用，最终才进入抛出异常的方法，这一系列方法调用的有序列表叫做调用栈。

JVM 会顺着调用栈去查找看是否有可以处理异常的代码，如果有，则调用异常处理代码。当 JVM 发现可以处理异常的代码时，会把发生的异常传递给它。如果 JVM 没有找到可以处理该异常的代码块，JVM 就会将该异常转交给默认的异常处理器（默认处理器为 JVM 的一部分），默认异常处理器打印出异常信息并终止应用程序。

## AQS

### 1、什么是AQS？

AQS：AbstractQueuedSynchronizer，是多线程的抽象队列同步器，是**阻塞式锁**和相关的同步器工具的框架，许多同步类实现都依赖于该同步器，其中ReentrantLock、Semaphore都是基于AQS实现的。内部实现了两个队列，分别是**同步队列**和**条件队列**。其中**同步队列**是一个双向链表，里面储存的是处于等待状态的线程，正在排队等待唤醒去获取锁，而**条件队列**是一个单向链表，里面储存的也是处于等待状态的线程，只不过这些线程唤醒的结果是加入到了同步队列的队尾，`AQS`所做的就是管理这两个队列里面线程之间的**等待状态-唤醒**的工作。
在同步队列中，还存在`2`种模式，分别是**独占模式**和**共享模式**，这两种模式的区别就在于`AQS`在唤醒线程节点的时候是不是传递唤醒，这两种模式分别对应**独占锁**和**共享锁**。`AQS`是一个抽象类，所以不能直接实例化，当我们需要实现一个自定义锁的时候可以去继承`AQS`然后重写**获取锁的方式**和**释放锁的方式**还有**管理state**

### 2、AQS流程？

AQS内部维护了一个先进显出的双向队列，队列中存储排队的线程；在AQS内部还有一个属性state，这个属性相当于是一个资源，默认是0，表示无锁状态，如果队列中有一个线程将state成功修改为1，则表示当前线程获取了资源。当有多个线程对state进行修改的时候，使用cas操作来保证修改的原子性。

### 2、AQS使用场景

- 锁：ReentrantLock、ReentrantReadWriteLock等锁的实现都是基于AQS的。
  - ReentrantReadWriteLock是一种读写锁的实现，它支持多个读线程同时访问共享资源，但只允许一个写线程访问。
- 信号量：Semaphore是一种基于计数器的同步机制，可以指定多个线程同时访问某个资源。
- CountDownLatch：CountDownLatch是一种倒计时器，它可以用于协调多个线程之间的同步。通常用来控制线程等待，可以让某一个线程等待直到倒计时结束再开始执行。
- CyclicBarrier：循环栅栏，让一组线程达到某一个屏障时才会开门，所有屏障被屏障拦截的线程才会继续干活。

### 3、ReentrantLock是怎么实现的？

`ReentrantLock`通过重写**锁获取方式**和**锁释放方式**这两个方法实现了**公平锁**和**非公平锁**，即`ReentrantLock`就是通过重写了`AQS`的`tryAcquire`和`tryRelease`方法实现的`lock`和`unlock`。

`ReentrantLock`有两个构造方法，无参构造方法默认是创建**非公平锁**，而传入`true`为参数的构造方法创建的是**公平锁**。

**非公平锁的实现原理**

**lock方法获取锁**

1. `lock`方法调用`CAS`方法设置`state`的值，如果`state`等于期望值`0`(代表锁没有被占用)，那么就将`state`更新为`1`(代表该线程获取锁成功)，然后执行`setExclusiveOwnerThread`方法直接将该线程设置成锁的所有者。如果`CAS`设置`state`的值失败，即`state`不等于`0`，代表锁正在被占领着，则执行`acquire(1)`，即下面的步骤。
2. `nonfairTryAcquire`方法首先调用`getState`方法获取`state`的值，如果`state`的值为`0`(之前占领锁的线程刚好释放了锁)，那么用`CAS`这是`state`的值，设置成功则将该线程设置成锁的所有者，并且返回`true`。如果`state`的值不为`0`，那就**调用getExclusiveOwnerThread方法查看占用锁的线程是不是自己**，如果是的话那就直接将`state + 1`，然后返回`true`。如果`state`不为`0`且锁的所有者又不是自己，那就返回`false`，**然后线程会进入到同步队列中**。

**tryRelease锁的释放**

1. 判断当前线程是不是锁的所有者，如果是则进行步骤`2`，如果不是则抛出异常。
2. 判断此次释放锁后`state`的值是否为0，如果是则代表**锁有没有重入**，然后将锁的所有者设置成`null`且返回`true`，然后执行步骤`3`，如果不是则**代表锁发生了重入**执行步骤`4`。
3. 现在锁已经释放完，即`state=0`，唤醒同步队列中的后继节点进行锁的获取。
4. 锁还没有释放完，即`state!=0`，不唤醒同步队列。

**公平锁的实现原理**

**lock方法获取锁**

1. 获取状态的`state`的值，如果`state=0`即代表锁没有被其它线程占用(但是并不代表同步队列没有线程在等待)，执行步骤`2`。如果`state!=0`则代表锁正在被其它线程占用，执行步骤`3`。
2. 判断同步队列是否存在线程(节点)，如果不存在则直接将锁的所有者设置成当前线程，且更新状态state，然后返回true。
3. 判断锁的所有者是不是当前线程，如果是则更新状态state的值，然后返回true，如果不是，那么返回false，即线程会被加入到同步队列中

通过步骤`2`实现了锁获取的公平性，即锁的获取按照先来先得的顺序，后来的不能抢先获取锁，非公平锁和公平锁也正是通过这个区别来实现了锁的公平性。

**tryRelease锁的释放**

公平锁的释放和非公平锁的释放一样，这里就不重复。
公平锁和非公平锁的公平性是在**获取锁**的时候体现出来的，释放的时候都是一样释放的。

### 4、AQS使用了哪些设计模式？

AQS同步器使用了模板方法模式，如果需要定义同步器，一般方式是：

1. 使用者继承AbstractQueuedSynchronizer，并重写指定的方法，一般是对共享资源state的获取的释放；
2. 将AQS组合再自定义同步组件的实现中，并调用模板方法，而这些模板方法会调用使用者重写的方法。
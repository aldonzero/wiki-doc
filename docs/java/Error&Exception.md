## 1、Error 和 Exception 区别是什么？

Java 中，所有的异常都有一个共同的祖先 java.lang 包中的 Throwable 类。 Throwable 类有两个重要的子类 `Exception （异常）`和 `Error （错误）`。

Exception 和 Error 二者都是 Java 异常处理的重要子类，各自都包含大量子类。

- Exception :**程序本身可以处理的异常**，可以通过 catch 来进行捕获，通常遇到这种错误，应对其进行处理，使应用程序可以继续正常运行。 Exception 又可以分为`运行时异常(RuntimeException, 又叫非受检查异常)`和`非运行时异常(又叫受检查异常)` 。
- Error ： Error 属于**程序无法处理的错误** ，我们没办法通过 catch 来进行捕获 。例如，系统崩溃，内存不足，堆栈溢出等，编译器不会对这类错误进行检测，一旦这类错误发生，通常应用程序会被终止，仅靠应用程序本身无法恢复。

## 2、非受检查异常(运行时异常)和受检查异常(一般异常)区别是什么？

非受检查异常：包括 RuntimeException 类及其子类，表示 **JVM 在运行期间可能出现的异常**。 **Java 编译器不会检查运行时异常**。例如： NullPointException(空指针) 、 NumberFormatException（字符串转换为数字） 、 IndexOutOfBoundsException(数组越界) 、 ClassCastException(类转换异常) 、ArrayStoreException(数据存储异常，操作数组时类型不一致) 等。
受检查异常：是Exception 中除 RuntimeException 及其子类之外的异常。 J**ava 编译器会检查受检查异常**。常见的受检查异常有： IO 相关的异常、 ClassNotFoundException 、 SQLException 等。

非受检查异常和受检查异常之间的区别：是否强制要求调用者必须处理此异常，如果强制要求调用者必须进行处理，那么就使用受检查异常，否则就选择非受检查异常。

## 3、throw 和 throws 的区别是什么?

- throw 关键字用在**方法内部**，只能用于**抛出一种异常**，用来抛出方法或代码块中的异常，受查异常和非受查异常都可以被抛出。
- throws 关键字用在**方法声明上**，可以**抛出多个异常**，用来标识该方法可能抛出的异常列表。一个方法用 throws 标识了可能抛出的异常列表，调用该方法的方法中必须包含可处理异常的代码，否则也要在方法签名中用 throws 关键字声明相应的异常。

## 3、 try-catch-finally 中哪个部分可以省略？

可以省略`catch`或者`finally` ，二者不可以同时省略，可以是`try-catch、try-finally、try-catch-finally` 。

普通异常如果选择捕获，则必须用catch显示声明以便进一步处理，运行时异常在编译时
没有如此规定。但是，如果有运行时异常并且没有catch捕获处理，会出现程序中断。

## 4、try-catch-finally 中，如果 catch 中 return 了，finally 还会执行吗？

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

## 5、JVM 是如何处理异常的？

在一个方法中如果发生异常，这个方法会创建一个异常对象，并转交给 JVM，该异常对象包含异常名称，异常描述以及异常发生时应用程序的状态。创建异常对象并转交给 JVM 的过程称为抛出异常。可能有一系列的方法调用，最终才进入抛出异常的方法，这一系列方法调用的有序列表叫做调用栈。

JVM 会顺着调用栈去查找看是否有可以处理异常的代码，如果有，则调用异常处理代码。当 JVM 发现可以处理异常的代码时，会把发生的异常传递给它。如果 JVM 没有找到可以处理该异常的代码块，JVM 就会将该异常转交给默认的异常处理器（默认处理器为 JVM 的一部分），默认异常处理器打印出异常信息并终止应用程序。
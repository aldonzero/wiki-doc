## 反射

***

### 1、什么是反射？

反射是在**运行状态中**，对**任意一个类**都能够知道这个类的**所有方法和属性**；对于**任意一个对象**，都能调用它**任意方法和属性**；这种动态获取信息以及动态调用对象方法的功能称为Java的反射机制。

### 2、反射机制的优缺点？

**优点：**能动态获取对象的实例，提高灵活性；可于动态编译集合。

如，`Class.forName('com.mysql.jdbc.Driver.class');` ，加载MySQL的驱动类。

**缺点：**使用反射性**能较低**，需要解析字节码，将内存中的对象进行解析。

其解决方案是：

- 通过`setAccessible(true)`关闭JDK的安全检查来提升反射速度；
- 多次创建一个类的实例时，有缓存会快很多；
- ReflflectASM工具类，通过字节码生成的方式加快反射速度。

### 3、如何获取反射中的Class对象？

1. `Class.forName(“类的路径”)；`当知道该类的全路径名时，你可以使用该方法获取 Class 类对象。

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

### 4、Java反射API有几类？

反射 API 用来生成 JVM 中的类、接口或则对象的信息。

- Class 类：反射的核心类，可以获取**类的属性，方法**等信息；
- Field 类：Java.lang.reflec 包中的类，表示类的**成员变量**，可以用来**获取和设置**类之中的属性值；
- Method 类：Java.lang.reflec 包中的类，表示**类的方法**，它可以用来**获取**类中的方法信息或者**执行**方法；
- Constructor 类：Java.lang.reflec 包中的类，表示类的**构造方法**。

### 5、反射使用步骤

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

### 6、为什么引入反射概念？反射机制的应用有哪些？

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

## 泛型

***

### 什么是泛型

在Java中，泛型是**将类型参数化，在编译时才确定具体的参数**，允许你在定义**类、接口或方法**时使用参数来表示类型。使用泛型可以使代码更加灵活、类型安全，并提高代码的重用性。

泛型的核心思想是类型参数化，它允许你在使用一个类或接口时指定具体的类型，从而在编译时期进行类型检查，**避免在运行时出现类型转换错误或类型安全问题**。

使用泛型的主要优点包括：

1. 类型安全：泛型提供了编译时的类型检查，可以在编译时捕获类型错误，避免在运行时出现类型转换异常。
2. 代码重用：泛型使得代码更加通用，可以在多种类型上进行重用，而不需要编写多个相似的方法或类。
3. 减少类型转换：使用泛型可以避免手动进行类型转换，提高代码的可读性和可维护性。

### 什么是类型擦除？

在使用泛型时加上类型参数，编译器在编译时去掉类型参数，即泛型类型参数会被替换为它们的上限或Object类型。

### 泛型中限定通配符和非限定通配符

限定通配符对类型进行了限制：

1. `<? extends T>` 通过确保类型必须是T的子类来限定类型的上界；
2. `<? super T>` 通过确保类型必须是T的父类来限定类型的上界。

非限定通配符`?` ，可以使用任意类型。例如，`List<?>`可以支持任意类型，如`List<A>、List<Integer>、List<B>`等。

泛型类型必须用限定内的类型初始化，否则会导致编译错误。

## 序列化与反序列化

***

### Java序列化与反序列化是什么？

- 序列化（Serialization）：是将**对象转换为字节序列**的过程，以便对象以字节序列形式在网络上传输或保存到持久存储介质（如磁盘）中；
- 反序列化（Deserialization）：是将**字节序列恢复为对象**的过程。

序列化的主要目的是实现对象的持久化和传输。当对象被序列化后，它的状态信息（实例变量）会被转换成字节序列，包括对象的类型、属性值等。这样，该字节序列就可以被写入到文件、数据库或通过网络发送出去。

1. 对象序列化可以实现分布式对象，如，RMI（远程调用Remote Method Invocation）利用对象序列化运行远程主机上的服务，就像在本地机器上运行一样。
2. Java对象序列化不仅是一个对象的数据，而且递归保存对象引用的每个对象数据；
   - 可以将整个对象写入字节流，保存在文件中或者网络传输；
   - 对象深clone，复制对象本身以及引用，可以得到整个对象的序列（包括对象中属性是对象的序列）。
3. 对象、文件、数据，有许多不同格式，很难统一传输和保存，序列化成字节流，可以进行通用的格式传输和保存。

### 序列化实现方式

实现`Serializable`接口或`Externalizable`接口。

 **`Serializable` **接口

类通过实现`java.io.Serializable`接口以启用序列化功能，`Serializable`接口没有方法或字段，仅用于标识可序列化。

**`Externalizable` **接口

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



**`Externalizable`**接口

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

### 什么是serialVersionUID？

`serialVersionUID`用来表明类的版本，校验版本的一致性。

在进行反序列化时，会将字节流中的`serialVersionUID`与本地类的`serialVersionUID`进行比较，如果一致，则可以进行反序列化，否则会出现版本不一致的异常。

### 为什么要显示指定serialVersionUID

在反序列化时会自动生成一个`serialVersionUID`,如果显示指定，则生成的`serialVersionUID`是我们显示指定的值，这样在进行版本比较时就是预想的结果。

如果不显示指定，会出现什么问题？

- 如果类写完不再修改，那不会出现问题。
- 实际开发中代码会不断迭代，一旦修改就对象反序列化就会报错，抛出序列化运行时异常，因此实际开发中要指定一个不变的`serialVersionUID`值。

### 如果有些字段不想序列化，该怎么办？

使用`transient`关键字修饰，只能修饰变量，不能修饰类和方法。

阻止变量序列化成字节流序列，在反序列化时用`transient`关键字修饰的变量设置为初始值。

### 静态变量会被修饰吗？

不会。

序列化是针对对象实例而言，而静态变量是随着类加载而加载。

虽然`serialVersionUID`是`static`修饰的，但JVM在序列化是会自动生成一个`serialVersionUID`，然后将显示指定的`serialVersionUID`值赋值给自动生成的`serialVersionUID`.

## 异常

***

### 1、Error 和 Exception 区别是什么？

Java 中，所有的异常都有一个共同的祖先 java.lang 包中的 Throwable 类。 Throwable 类有两个重要的子类 `Exception （异常）`和 `Error （错误）`。

Exception 和 Error 二者都是 Java 异常处理的重要子类，各自都包含大量子类。

- Exception :**程序本身可以处理的异常**，可以通过 catch 来进行捕获，通常遇到这种错误，应对其进行处理，使应用程序可以继续正常运行。 Exception 又可以分为`运行时异常(RuntimeException, 又叫非受检查异常)`和`非运行时异常(又叫受检查异常)` 。
- Error ： Error 属于**程序无法处理的错误** ，我们没办法通过 catch 来进行捕获 。例如，系统崩溃，内存不足，堆栈溢出等，编译器不会对这类错误进行检测，一旦这类错误发生，通常应用程序会被终止，仅靠应用程序本身无法恢复。

### 2、非受检查异常(运行时异常)和受检查异常(一般异常)区别是什么？

非受检查异常：包括 RuntimeException 类及其子类，表示 **JVM 在运行期间可能出现的异常**。 **Java 编译器不会检查运行时异常**。例如： NullPointException(空指针) 、 NumberFormatException（字符串转换为数字） 、 IndexOutOfBoundsException(数组越界) 、 ClassCastException(类转换异常) 、ArrayStoreException(数据存储异常，操作数组时类型不一致) 等。
受检查异常：是Exception 中除 RuntimeException 及其子类之外的异常。 J**ava 编译器会检查受检查异常**。常见的受检查异常有： IO 相关的异常、 ClassNotFoundException 、 SQLException 等。

非受检查异常和受检查异常之间的区别：是否强制要求调用者必须处理此异常，如果强制要求调用者必须进行处理，那么就使用受检查异常，否则就选择非受检查异常。

### 3、throw 和 throws 的区别是什么?

- throw 关键字用在**方法内部**，只能用于**抛出一种异常**，用来抛出方法或代码块中的异常，受查异常和非受查异常都可以被抛出。
- throws 关键字用在**方法声明上**，可以**抛出多个异常**，用来标识该方法可能抛出的异常列表。一个方法用 throws 标识了可能抛出的异常列表，调用该方法的方法中必须包含可处理异常的代码，否则也要在方法签名中用 throws 关键字声明相应的异常。

### 3、 try-catch-finally 中哪个部分可以省略？

可以省略`catch`或者`finally` ，二者不可以同时省略，可以是`try-catch、try-finally、try-catch-finally` 。

普通异常如果选择捕获，则必须用catch显示声明以便进一步处理，运行时异常在编译时
没有如此规定。但是，如果有运行时异常并且没有catch捕获处理，会出现程序中断。

### 4、try-catch-finally 中，如果 catch 中 return 了，finally 还会执行吗？

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

### 5、JVM 是如何处理异常的？

在一个方法中如果发生异常，这个方法会创建一个异常对象，并转交给 JVM，该异常对象包含异常名称，异常描述以及异常发生时应用程序的状态。创建异常对象并转交给 JVM 的过程称为抛出异常。可能有一系列的方法调用，最终才进入抛出异常的方法，这一系列方法调用的有序列表叫做调用栈。

JVM 会顺着调用栈去查找看是否有可以处理异常的代码，如果有，则调用异常处理代码。当 JVM 发现可以处理异常的代码时，会把发生的异常传递给它。如果 JVM 没有找到可以处理该异常的代码块，JVM 就会将该异常转交给默认的异常处理器（默认处理器为 JVM 的一部分），默认异常处理器打印出异常信息并终止应用程序。
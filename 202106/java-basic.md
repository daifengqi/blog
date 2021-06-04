# 学点Java：入门笔记

# Java Tutorials

## 2021 年 6 月 1 日

## Jun 1 2021



### 参考

- [廖雪峰的Java教程]()

注意，这个教程是面向零基础Java开发的同学，需要深入理解的地方还可以参考以下书单，

- Effective Java
- Head First Java
- Java核心技术 卷1 基础知识
- Java核心技术 卷2 高级特性
- Java编程思想
- Java语言程序设计 基础篇

### JVM、JDK、JRE

- JDK：Java Development Kit（Java源码编译到字节码的Compiler，debugger等 + JRE）
- JRE：Java Runtime Environment（Java字节码运行时的虚拟机，即JVM + Runtime Library）
- JVM：Java Virtual Machine（Java虚拟机）

它们的关系如下，

```ascii
  ┌─    ┌──────────────────────────────────┐
  │     │     Compiler, debugger, etc.     │
  │     └──────────────────────────────────┘
 JDK ┌─ ┌──────────────────────────────────┐
  │  │  │                                  │
  │ JRE │      JVM + Runtime Library       │
  │  │  │                                  │
  └─ └─ └──────────────────────────────────┘
        ┌───────┐┌───────┐┌───────┐┌───────┐
        │Windows││ Linux ││ macOS ││others │
        └───────┘└───────┘└───────┘└───────┘
```

一些文件后缀说明，

- java：点击运行Java程序，启动JVM，然后让JVM执行指定的编译后的代码；
- javac：Java的编译器，把Java源码文件（以`.java`后缀结尾）编译为Java字节码文件（以`.class`后缀结尾）；
- jar：用于把一组`.class`文件打包成一个`.jar`文件，便于发布；
- javadoc：用于从Java源码中自动提取注释并生成文档；
- jdb：Java调试器，用于开发阶段的运行调试。

### Java程序启动

**Java程序的入口**

Java规定，某个类定义的`public static void main(String[] args)`是Java程序的固定入口方法，因此，Java程序总是从`main`方法开始执行。

**文件命名**

Java的类名和文件名必须对应，比如写了一个`Hello`类的Java程序，当我们把代码保存为文件时，文件名必须是`Hello.java`，而且文件名也要注意大小写，因为要和我们定义的类名`Hello`完全保持一致。

**执行顺序**

```ascii
┌──────────────────┐
│    Hello.java    │<─── source code
└──────────────────┘
          │ compile
          ▼
┌──────────────────┐
│   Hello.class    │<─── byte code
└──────────────────┘
          │ execute
          ▼
┌──────────────────┐
│    Run on JVM    │
└──────────────────┘
```

执行顺序如下，

```shell
javac Hello.java
java Hello
```

也可以直接运行，

```
java Hello.java 
```

这是Java 11新增的一个功能，它可以直接运行一个单文件源码！需要注意的是，在实际项目中，单个不依赖第三方库的Java源码是非常罕见的，所以，绝大多数情况下，我们无法直接运行一个Java源码文件，原因是它需要依赖其他的库。

### Java程序基础之数据结构

Java的数据类型分两种：

- 基本类型：`byte`，`short`，`int`，`long`，`boolean`，`float`，`double`，`char`
- 引用类型：所有`class`和`interface`类型

引用类型可以赋值为`null`，表示空，但基本类型不能赋值为`null`。

基本类型要转换成引用类型，可以使用包装类（Wrapper Class）。

**Auto Boxing**

因为`int`和`Integer`可以互相转换：

```
int i = 100;
Integer n = Integer.valueOf(i);
int x = n.intValue();
```

所以，Java编译器可以帮助我们自动在`int`和`Integer`之间转型：

```
Integer n = 100; // 编译器自动使用Integer.valueOf(int)
int x = n; // 编译器自动使用Integer.intValue()
```

这种直接把`int`变为`Integer`的赋值写法，称为自动装箱（Auto Boxing），反过来，把`Integer`变为`int`的赋值写法，称为自动拆箱（Auto Unboxing）。

注意：自动装箱和自动拆箱只发生在编译阶段，目的是为了少写代码。

**所有的包装类型都是不变类。**所以，对两个`Integer`实例进行比较要特别注意：绝对不能用`==`比较，因为`Integer`是引用类型，必须使用`equals()`比较。

**基本数据类型**

基本数据类型是CPU可以直接进行运算的类型。Java定义了以下几种基本数据类型：

- 整数类型：byte，short，int，long
- 浮点数类型：float，double
- 字符类型：char
- 布尔类型：boolean

`int`类型可能会有溢出问题，要解决溢出，可以把`int`换成`long`类型，由于`long`可表示的整型范围更大，所以结果就不会溢出。

在Java数值运算中，会存在<u>类型自动提升</u>和<u>强制转换</u>，自动提升是指小范围类型会被自动转换为大范围类型，当然也可以做强制转换，如下，

```java
int i = 12345;
short s = (short) i; // 12345
```

对于`float`类型，需要加上`f`后缀。浮点数可表示的范围非常大，`float`类型可最大表示3.4x1038，而`double`类型可最大表示1.79x10308。如果参与运算的两个数其中一个是整型，那么整型也会自动提升到浮点型。

注意`char`类型使用单引号`'`，且仅有一个字符，要和双引号`"`的字符串类型区分开。

Java在内存中总是使用Unicode表示字符，所以，一个英文字符和一个中文字符都用一个`char`类型表示，它们都占用两个字节。

可以使用`+`连接任意字符串和其他数据类型。

**字符串**

`String`字符串定义如下

```java
String s = "hello";
```

注意，Java中的基础类型和字符串都是不可变的，如下，

```java
public class Main {
    public static void main(String[] args) {
        String[] names = {"ABC", "XYZ", "zoo"};
        String s = names[1];
        names[1] = "cat";
        System.out.println(s); // "XYZ"
    }
}
```

**常量**

定义变量的时候，如果加上`final`修饰符，这个变量就变成了常量。常量在定义时进行初始化后就不可再次赋值，再次赋值会导致编译错误。根据习惯，常量名通常全部大写。

```java
final double PI = 3.14; // PI是一个常量
```

**数组**

创建数组的方式，

```java
// 直接赋值
int[] ns = new int[] { 68, 79, 91, 85, 62 };
// 声明个数
int[] ns = new int[5];
// 数组排序
import java.util.Arrays;
Arrays.sort(ns);
// 二维数组
int[][] matrix=new int[5][5];
```

### Java程序基础之流程控制

- 输入和输出
- if判断
- switch多重选择
- while循环
- do while循环
- for循环
- break和continue

### Java面向对象基础

- 方法
- 构造函数
- 重载
- 继承

**多态和覆写**

在继承关系中，子类如果定义了一个与父类方法签名完全相同的方法，被称为覆写（Override）。

多态是指，针对某个类型的方法调用，其真正执行的方法取决于运行时期实际类型的方法。

因为所有的`class`最终都继承自`Object`，而`Object`定义了几个重要的方法：

- `toString()`：把instance输出为`String`；
- `equals()`：判断两个instance是否逻辑相等；
- `hashCode()`：计算一个instance的哈希值。

在必要的情况下，我们也可以覆写`Object`的这几个方法

- 抽象类
- 接口
- 静态字段、静态方法

**包（package）**

在Java中，我们使用`package`解决命名冲突。包是Java定义的一种名字空间。一个类总是属于某个包，类名（比如`Person`）只是一个简写，真正的完整类名是`包名.类名`。

在定义`class`的时候，我们需要在第一行声明这个`class`属于哪个包。在Java虚拟机执行的时候，JVM只看完整类名，因此，只要包名不同，类就不同。包可以是多层结构，用`.`隔开。例如：`java.util`。包没有父子关系。`java.util`和`java.util.zip`是不同的包，两者没有任何继承关系。

**包作用域**

位于同一个包的类，可以访问包作用域的字段和方法。不用`public`、`protected`、`private`修饰的字段和方法在同一个包作用域以内。

Java编译器最终编译出的`.class`文件只使用*完整类名*，因此，在代码中，当编译器遇到一个`class`名称时：

- 如果是完整类名，就直接根据完整类名查找这个`class`；
- 如果是简单类名，按下面的顺序依次查找：
  - 查找当前`package`是否存在这个`class`；
  - 查找`import`的包是否包含这个`class`；
  - 查找`java.lang`包是否包含这个`class`。

如果按照上面的规则还无法确定类名，则编译报错。

**局部变量**

在方法内部定义的变量称为局部变量，局部变量作用域从变量声明处开始到对应的块结束。方法参数也是局部变量。

**嵌套类/内部类**

如果一个类定义在另一个类的内部，这个类就是Inner Class。

Java的内部类可分为Inner Class、Anonymous Class和Static Nested Class三种：

- Inner Class和Anonymous Class本质上是相同的，都必须依附于Outer Class的实例，即隐含地持有`Outer.this`实例，并拥有Outer Class的`private`访问权限；
- Static Nested Class是独立类，但拥有Outer Class的`private`访问权限。

**classpath、jar**

`classpath`是JVM用到的一个环境变量，它用来指示JVM如何搜索`class`。

因为Java是编译型语言，源码文件是`.java`，而编译后的`.class`文件才是真正可以被JVM执行的字节码。因此，JVM需要知道，如果要加载一个`abc.xyz.Hello`的类，应该去哪搜索对应的`Hello.class`文件。

如果有很多`.class`文件，散落在各层目录中，肯定不便于管理。如果能把目录打一个包，变成一个文件，就方便多了。<u>jar包</u>就是用来干这个事的，它可以把`package`组织的目录层级，以及各个目录下的所有文件（包括`.class`文件和其他文件）都打成一个jar文件，这样一来，无论是备份，还是发给客户，就简单多了。jar包实际上就是一个zip格式的压缩文件，而jar包相当于目录。

> `.class`文件是JVM看到的最小可执行文件，而一个大型程序需要编写很多Class，并生成一堆`.class`文件，很不便于管理，所以，`jar`文件就是`class`文件的容器。
>
> 所以，jar只是用于存放class的容器，它并不关心class之间的依赖。

**模块**

模块是Java9引入的。

在Java 9之前，一个大型Java程序会生成自己的jar文件，同时引用依赖的第三方jar文件，而JVM自带的Java标准库，实际上也是以jar文件形式存放的，这个文件叫`rt.jar`，一共有60多M。

从Java 9开始引入的模块，主要是为了解决“依赖”这个问题。如果`a.jar`必须依赖另一个`b.jar`才能运行，那我们应该给`a.jar`加点说明啥的，让程序在编译和运行的时候能自动定位到`b.jar`，这种自带“依赖关系”的class容器就是模块。

### Java核心类、异常

**华丽地处理字符串**

在Java中，`String`是一个引用类型，它本身也是一个`class`。但是，Java编译器对`String`有特殊处理，即可以直接用`"..."`来表示一个字符串：

```
String s1 = "Hello!";
```

实际上字符串在`String`内部是通过一个`char[]`数组表示的，因此，按下面的写法也是可以的：

```
String s2 = new String(new char[] {'H', 'e', 'l', 'l', 'o', '!'});
```

因为`String`太常用了，所以Java提供了`"..."`这种字符串字面量表示方法。

当我们想要比较两个字符串是否相同时，要特别注意，我们实际上是想比较字符串的内容是否相同。必须使用`equals()`方法而不能用`==`。

```java
public class Main {
    public static void main(String[] args) {
        String s1 = "hello";
        String s2 = "hello";
        System.out.println(s1 == s2); // true
        System.out.println(s1.equals(s2)); // true
    }
}

```

从表面上看，两个字符串用`==`和`equals()`比较都为`true`，但实际上那只是Java编译器在编译期，会自动把所有相同的字符串当作一个对象放入常量池，自然`s1`和`s2`的引用就是相同的。

所以，这种`==`比较返回`true`纯属巧合。换一种写法，`==`比较就会失败：

```java
String s1 = "hello";
String s2 = "HELLO".toLowerCase();
System.out.println(s1 == s2); // false
System.out.println(s1.equals(s2)); // true
```

`String`类还提供了很多自带方法，

```java
// 常用方法
"Hello".contains("ll"); // true
"Hello".indexOf("l"); // 2
"Hello".lastIndexOf("l"); // 3
"Hello".startsWith("He"); // true
"Hello".endsWith("lo"); // true
// 提取子串
"Hello".substring(2); // "llo"
"Hello".substring(2, 4); // "ll"
// 去掉头尾空格
"  \tHello\r\n ".trim(); // "Hello"
"\u3000Hello\u3000".strip(); // "Hello"
" Hello ".stripLeading(); // "Hello "
" Hello ".stripTrailing(); // " Hello"
// 空值判断
"".isEmpty(); // true，因为字符串长度为0
"  ".isEmpty(); // false，因为字符串长度不为0
"  \n".isBlank(); // true，因为只包含空白字符
" Hello ".isBlank(); // false，因为包含非空白字符
// 替代
String s = "hello";
s.replace('l', 'w'); // "hewwo"，所有字符'l'被替换为'w'
s.replace("ll", "~~"); // "he~~o"，所有子串"ll"被替换为"~~"
// 正则表达式
String s = "A,,B;C ,D";
s.replaceAll("[,;s]+", ","); // "A,B,C,D"
// 分割
String s = "A,B,C,D";
String[] ss = s.split(","); // {"A", "B", "C", "D"}
// 拼接
String[] arr = {"A", "B", "C"};
String s = String.join("***", arr); // "A***B***C"
```

`char[]`和`String`类型相互转换，

```java
char[] cs = "Hello".toCharArray(); // String -> char[]
String s = new String(cs); // char[] -> String
```

**StringBuilder**

为了能高效拼接字符串，Java标准库提供了`StringBuilder`，它是一个可变对象，可以预分配缓冲区，这样，往`StringBuilder`中新增字符时，不会创建新的临时对象：

```java
StringBuilder sb = new StringBuilder(1024);
for (int i = 0; i < 1000; i++) {
    sb.append(',');
    sb.append(i);
}
String s = sb.toString();
```

**StringJoiner**

Java标准库还提供了一个`StringJoiner`来实现分隔符拼接数组的需求。

```java
String[] names = {"Bob", "Alice", "Grace"};
var sj = new StringJoiner(", ", "Hello ", "!");
for (String name : names) {
  sj.add(name);
}
System.out.println(sj.toString()); // Hello Bob, Alice, Grace!
```

`String`还提供了一个静态方法`join()`，这个方法在内部使用了`StringJoiner`来拼接字符串，在不需要指定“开头”和“结尾”的时候，用`String.join()`更方便。

```java
String[] names = {"Bob", "Alice", "Grace"};
var s = String.join(", ", names);
```

**JavaBean**

在Java中，有很多`class`的定义都符合这样的规范：

- 若干`private`实例字段；
- 通过`public`方法来读写实例字段。

如果读写方法符合以下这种命名规范：

```
// 读方法:
public Type getXyz()
// 写方法:
public void setXyz(Type value)
```

那么这种`class`被称为`JavaBean`：

JavaBean主要用来传递数据，即把一组数据组合成一个JavaBean便于传输。此外，JavaBean可以方便地被IDE工具分析，生成读写属性的代码，主要用在图形界面的可视化设计中。

要枚举一个JavaBean的所有属性，可以直接使用Java核心库提供的`Introspector`。

**枚举类型**

```java
enum Weekday {
    SUN, MON, TUE, WED, THU, FRI, SAT;
}

// 常量名
String s = Weekday.SUN.name(); // "SUN"
// 定义时的顺序
int n = Weekday.MON.ordinal(); // 1

```

最后，枚举类可以应用在`switch`语句中。因为枚举类天生具有类型信息和有限个枚举常量，所以比`int`、`String`类型更适合用在`switch`语句中。

**BigInteger**

在Java中，由CPU原生提供的整型最大范围是64位`long`型整数。使用`long`型整数可以直接通过CPU指令进行计算，速度非常快。

如果我们使用的整数范围超过了`long`型怎么办？这个时候，就只能用软件来模拟一个大整数。`java.math.BigInteger`就是用来表示任意大小的整数。

```java
BigInteger bi = new BigInteger("1234567890");
System.out.println(bi.pow(5)); // 很大的输出
// 只能用内置方法做运算
BigInteger i1 = new BigInteger("1234567890");
BigInteger i2 = new BigInteger("12345678901234567890");
BigInteger sum = i1.add(i2); 
```

如果带有小数，还可以使用`BigDecimal`。

**Math、Random**

数学计算、随机数生成类。

### 异常处理

略。

### 反射、注解、泛型

这三个东西概念性都比较强，要在实际应用中去体会，这里仅给出一些简单解释。

- 反射就是Reflection，Java的反射是指程序在运行期可以拿到一个对象的所有信息。
- 注解是放在Java源码的类、方法、字段、参数前的一种特殊“注释”。
- 泛型是一种“代码模板”，可以用一套代码套用各种类型。

### Java集合






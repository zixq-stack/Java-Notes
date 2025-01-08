# JVM整体结构

> 本文主要说的是HotSpot虚拟机，



JVM 全称是 Java Virtual Machine，中文译名：Java虚拟机

<img src="https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031181910930-41332754.jpg" alt="JVM-framework" />



简化一下：

![image-20231110204142543](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231110204142530-1227278687.png)









# Java字节码文件

> Class文件本质上是一个以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑的排列在Class文件中，JVM根据其特定的规则解析该二进制数据，从而得到相关信息
>
> Class文件采用一种伪结构来存储数据，它有两种类型：无符号数和表

首先从整体上看一下Java字节码文件所包含的内容：

![image-20231031215759601](https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031215800890-1616844151.png)









## 初识Class文件、基础信息

```java
package com.zixieqing;

public class KnowClass {
    static int a = 0;

    public static void main(String[] args) {
        int b = a++;

        System.out.println("b = " + b);
    }
}
```

通过以下命令, 可以在当前所在路径下生成一个 .Class 文件

```java
javac KnowClass.java
```

使用NotePad++的十六进制插件（HEX-Editor）打开编译后的Class文件，部分截图如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031223853827-38591583.png" alt="image-20231031223852875" />



其中：

- 左边Address这一列：是当前文件中的地址
- 中间部分：是整个十六进制数据
- 右边Dump这一列：是编码之后的结果

对于中间部分数据：

1. 文件开头的4个字节（“cafe babe”）就是所谓的“magic魔数”。唯有以"cafe babe"开头的Class文件方可被虚拟机所接受，这4个字节就是字节码文件的身份识别

文件是无法通过文件扩展名来确定文件类型的，文件扩展名可以随意修改，不影响文件的内容

软件使用文件的头几个字节（文件头）去校验文件的类型，如果软件不支持该种类型就会出错

**Java字节码文件中，将文件头称为magic魔数**

![image-20231031214631078](https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031214632056-1402615546.png)



2. 0000是编译器JDK版本的次版本号0，0034转化为十进制是52，是主版本号

主次版本号指的是编译字节码文件的JDK版本号

主版本号用来标识大版本号

JDK1.0-1.1使用了45.0-45.3，JDK1.2是46之后每升级一个大版本就加1；副版本号是当主版本号相同时作为区分不同版本的标识，一般只需要关心主版本号

```txt
1.2之后大版本号计算方法就是:
主版本号 – 44
比如主版本号52就是52 - 44 = 8，即JDK8

以前用的 Java -version 命令也就可以验证
PS C:\Users\zixq\Desktop> java -version
java version "1.8.0_221"
Java(TM) SE Runtime Environment (build 1.8.0_221-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.221-b11, mixed mode)
```

**版本号的作用主要是判断当前字节码的版本和运行时的JDK是否兼容**

```txt
主版本号不兼容导致的错误的两种解决方案：
1.升级JDK版本													（容易引发其他的兼容性问题，并且需要大量的测试）
2.将第三方依赖的版本号降低或者更换依赖，以满足JDK版本的要求			√ 建议采用
```







## 反编译Class文件

> 使用Java内置的一个反编译工具Javap可以反编译字节码文件, 用法: `Javap <options> <Classes>`

其中`<options>`选项包括:

```bash
-help  --help  -?        输出此用法消息
-version                 版本信息
-v  -verbose             输出附加信息
-l                       输出行号和本地变量表
-public                  仅显示公共类和成员
-protected               显示受保护的/公共类和成员
-package                 显示程序包/受保护的/公共类和成员 (默认)
-p  -private             显示所有类和成员
-c                       对代码进行反汇编
-s                       输出内部类型签名
-sysinfo                 显示正在处理的类的系统信息 (路径, 大小, 日期, MD5 散列)
-constants               显示最终常量
-Classpath <path>        指定查找用户类文件的位置
-cp <path>               指定查找用户类文件的位置
-bootClasspath <path>    覆盖引导类文件的位置
```

输入命令`Javap -verbose -p KnowClass.Class`查看输出内容:

```java
Classfile /E:/Study/JVM-Demo/out/production/JVM-Demo/com/zixieqing/KnowClass.class			// Class文件当前所在位置
  Last modified 2023-10-31; size 862 bytes													// 最后修改时间、文件大小
  MD5 checksum 1b6100d02bb70d920adceac139839609												// MD5值
  Compiled from "KnowClass.java"															// 编译自哪个文件
public class com.zixieqing.KnowClass														// 类全限定名
  minor version: 0																			// 次版本号
  major version: 52																			// 主版本号
  flags: ACC_PUBLIC, ACC_SUPER																// 该类的访问标志	一会儿单独说明有哪些
Constant pool:																				// 常量池
   #1 = Methodref          #12.#30        // java/lang/Object."<init>":()V
   #2 = Fieldref           #11.#31        // com/zixieqing/KnowClass.a:I
   #3 = Fieldref           #32.#33        // java/lang/System.out:Ljava/io/PrintStream;
   #4 = Class              #34            // java/lang/StringBuilder
   #5 = Methodref          #4.#30         // java/lang/StringBuilder."<init>":()V
   #6 = String             #35            // b =
   #7 = Methodref          #4.#36         // java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   #8 = Methodref          #4.#37         // java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
   #9 = Methodref          #4.#38         // java/lang/StringBuilder.toString:()Ljava/lang/String;
  #10 = Methodref          #39.#40        // java/io/PrintStream.println:(Ljava/lang/String;)V
  #11 = Class              #41            // com/zixieqing/KnowClass
  #12 = Class              #42            // java/lang/Object
  #13 = Utf8               a
  #14 = Utf8               I
  #15 = Utf8               <init>
  #16 = Utf8               ()V
  #17 = Utf8               Code
  #18 = Utf8               LineNumberTable
  #19 = Utf8               LocalVariableTable
  #20 = Utf8               this
  #21 = Utf8               Lcom/zixieqing/KnowClass;
  #22 = Utf8               main
  #23 = Utf8               ([Ljava/lang/String;)V
  #24 = Utf8               args
  #25 = Utf8               [Ljava/lang/String;
  #26 = Utf8               b
  #27 = Utf8               <clinit>
  #28 = Utf8               SourceFile
  #29 = Utf8               KnowClass.java
  #30 = NameAndType        #15:#16        // "<init>":()V
  #31 = NameAndType        #13:#14        // a:I
  #32 = Class              #43            // java/lang/System
  #33 = NameAndType        #44:#45        // out:Ljava/io/PrintStream;
  #34 = Utf8               java/lang/StringBuilder
  #35 = Utf8               b =
  #36 = NameAndType        #46:#47        // append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #37 = NameAndType        #46:#48        // append:(I)Ljava/lang/StringBuilder;
  #38 = NameAndType        #49:#50        // toString:()Ljava/lang/String;
  #39 = Class              #51            // java/io/PrintStream
  #40 = NameAndType        #52:#53        // println:(Ljava/lang/String;)V
  #41 = Utf8               com/zixieqing/KnowClass
  #42 = Utf8               java/lang/Object
  #43 = Utf8               java/lang/System
  #44 = Utf8               out
  #45 = Utf8               Ljava/io/PrintStream;
  #46 = Utf8               append
  #47 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #48 = Utf8               (I)Ljava/lang/StringBuilder;
  #49 = Utf8               toString
  #50 = Utf8               ()Ljava/lang/String;
  #51 = Utf8               java/io/PrintStream
  #52 = Utf8               println
  #53 = Utf8               (Ljava/lang/String;)V
{
  static int a;
    descriptor: I
    flags: ACC_STATIC

  public com.zixieqing.KnowClass();				// 方法表集合，就是前面看Class整体中说的“方法”
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 12: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/zixieqing/KnowClass;

  public static void main(java.lang.String[]);	// 方法表集合		一会儿说明
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=2, args_size=1
         0: getstatic     #2                  // Field a:I
         3: dup
         4: iconst_1
         5: iadd
         6: putstatic     #2                  // Field a:I
         9: istore_1
        10: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        13: new           #4                  // class java/lang/StringBuilder
        16: dup
        17: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
        20: ldc           #6                  // String b =
        22: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        25: iload_1
        26: invokevirtual #8                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
        29: invokevirtual #9                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        32: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        35: return
      LineNumberTable:
        line 16: 0
        line 18: 10
        line 19: 35
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      36     0  args   [Ljava/lang/String;
           10      26     1     b   I

  static {};
    descriptor: ()V
    flags: ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: iconst_0
         1: putstatic     #2                  // Field a:I
         4: return
      LineNumberTable:
        line 13: 0
}
SourceFile: "KnowClass.java"				// 资源文件名	即源代码的文件名
```

上面提到了类的访问标志：ACC_PUBLIC, ACC_SUPER，访问标志的含义如下:

| 标志名称       | 标志值 | 含义                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| ACC_PUBLIC     | 0x0001 | 是否为Public类型                                             |
| ACC_FINAL      | 0x0010 | 是否被声明为final，只有类可以设置                            |
| ACC_SUPER      | 0x0020 | 是否允许使用invokespecial字节码指令的新语义．                |
| ACC_INTERFACE  | 0x0200 | 标志这是一个接口                                             |
| ACC_ABSTRACT   | 0x0400 | 是否为abstract类型，对于接口或者抽象类来说，次标志值为真，其他类型为假 |
| ACC_SYNTHETIC  | 0x1000 | 标志这个类并非由用户代码产生                                 |
| ACC_ANNOTATION | 0x2000 | 标志这是一个注解                                             |
| ACC_ENUM       | 0x4000 | 标志这是一个枚举                                             |







## 常量池

`Constant pool`意为常量池

常量池中的数据都有一个编号，编号从1开始。在字段或者字节码指令中通过编号可以快速的找到对应的数据

字节码指令中通过编号引用到常量池的过程称之为“符号引用”

常量池可以理解成Class文件中的资源仓库，主要存放的是两大类常量：字面量(Literal)和符号引用(Symbolic References)，字面量类似于Java中的常量概念，如文本字符串，final常量等，而符号引用则属于编译原理方面的概念，包括以下三种:

- 类和接口的全限定名(Fully Qualified Name)
- 字段的名称和描述符号(Descriptor)
- 方法的名称和描述符

不同于C/C++,，JVM是在加载Class文件的时候才进行的动态链接，也就是说这些字段和方法符号引用只有在运行期转换后才能获得真正的内存入口地址，当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建或运行时解析并翻译到具体的内存地址中。如上一节反编译的文件中的常量池：

```java
Constant pool:							  // 常量池
   #1 = Methodref          #12.#30        // java/lang/Object."<init>":()V
   #2 = Fieldref           #11.#31        // com/zixieqing/KnowClass.a:I
   #3 = Fieldref           #32.#33        // java/lang/System.out:Ljava/io/PrintStream;
   #4 = Class              #34            // java/lang/StringBuilder
   #5 = Methodref          #4.#30         // java/lang/StringBuilder."<init>":()V
   #6 = String             #35            // b =
   #7 = Methodref          #4.#36         // java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   #8 = Methodref          #4.#37         // java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
   #9 = Methodref          #4.#38         // java/lang/StringBuilder.toString:()Ljava/lang/String;
  #10 = Methodref          #39.#40        // java/io/PrintStream.println:(Ljava/lang/String;)V
  #11 = Class              #41            // com/zixieqing/KnowClass
  #12 = Class              #42            // java/lang/Object
  #13 = Utf8               a
  #14 = Utf8               I
  #15 = Utf8               <init>
  #16 = Utf8               ()V
  #17 = Utf8               Code
  #18 = Utf8               LineNumberTable
  #19 = Utf8               LocalVariableTable
  #20 = Utf8               this
  #21 = Utf8               Lcom/zixieqing/KnowClass;
  #22 = Utf8               main
  #23 = Utf8               ([Ljava/lang/String;)V
  #24 = Utf8               args
  #25 = Utf8               [Ljava/lang/String;
  #26 = Utf8               b
  #27 = Utf8               <clinit>
  #28 = Utf8               SourceFile
  #29 = Utf8               KnowClass.java
  #30 = NameAndType        #15:#16        // "<init>":()V
  #31 = NameAndType        #13:#14        // a:I
  #32 = Class              #43            // java/lang/System
  #33 = NameAndType        #44:#45        // out:Ljava/io/PrintStream;
  #34 = Utf8               java/lang/StringBuilder
  #35 = Utf8               b =
  #36 = NameAndType        #46:#47        // append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #37 = NameAndType        #46:#48        // append:(I)Ljava/lang/StringBuilder;
  #38 = NameAndType        #49:#50        // toString:()Ljava/lang/String;
  #39 = Class              #51            // java/io/PrintStream
  #40 = NameAndType        #52:#53        // println:(Ljava/lang/String;)V
  #41 = Utf8               com/zixieqing/KnowClass
  #42 = Utf8               java/lang/Object
  #43 = Utf8               java/lang/System
  #44 = Utf8               out
  #45 = Utf8               Ljava/io/PrintStream;
  #46 = Utf8               append
  #47 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #48 = Utf8               (I)Ljava/lang/StringBuilder;
  #49 = Utf8               toString
  #50 = Utf8               ()Ljava/lang/String;
  #51 = Utf8               java/io/PrintStream
  #52 = Utf8               println
  #53 = Utf8               (Ljava/lang/String;)V
```

**第一个常量**是一个方法定义，指向了第12和第30个常量。以此类推查看第12和第30个常量最后可以拼接成第一个常量右侧的注释内容:

```java
// java/lang/Object."<init>":()V
```

这段可以理解为该类的实例构造器的声明，由于Main类没有重写构造方法，所以调用的是父类的构造方法。此处也说明了Main类的直接父类是Object，该方法默认返回值是V, 也就是void，无返回值

**第二个常量**同理可得:

```java
  #2 = Fieldref           #11.#31        // com/zixieqing/KnowClass.a:I
  #11 = Class              #41            // com/zixieqing/KnowClass
  #13 = Utf8               a
  #14 = Utf8               I
  #41 = Utf8               com/zixieqing/KnowClass
  #31 = NameAndType        #13:#14        // a:I
```

此处声明了一个字段a，类型为I，I即为int类型。关于字节码的类型对应如下：

| 标识字符 | 含义                                       | 备注   |
| -------- | ------------------------------------------ | ------ |
| B        | 基本类型byte                               |        |
| C        | 基本类型char                               |        |
| D        | 基本类型double                             |        |
| F        | 基本类型float                              |        |
| I        | 基本类型int                                |        |
| J        | 基本类型long                               | 特殊记 |
| S        | 基本类型short                              |        |
| Z        | 基本类型boolean                            | 特殊记 |
| V        | 特殊类型void                               |        |
| L        | 对象类型，以分号结尾，如LJava/lang/Object; | 特殊记 |

对于数组类型，每一位使用一个前置的`[`字符来描述，如定义一个`Java.lang.String[][]`类型的二维数组，将被记录为`[[LJava/lang/String;`













## 方法表集合

在常量池之后的是对类内部的方法描述，在字节码中以表的集合形式表现，暂且不管字节码文件的16进制文件内容如何，我们直接看反编译后的内容

```java
  static int a;
    descriptor: I
    flags: ACC_STATIC
```

此处声明了一个static的变量a，类型为int

```java
  public com.zixieqing.KnowClass();				// 方法表集合，就是前面看Class整体中说的“方法”
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 12: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/zixieqing/KnowClass;
```

这里是构造方法：KnowClass()，返回值为void, 权限修饰符为public

code内的主要属性为:

- **stack**:：最大操作数栈，JVM运行时会根据这个值来分配栈帧(Frame)中的操作栈深度,此处为1
- **locals**:：局部变量所需的存储空间，单位为Slot, Slot是虚拟机为局部变量分配内存时所使用的最小单位，为4个字节大小，方法参数(包括实例方法中的隐藏参数this)，显示异常处理器的参数(try catch中的catch块所定义的异常)，方法体中定义的局部变量都需要使用局部变量表来存放。值得一提的是，locals的大小并不一定等于所有局部变量所占的Slot之和，因为局部变量中的Slot是可以重用的
- **args_size**:：方法参数的个数，这里是1，因为每个实例方法都会有一个隐藏参数this
- **attribute_info**:：方法体内容，0,1,4为字节码"行号"，该段代码的意思是将第一个引用类型本地变量推送至栈顶，然后执行该类型的实例方法，也就是常量池存放的第一个变量，也就是注释里的`Java/lang/Object."":()V`, 然后执行返回语句，结束方法
- **LineNumberTable**:：该属性的作用是描述源码行号与字节码行号(字节码偏移量)之间的对应关系。可以使用 `-g:none` 或`-g:lines` 选项来取消或要求生成这项信息，如果选择不生成LineNumberTable，当程序运行异常时将无法获取到发生异常的源码行号，也无法按照源码的行数来调试程序
- **LocalVariableTable**：该属性的作用是描述帧栈中局部变量与源码中定义的变量之间的关系。可以使用 `-g:none` 或` -g:vars` 来取消或生成这项信息，如果没有生成这项信息，那么当别人引用这个方法时，将无法获取到参数名称，取而代之的是arg0, arg1这样的占位符 

```txt
start		    表示该局部变量在哪一行开始可见
length		    表示可见行数	start和length也可以称之为变量的作用域。从字节码的start 到 length行这个作用域里该变量一直有效/可见
Slot		    代表所在帧栈位置/变量槽的索引
Name		    变量名称
Signature		类型签名/变量类型		就是前面“常量池”中说的字节码的类型
```



上面Code中有一个小东西：`0: aload_0`，这里的 `aload_0` 叫做虚拟机字节码指令

> 关于更多虚拟机字节码指令，也可以在《深入理解Java虚拟机 ：JVM高级特性与最佳实践-附录B》中获取：[深入理解Java虚拟机 ：JVM高级特性与最佳实践_周志明_V3 下载](https://github.com/zixq-stack/Java-Note/blob/master/Java/JVM%EF%BC%9AJava%E8%99%9A%E6%8B%9F%E6%9C%BA/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%EF%BC%9AJVM%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E4%B8%8E%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%E3%80%90%40%E5%91%A8%E5%BF%97%E6%98%8E%20-%20%E7%AC%AC3%E7%89%88%E3%80%91.pdf)

这里说几个最基本的虚拟机字节码指令：

<img src="https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031231841053-1227524628.png" alt="image-20231031231839977" />



再来个示例：

<img src="https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031231920242-1520087013.png" alt="image-20231031231919319" />









## 字节码常用工具

### javap命令

这个命令前面已经玩过

javap是JDK自带的反编译工具，可以通过控制台查看字节码文件的内容。**适合在服务器上查看字节码文件内容**

直接输入javap查看所有参数

输入` javap -v 字节码文件名称` 查看具体的字节码信息。（jar包需要先使用 jar –xvf 命令解压）

![image-20231031233150816](https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031233151837-1538049901.png)







### jclasslib工具

1. exe方式安装：https://github.com/ingokegel/jclasslib
2. IDEA集成插件：直接在IDEA的plugins中搜索jclasslib即可

![image-20231031233459824](https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031233500561-92800441.png)



![image-20231031233844283](https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031233845402-930599476.png)



**注意点：**源代码改变之后需要重编译，然后刷新jclasslib，否则看到的就是旧的class文件

![image-20231031234134747](https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031234135753-23584882.png)



附加：遇到不会的虚拟机字节码指令时，可以通过jclasslib中右键对应指令，跳入官方文档查看描述

![image-20231031234427465](https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031234428335-284470540.png)





### 阿里Arthas

Arthas 是一款线上监控诊断产品，通过全局视角实时查看应用 load、内存、gc、线程的状态信息，并能在不修改应用代码的情况下，对业务问题进行诊断，大大提升线上问题排查效率

官网：https://arthas.aliyun.com/doc/

<img src="https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031234600724-1208870057.png" alt="image-20231031234559729" />



1. 下载jar包，运行jar包：本地查看就下载jar包到本地，Linux中查看线上代码就丢在Linux中即可

````bash
java -jar xxx.jar
````

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231104213441973-819577938.png" alt="image-20231104213439088" />



进入后使用示例：更多命令参考官网 https://arthas.aliyun.com/doc/commands.html

```bash
dump 类的全限定名		命令含义：dump已加载类的字节码文件到特定目录
					场景：线上查看Class文件，就可选择将此Class文件整到自己规定的目录中去
					
jad 类的全限定名		命令含义：反编译已加载类的源码			
					场景：BUG修复，然后上线，但BUG还在，就可选择此命令将Class文件反编译为源代码（即xxxx.java文件）
						 然后看部署的是修复好BUG的代码 还是 旧代码
```







# 字节码增强技术

> 声明：本章节内容转载于 https://www.pdai.tech/md/java/jvm/java-jvm-class-enhancer.html



## 字节码增强技术

在上文中，着重介绍了字节码的结构，这为我们了解字节码增强技术的实现打下了基础。字节码增强技术就是一类对现有字节码进行修改或者动态生成全新字节码文件的技术。接下来，我们将从最直接操纵字节码的实现方式开始深入进行剖析

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511122938700-1810862780.png)

### ASM

对于需要手动操纵字节码的需求，可以使用ASM，它可以直接生产 .Class字节码文件，也可以在类被加载入JVM之前动态修改类行为（如下图所示）ASM的应用场景有AOP（Cglib就是基于ASM）、热部署、修改其他jar包中的类等。当然，涉及到如此底层的步骤，实现起来也比较麻烦。接下来，本文将介绍ASM的两种API，并用ASM来实现一个比较粗糙的AOP。但在此之前，为了让大家更快地理解ASM的处理流程，强烈建议读者先对访问者模式进行了解。简单来说，访问者模式主要用于修改或操作一些数据结构比较稳定的数据，而通过第一章，我们知道字节码文件的结构是由JVM固定的，所以很适合利用访问者模式对字节码文件进行修改

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511122953285-894496650.png)

#### ASM API

##### 核心API

ASM Core API可以类比解析XML文件中的SAX方式，不需要把这个类的整个结构读取进来，就可以用流式的方法来处理字节码文件，好处是非常节约内存，但是编程难度较大。然而出于性能考虑，一般情况下编程都使用Core AP。I在Core API中有以下几个关键类：

- ClassReader：用于读取已经编译好的.Class文件
- ClassWriter：用于重新构建编译后的类，如修改类名、属性以及方法，也可以生成新的类的字节码文件
- 各种Visitor类：如上所述，Core API根据字节码从上到下依次处理，对于字节码文件中不同的区域有不同的Visitor，比如用于访问方法的MethodVisitor、用于访问类变量的FieldVisitor、用于访问注解的AnnotationVisitor等。为了实现AOP，重点要使用的是MethodVisitor



##### 树形API

ASM Tree API可以类比解析XML文件中的DOM方式，把整个类的结构读取到内存中，缺点是消耗内存多，但是编程比较简单。Tree Api不同于Core API，Tree API通过各种Node类来映射字节码的各个区域，类比DOM节点，就可以很好地理解这种编程方式

#### 直接利用ASM实现AOP

利用ASM的Core API来增强类，这里不纠结于AOP的专业名词如切片、通知，只实现在方法调用前、后增加逻辑，通俗易懂且方便理解，首先定义需要被增强的Base类：其中只包含一个process()方法，方法内输出一行“process”。增强后，我们期望的是，方法执行前输出“start”，之后输出”end”

```java
public Class Base {
    public void process(){
        System.out.println("process");
    }
}
```

为了利用ASM实现AOP，需要定义两个类：一个是MyClassVisitor类，用于对字节码的visit以及修改；另一个是Generator类，在这个类中定义ClassReader和ClassWriter，其中的逻辑是，ClassReader读取字节码，然后交给MyClassVisitor类处理，处理完成后由ClassWriter写字节码并将旧的字节码替换掉，Generator类较简单，我们先看一下它的实现，如下所示，然后重点解释MyClassVisitor类

```java
import org.objectweb.asm.ClassReader;
import org.objectweb.asm.ClassVisitor;
import org.objectweb.asm.ClassWriter;

public Class Generator {
    public static void main(String[] args) throws Exception {
		// 读取
        ClassReader ClassReader = new ClassReader("meituan/bytecode/asm/Base");
        ClassWriter ClassWriter = new ClassWriter(ClassWriter.COMPUTE_MAXS);
        // 处理
        ClassVisitor ClassVisitor = new MyClassVisitor(ClassWriter);
        ClassReader.accept(ClassVisitor, ClassReader.SKIP_DEBUG);
        byte[] data = ClassWriter.toByteArray();
        // 输出
        File f = new File("operation-server/target/Classes/meituan/bytecode/asm/Base.Class");
        FileOutputStream fout = new FileOutputStream(f);
        fout.write(data);
        fout.close();
        System.out.println("now generator cc success!!!!!");
    }
}
```

MyClassVisitor继承自ClassVisitor，用于对字节码的观察，它还包含一个内部类MyMethodVisitor，继承自MethodVisitor，用于对类内方法的观察，它的整体代码如下：

```java
import org.objectweb.asm.ClassVisitor;
import org.objectweb.asm.MethodVisitor;
import org.objectweb.asm.Opcodes;

public Class MyClassVisitor extends ClassVisitor implements Opcodes {
    
    public MyClassVisitor(ClassVisitor cv) {
        super(ASM5, cv);
    }
    
    @Override
    public void visit(int version, int access, String name, String signature,
                      String superName, String[] interfaces) {
        cv.visit(version, access, name, signature, superName, interfaces);
    }
    
    @Override
    public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {
        
        MethodVisitor mv = cv.visitMethod(access, name, desc, signature,exceptions);
        // Base类中有两个方法：无参构造以及process方法，这里不增强构造方法
        if (!name.equals("<init>") && mv != null) {
            mv = new MyMethodVisitor(mv);
        }
        return mv;
    }
    
    Class MyMethodVisitor extends MethodVisitor implements Opcodes {
        public MyMethodVisitor(MethodVisitor mv) {
            super(Opcodes.ASM5, mv);
        }

        @Override
        public void visitCode() {
            super.visitCode();
            mv.visitFieldInsn(GETSTATIC, "Java/lang/System", "out", "LJava/io/PrintStream;");
            mv.visitLdcInsn("start");
            mv.visitMethodInsn(INVOKEVIRTUAL, "Java/io/PrintStream", "println", "(LJava/lang/String;)V", false);
        }
        @Override
        public void visitInsn(int opcode) {
            if ((opcode >= Opcodes.IRETURN && opcode <= Opcodes.RETURN)
                    || opcode == Opcodes.ATHROW) {
                // 方法在返回之前，打印"end"
                mv.visitFieldInsn(GETSTATIC, "Java/lang/System", "out", "LJava/io/PrintStream;");
                mv.visitLdcInsn("end");
                mv.visitMethodInsn(INVOKEVIRTUAL, "Java/io/PrintStream", "println", "(LJava/lang/String;)V", false);
            }
            mv.visitInsn(opcode);
        }
    }
}
```

利用这个类就可以实现对字节码的修改详细解读其中的代码，对字节码做修改的步骤是：

- 首先通过MyClassVisitor类中的visitMethod方法，判断当前字节码读到哪一个方法了跳过构造方法 `<init>` 后，将需要被增强的方法交给内部类MyMethodVisitor来进行处理
- 接下来，进入内部类MyMethodVisitor中的visitCode方法，它会在ASM开始访问某一个方法的Code区时被调用，重写visitCode方法，将AOP中的前置逻辑就放在这里，MyMethodVisitor继续读取字节码指令，每当ASM访问到无参数指令时，都会调用MyMethodVisitor中的visitInsn方法，我们判断了当前指令是否为无参数的“return”指令，如果是就在它的前面添加一些指令，也就是将AOP的后置逻辑放在该方法中
- 综上，重写MyMethodVisitor中的两个方法，就可以实现AOP了，而重写方法时就需要用ASM的写法，手动写入或者修改字节码，通过调用methodVisitor的visitXXXXInsn()方法就可以实现字节码的插入，XXXX对应相应的操作码助记符类型，比如mv.visitLdcInsn(“end”)对应的操作码就是ldc “end”，即将字符串“end”压入栈 完成这两个visitor类后，运行Generator中的main方法完成对Base类的字节码增强，增强后的结果可以在编译后的target文件夹中找到Base.Class文件进行查看，可以看到反编译后的代码已经改变了然后写一个测试类MyTest，在其中new Base()，并调用base.process()方法，可以看到下图右侧所示的AOP实现效果：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123049539-1608513746.png)

#### ASM工具

利用ASM手写字节码时，需要利用一系列visitXXXXInsn()方法来写对应的助记符，所以需要先将每一行源代码转化为一个个的助记符，然后通过ASM的语法转换为visitXXXXInsn()，这种写法第一步将源码转化为助记符就已经够麻烦了，不熟悉字节码操作集合的话，需要我们将代码编译后再反编译，才能得到源代码对应的助记符；第二步利用ASM写字节码时，如何传参也很令人头疼，ASM社区也知道这两个问题，所以提供了工具[ASM Byte Code Outline](https://plugins.jetbrains.com/plugin/5918-asm-bytecode-outline)

安装后，右键选择“Show Bytecode Outline”，在新标签页中选择“ASMified”这个tab，如下图所示，就可以看到这个类中的代码对应的ASM写法了，图中上下两个红框分别对应AOP中的前置逻辑与后置逻辑，将这两块直接复制到visitor中的visitMethod()以及visitInsn()方法中，就可以了

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123110707-1707923633.png)

### Javassist

ASM是在指令层次上操作字节码的，阅读上文后，我们的直观感受是在指令层次上操作字节码的框架实现起来比较晦涩，故除此之外，我们再简单介绍另外一类框架：强调源代码层次操作字节码的框架Javassist

利用Javassist实现字节码增强时，可以无须关注字节码刻板的结构，其优点就在于编程简单直接使用Java编码的形式，而不需要了解虚拟机指令，就能动态改变类的结构或者动态生成类，其中最重要的是ClassPool、CtClass、CtMethod、CtField这四个类：

- CtClass（compile-time Class）：编译时类信息，它是一个Class文件在代码中的抽象表现形式，可以通过一个类的全限定名来获取一个CtClass对象，用来表示这个类文件
- ClassPool：从开发视角来看，ClassPool是一张保存CtClass信息的HashTable，key为类名，value为类名对应的CtClass对象，当我们需要对某个类进行修改时，就是通过pool.getCtClass(“ClassName”)方法从pool中获取到相应的CtClass
- CtMethod、CtField：这两个比较好理解，对应的是类中的方法和属性

了解这四个类后，我们可以写一个小Demo来展示Javassist简单、快速的特点，我们依然是对Base中的process()方法做增强，在方法调用前后分别输出”start”和”end”，实现代码如下，我们需要做的就是从pool中获取到相应的CtClass对象和其中的方法，然后执行method.insertBefore和insertAfter方法，参数为要插入的Java代码，再以字符串的形式传入即可，实现起来也极为简单

```java
import com.meituan.mtrace.agent.Javassist.*;

public Class JavassistTest {
    public static void main(String[] args) throws NotFoundException, CannotCompileException, IllegalAccessException, InstantiationException, IOException {
        ClassPool cp = ClassPool.getDefault();
        CtClass cc = cp.get("meituan.bytecode.Javassist.Base");
        CtMethod m = cc.getDeclaredMethod("process");
        m.insertBefore("{ System.out.println(\"start\"); }");
        m.insertAfter("{ System.out.println(\"end\"); }");
        Class c = cc.toClass();
        cc.writeFile("/Users/zen/projects");
        Base h = (Base)c.newInstance();
        h.process();
    }
}
```

## 运行时类的重载

### 问题引出

上一章重点介绍了两种不同类型的字节码操作框架，且都利用它们实现了较为粗糙的AOP。其实，为了方便大家理解字节码增强技术，在上文中我们避重就轻将ASM实现AOP的过程分为了两个main方法：第一个是利用MyClassVisitor对已编译好的Class文件进行修改，第二个是new对象并调用，这期间并不涉及到JVM运行时对类的重加载，而是在第一个main方法中，通过ASM对已编译类的字节码进行替换，在第二个main方法中，直接使用已替换好的新类信息，另外在Javassist的实现中，我们也只加载了一次Base类，也不涉及到运行时重加载类

如果我们在一个JVM中，先加载了一个类，然后又对其进行字节码增强并重新加载会发生什么呢？模拟这种情况，只需要我们在上文中Javassist的Demo中main()方法的第一行添加Base b=new Base()，即在增强前就先让JVM加载Base类，然后在执行到c.toClass()方法时会抛出错误，如下图20所示跟进c.toClass()方法中，我们会发现它是在最后调用了ClassLoader的native方法defineClass()时报错，也就是说，JVM是不允许在运行时动态重载一个类的

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123143951-474784759.png)

显然，如果只能在类加载前对类进行强化，那字节码增强技术的使用场景就变得很窄了。我们期望的效果是：在一个持续运行并已经加载了所有类的JVM中，还能利用字节码增强技术对其中的类行为做替换并重新加载，为了模拟这种情况，我们将Base类做改写，在其中编写main方法，每五秒调用一次process()方法，在process()方法中输出一行“process”

我们的目的就是，在JVM运行中的时候，将process()方法做替换，在其前后分别打印“start”和“end”，也就是在运行中时，每五秒打印的内容由”process”变为打印”start process end”。那如何解决JVM不允许运行时重加载类信息的问题呢？为了达到这个目的，我们接下来一一来介绍需要借助的Java类库

```java
import Java.lang.management.ManagementFactory;

public Class Base {
    public static void main(String[] args) {
        String name = ManagementFactory.getRuntimeMXBean().getName();
        String s = name.split("@")[0];
        // 打印当前Pid
        System.out.println("pid:"+s);
        while (true) {
            try {
                Thread.sleep(5000L);
            } catch (Exception e) {
                break;
            }
            process();
        }
    }

    public static void process() {
        System.out.println("process");
    }
}
```

### Instrument

instrument是JVM提供的一个可以修改已加载类的类库，专门为Java语言编写的插桩服务，提供支持它需要依赖JVMTI的Attach API机制实现，JVMTI这一部分，我们将在下一小节进行介绍，在JDK 1.6以前，instrument只能在JVM刚启动开始加载类时生效，而在JDK 1.6之后，instrument支持了在运行时对类定义的修改，要使用instrument的类修改功能，我们需要实现它提供的ClassFileTransformer接口，定义一个类文件转换器，接口中的transform()方法会在类文件被加载时调用，而在transform方法里，我们可以利用上文中的ASM或Javassist对传入的字节码进行改写或替换，生成新的字节码数组后返回

我们定义一个实现了ClassFileTransformer接口的类TestTransformer，依然在其中利用Javassist对Base类中的process()方法进行增强，在前后分别打印“start”和“end”，代码如下：

```java
import Java.lang.instrument.ClassFileTransformer;

public Class TestTransformer implements ClassFileTransformer {
    @Override
    public byte[] transform(ClassLoader loader, String ClassName, Class<?> ClassBeingRedefined, ProtectionDomain protectionDomain, byte[] ClassfileBuffer) {
        System.out.println("Transforming " + ClassName);
        try {
            ClassPool cp = ClassPool.getDefault();
            CtClass cc = cp.get("meituan.bytecode.JVMti.Base");
            CtMethod m = cc.getDeclaredMethod("process");
            m.insertBefore("{ System.out.println(\"start\"); }");
            m.insertAfter("{ System.out.println(\"end\"); }");
            return cc.toBytecode();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

现在有了Transformer，那么它要如何注入到正在运行的JVM呢？还需要定义一个Agent，借助Agent的能力将Instrument注入到JVM中，我们将在下一小节介绍Agent，现在要介绍的是Agent中用到的另一个类Instrumentation，在JDK 1.6之后，Instrumentation可以做启动后的Instrument、本地代码（Native Code）的Instrument，以及动态改变Classpath等等。我们可以向Instrumentation中添加上文中定义的Transformer，并指定要被重加载的类，代码如下所示这样，当Agent被Attach到一个JVM中时，就会执行类字节码替换并重载入JVM的操作

```java
import Java.lang.instrument.Instrumentation;

public Class TestAgent {
    public static void agentmain(String args, Instrumentation inst) {
        //指定我们自己定义的Transformer，在其中利用Javassist做字节码替换
        inst.addTransformer(new TestTransformer(), true);
        try {
            //重定义类并载入新的字节码
            inst.retransformClasses(Base.Class);
            System.out.println("Agent Load Done.");
        } catch (Exception e) {
            System.out.println("agent load failed!");
        }
    }
}
```

### JVMTI & Agent & Attach API

上一小节中，我们给出了Agent类的代码，追根溯源需要先介绍JPDA（Java Platform Debugger Architecture）如果JVM启动时开启了JPDA，那么类是允许被重新加载的，在这种情况下，已被加载的旧版本类信息可以被卸载，然后重新加载新版本的类。正如JDPA名称中的Debugger，JDPA其实是一套用于调试Java程序的标准，任何JDK都必须实现该标准

JPDA定义了一整套完整的体系，它将调试体系分为三部分，并规定了三者之间的通信接口，三部分由低到高分别是Java 虚拟机工具接口（JVMTI），Java 调试协议（JDWP）以及 Java 调试接口（JDI），三者之间的关系如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123211335-980002630.png)

现在回到正题，我们可以借助JVMTI的一部分能力，帮助动态重载类信息JVM TI（JVM TOOL INTERFACE，JVM工具接口）是JVM提供的一套对JVM进行操作的工具接口通过JVMTI，可以实现对JVM的多种操作，它通过接口注册各种事件勾子，在JVM事件触发时，同时触发预定义的勾子，以实现对各个JVM事件的响应，事件包括类文件加载、异常产生与捕获、线程启动和结束、进入和退出临界区、成员变量修改、GC开始和结束、方法调用进入和退出、临界区竞争与等待、VM启动与退出等等

而Agent就是JVMTI的一种实现，Agent有两种启动方式，一是随Java进程启动而启动，经常见到的Java -agentlib就是这种方式；二是运行时载入，通过attach API，将模块（jar包）动态地Attach到指定进程id的Java进程内

Attach API 的作用是提供JVM进程间通信的能力，比如说我们为了让另外一个JVM进程把线上服务的线程Dump出来，会运行jstack或jmap的进程，并传递pid的参数，告诉它要对哪个进程进行线程Dump，这就是Attach API做的事情在下面，我们将通过Attach API的loadAgent()方法，将打包好的Agent jar包动态Attach到目标JVM上具体实现起来的步骤如下：

- 定义Agent，并在其中实现AgentMain方法，如上一小节中定义的代码块7中的TestAgent类；
- 然后将TestAgent类打成一个包含MANIFEST.MF的jar包，其中MANIFEST.MF文件中将Agent-Class属性指定为TestAgent的全限定名，如下图所示；

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123224970-1235488534.png)

- 最后利用Attach API，将我们打包好的jar包Attach到指定的JVM pid上，代码如下：

```java
import com.sun.tools.attach.VirtualMachine;

public Class Attacher {
    public static void main(String[] args) throws AttachNotSupportedException, IOException, AgentLoadException, AgentInitializationException {
        // 传入目标 JVM pid
        VirtualMachine vm = VirtualMachine.attach("39333");
        vm.loadAgent("/Users/zen/operation_server_jar/operation-server.jar");
    }
}
```

- 由于在MANIFEST.MF中指定了Agent-Class，所以在Attach后，目标JVM在运行时会走到TestAgent类中定义的agentmain()方法，而在这个方法中，我们利用Instrumentation，将指定类的字节码通过定义的类转化器TestTransformer做了Base类的字节码替换（通过Javassist），并完成了类的重新加载由此，我们达成了“在JVM运行时，改变类的字节码并重新载入类信息”的目的

以下为运行时重新载入类的效果：先运行Base中的main()方法，启动一个JVM，可以在控制台看到每隔五秒输出一次”process”接着执行Attacher中的main()方法，并将上一个JVM的pid传入此时回到上一个main()方法的控制台，可以看到现在每隔五秒输出”process”前后会分别输出”start”和”end”，也就是说完成了运行时的字节码增强，并重新载入了这个类

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123237414-2118602051.png)

### 使用场景

至此，字节码增强技术的可使用范围就不再局限于JVM加载类前了。通过上述几个类库，我们可以在运行时对JVM中的类进行修改并重载了。通过这种手段，可以做的事情就变得很多了：

- 热部署：不部署服务而对线上服务做修改，可以做打点、增加日志等操作
- Mock：测试时候对某些服务做Mock
- 性能诊断工具：比如bTrace就是利用Instrument，实现无侵入地跟踪一个正在运行的JVM，监控到类和方法级别的状态信息

## 总结

字节码增强技术相当于是一把打开运行时JVM的钥匙，利用它可以动态地对运行中的程序做修改，也可以跟踪JVM运行中程序的状态。此外，我们平时使用的动态代理、AOP也与字节码增强密切相关，它们实质上还是利用各种手段生成符合规范的字节码文件。综上所述，掌握字节码增强后可以高效地定位并快速修复一些棘手的问题（如线上性能问题、方法出现不可控的出入参需要紧急加日志等问题），也可以在开发中减少冗余代码，大大提高开发效率















# 类的生命周期

整个流程如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109171703028-1277212645.png" alt="image-20231109171701801" />

注意：`加载`、`验证`、`准备`和`初始化`这四个阶段发生的顺序是确定的，**而`解析`阶段则不一定，它在某些情况下可以在初始化阶段之后开始，这是为了支持Java语言的运行时绑定(也称为动态绑定或晚期绑定)**。另外：这里的几个阶段是按顺序开始，而不是按顺序进行或完成，因为这些阶段通常都是互相交叉地混合进行的，通常在一个阶段执行的过程中调用或激活另一个阶段





## 加载：查找并加载类的二进制数据

加载是类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下三件事情:

1. 类加载器根据类的全限定名通过不同的渠道以二进制流的方式获取字节码信息。如下列的不同渠道：

- 从本地系统中直接加载
- 通过网络下载.Class文件
- 从zip，jar等归档文件中加载.Class文件
- 从专有数据库中提取.Class文件
- 将Java源文件动态编译为.Class文件

2. 将这个字节流所代表的静态存储结构转化为**方法区**的运行时数据结构，**生成一个InstanceKlass对象**，保存类的所有信息，里边还包含实现特定功能比如多态的信息。

![image-20231107180858869](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107180900267-132766341.png)

3. Java虚拟机会在**堆中生成**一份与方法区中数据“类似”的**java.lang.Class对象**，作为对方法区中相关数据的访问入口。

作用：在Java代码中去获取类的信息以及存储静态字段的数据（这里的静态字段是说的JDK8及之后的Java虚拟机的设计）

PS：堆中生成的java.lang.Class对象信息是方法区中生成的InstanceKlass对象信息的浓缩版，也就是将方法区InstanceKlass中程序员不需要的信息剔除就成为堆中的java.lang.Class对象信息

对于开发者来说，只需要访问堆中的Class对象而不需要访问方法区中所有信息。这样Java虚拟机就能很好地控制开发者访问数据的范围

![image-20231107181729831](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107181731071-1942446648.png)

> **注意：**类加载器并不需要等到某个类被“首次主动使用”时再加载它，JVM规范允许类加载器在预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了 .Class文件 缺失或存在错误，类加载器必须在程序首次主动使用该类时才报告错误(LinkageError错误)，如果这个类一直没有被程序主动使用，那么类加载器就不会报告错误



既然要加载类，那是怎么加载的就得了解了，而这就不得不了解“JVM类加载机制”了





### JVM类加载机制

#### 类加载器的分类

> 类加载器（ClassLoader）是Java虚拟机提供给应用程序去实现获取类和接口字节码数据的技术。
>
> 类加载器只参与加载过程中的字节码获取并加载到内存这一部分。

**类加载器分为两类，一类是Java代码中实现的，一类是Java虚拟机底层源码实现的**。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107201852011-1580861569.png" alt="image-20231107201850757" />

 

类加载器的设计JDK8和8之后的版本差别较大，JDK8及之前的版本中默认的类加载器有如下几种：

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107201957359-1777713058.png" alt="image-20231107201956280" />

##### 查找类加载器

1. **Arthas中类加载器的详细信息的查看方式**。[Arthas之classloader命令官网说明](https://arthas.aliyun.com/doc/classloader.html)

```bash
classloader		# 查看 classloader 的继承树，urls，类加载信息，使用 classloader 去获取资源
```

参数说明：

|              参数名称 | 参数说明                                   |
| --------------------: | :----------------------------------------- |
|                   [l] | 按类加载实例进行统计                       |
|                   [t] | 打印所有 ClassLoader 的继承树              |
|                   [a] | 列出所有 ClassLoader 加载的类，请谨慎使用  |
|                `[c:]` | ClassLoader 的 hashcode                    |
| `[classLoaderClass:]` | 指定执行表达式的 ClassLoader 的 class name |
|             `[c: r:]` | 用 ClassLoader 去查找 resource             |
|          `[c: load:]` | 用 ClassLoader 去加载指定的类              |

示例：

PS：下图那些数量指的是我进入的当前这个线程相关的，不是说所有的数量就是那些，根据进程会发生改变

![image-20231107203154937](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107203156691-1726406511.png)

另外：Arthas也可以查看 JVM 已加载的”某个类“的类信息，[Arthas之sc命令官网说明](https://arthas.aliyun.com/doc/sc.html)

sc命令，即“Search-Class” 的简写，这个命令能搜索出所有已经加载到 JVM 中的 Class 信息，这个命令支持的参数有 `[d]`、`[E]`、`[f]` 和 `[x:]`，参数说明如下：

|              参数名称 | 参数说明                                                     |
| --------------------: | :----------------------------------------------------------- |
|       *class-pattern* | 类名表达式匹配                                               |
|      *method-pattern* | 方法名表达式匹配                                             |
|                   [d] | 输出当前类的详细信息，包括这个类所加载的原始文件来源、类的声明、加载的 ClassLoader 等详细信息。 如果一个类被多个 ClassLoader 所加载，则会出现多次 |
|                   [E] | 开启正则表达式匹配，默认为通配符匹配                         |
|                   [f] | 输出当前类的成员变量信息（需要配合参数-d 一起使用）          |
|                  [x:] | 指定输出静态变量时属性的遍历深度，默认为 0，即直接使用 `toString` 输出 |
|                `[c:]` | 指定 class 的 ClassLoader 的 hashcode                        |
| `[classLoaderClass:]` | 指定执行表达式的 ClassLoader 的 class name                   |
|                `[n:]` | 具有详细信息的匹配类的最大数量（默认为 100）                 |
|          `[cs <arg>]` | 指定 class 的 ClassLoader#toString() 返回值。长格式`[classLoaderStr <arg>]` |

示例：

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107210801646-778180858.png" alt="image-20231107210759606" />



2. **自己写代码查看**

```java
package com.zixieqing;

import java.io.IOException;

/**
 * <p>
 * 查找类加载器
 * </p>
 *
 * <p>@author     : ZiXieqing</p>
 */

public class FindClassLoader {
    public static void main(String[] args) throws IOException {
        // 获取当前类的类加载器   即应用程序类加载器
        ClassLoader classLoader = FindClassLoader.class.getClassLoader();
        System.out.println("classLoader = " + classLoader);
        // 获取父类加载器  即扩展类加载器
        ClassLoader parentclassLoader = classLoader.getParent();
        System.out.println("parentclassLoader = " + parentclassLoader);
        // 获取启动类加载器     C语言实现，所以结果为 null	目的：出于安全考虑，所以不许操作此类加载器
        ClassLoader loader = parentclassLoader.getParent();
        System.out.println("loader = " + loader);

        System.in.read();
    }
}
```

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107204235089-2007068259.png" alt="image-20231107204233782" />

还有几种获取类加载器的方式：

```java
// 获取当前 ClassLoader
clazz.getClassLoader();

// 获取当前线程上下文的 ClassLoader
Thread.currentThread().getContextClassLoader();

// 获取系统的 ClassLoader
ClassLoader.getSystemClassLoader();

// 获取调用者的 ClassLoader
DriverManager.getCallerClassLoader();
```



通过这个代码查看之后，也可以说三个类加载器的层次关系长下面这个鸟样儿：

> **注：**这几个类加载器并不是继承关系，而是组合，或者可说是层级/上下级关系，只是源码中用的是ClassLoader类型的Parent字段来实现“双亲委派机制”（后续会上源码）。



![image-20240129150541913](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240129150523738-1741029320.png)





##### 启动类加载器

启动类加载器（Bootstrap ClassLoader）是由Hotspot虚拟机提供的、使用C++编写的类加载器。

启动类加载器是无法被Java程序直接引用的。

**默认加载Java安装目录/jre/lib**下的类文件，比如rt.jar（Java的核心类库，如java.lang.String），tools.jar，resources.jar等。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107204705687-1146242012.png" alt="image-20231107204704719" />





###### 启动类加载器加载用户jar包的方式

1. 放入  **JDK安装目录/jre/lib** 下进行扩展。 ==不推荐==

尽可能不要去更改JDK安装目录中的内容，会出现即时放进去由于文件名不匹配的问题也不会正常地被加载

2. 使用虚拟机参数 **-Xbootclasspath/a:jar包目录/jar包名** 命令扩展。==推荐==

PS：此命令中有一个 “a” 即add的意思，意为添加一个“jar包目录/jar包名”的jar包

IDEA中示例：

```bash
-Xbootclasspath/a:E:/Study/JVM-Demo/jar/FindClassLoader.jar
```

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107212413880-990849318.png" alt="image-20231107212412442" />



##### 扩展 与 应用程序类加载器

扩展类加载器和应用程序类加载器都是JDK中提供的、使用Java编写的类加载器。

它们的源码都位于sun.misc.Launcher中，是一个静态内部类。继承自URLClassLoader。具备通过目录或者指定jar包将字节码文件加载到内存中。

**扩展类加载器**：加载 **Java安装目录/jre/lib/ext** 下的类文件

**应用程序类加载器**：加载 **classpath** 下的类文件，此classpath包括“自己项目中编写的类或接口中的文件 和 第三方jar包中的类或接口中的文件（如：maven依赖中的）”

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107213417227-2043964033.png" alt="image-20231107213415977" />

Arthas验证上述目录：

```bash
classloader -l						# 查看所有类加载器的hash码

classloader –c 类加载器的hash码		# 查看当前类加载器的加载路径
```

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107215447835-2033678429.png" alt="image-20231107215445557" />





###### 扩展类加载器加载用户jar包的方式

1. 放入 **JDK安装目录/jre/lib/ext** 下进行扩展。==不推荐==

尽可能不要去更改JDK安装目录中的内容

2. 使用虚拟机参数 **-Djava.ext.dirs=jar包目录** 进行扩展。==推荐==

注意：此种方式会覆盖掉扩展类加载器原始加载的目录。Windows可以使用`;`分号，macos/Linux可以使用`:`冒号隔开，从而将原始目录添加在后面

```bash
# Windows中示例
-Djava.ext.dirs=E:/Study/JVM-Demo/jar/ext;D:/Install/JDK/JDK8/jre/lib/ext
```







#### 双亲委派机制

若是前面查看了应用程序类加载器的加载路径的话，会发现一个有意思的地方：

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107224850080-1625458848.png" alt="image-20231107224848371" />

查看应用程序类加载器的加载路径，发现启动类加载器的加载路径也有，为什么？

要搞清楚这个问题，就需要知道“双亲委派机制”了。

>  **双亲委派机制：**
>
> ![image-20231108204546605](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231108210325720-978226333.png)
>
> 1. 当一个类加载器加载某个类的时候，会**自底向上查找是否加载过**；若加载过直接返回，若一直到最顶层的类加载器都没有加载，再**自顶向下进行加载**。
> 2. **应用程序类加载器的父类加载器是扩展类加载器，扩展类加载器的父类加载器是启动类加载器**。
>
> - PS：严谨点来说，**扩展类加载器没有父类加载器，只是会“委派”给启动类加载器**，即如果类加载的parent为null，则会提交给启动类加载器处理。
>
> 3. 双亲委派机制的好处：一是避免恶意代码替换JDK中的核心类库（如：java.lang.String），确保核心类库得到完整性和安全性；二是避免一个类重复被加载



**双亲委派机制的作用：**

1. **避免类被重复加载**：若一个类重复出现在三个类加载器的加载位置，应该由谁加载？

启动类加载器加载，根据双亲委派机制，它的优先级是最高的。

双亲委派机制可以避免同一个类被多次加载，上层的类加载器如果加载过该类，就会直接返回该类，避免重复加载。

2. **保证类加载的安全性**：如在自己的项目中创建一个java.lang.String类，会被加载吗？

不能，会交由启动类加载器加载在rt.jar包中的String类。

通过双亲委派机制，让顶层的类加载器去加载核心类，**避免恶意代码替换JDK中的核心类库**（这个也叫**沙箱安全机制**），如：上述的java.lang.String，确保核心类库的完整性和安全性。

同时底层源码中建包“禁止”以`java.`开头

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231110195924054-1517961328.png" alt="image-20231110195922678" />



上面定义中第一点说“类加载器加载某个类的时候”，那在Java中怎么加载一个类？这就需要知道Java中类加载的方式了。





##### Java中类加载的方式

在Java中如何使用代码的方式去主动加载一个类？Java中类加载有两种方式：

1. 通过 `Class.forName()` 方法动态加载

2. 通过 `ClassLoader.loadClass()` 方法动态加载

```java
public Class loaderTest { 
        public static void main(String[] args) throws ClassNotFoundException { 
                ClassLoader loader = HelloWorld.Class.getClassLoader(); 
                System.out.println(loader); 
                // 使用ClassLoader.loadClass()来加载类，不会执行初始化块 
                loader.loadClass("Test2"); 
                // 使用Class.forName()来加载类，默认会执行初始化块 
			    // Class.forName("Test2"); 
                // 使用Class.forName()来加载类，并指定ClassLoader，初始化时不执行静态块 
			    // Class.forName("Test2", false, loader); 
        } 
}

public Class Test2 { 
        static { 
                System.out.println("静态初始化块执行了！"); 
        } 
}
```

分别切换加载方式，会有不同的输出结果

> Class.forName() 和 ClassLoader.loadClass()区别?

- Class.forName():：将类的.Class文件加载到JVM中之外，还会对类进行解释，执行类中的static块；
- ClassLoader.loadClass():：只干一件事情，就是将.Class文件加载到JVM中，不会执行static中的内容,只有在newInstance才会去执行static块
- Class.forName(name, initialize, loader)带参函数也可控制是否加载static块并且只有调用了newInstance()方法采用调用构造函数，创建类的对象 







##### Java中查看双亲委派机制的实现

```java
package java.lang;


public abstract class ClassLoader {
    
    // 每个Java实现的类加载器中保存了一个成员变量叫“父”（Parent）类加载器，可以理解为它的上级，并不是继承关系
    private final ClassLoader parent;
    
    
	public Class<?> loadClass(String name) throws ClassNotFoundException {
        // 第二个参数 会决定着是否要开始类的生命周期的解析阶段，实质执行的是"类生命周期中的连接阶段"
        return loadClass(name, false);
    }
    
	protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // 首先判断该类型是否已经被加载
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    // 如果没有被加载，就委托给父类加载或者委派给启动类加载器加载
                    if (parent != null) {
                        // 如果存在父类加载器，就委派给父类加载器加载
                        c = parent.loadClass(name, false);
                    } else {
                        // 如果不存在父类加载器，就检查是否是由启动类加载器加载的类，
                        // 通过调用本地方法native Class findBootstrapClass(String name)
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                // 如果父类加载器和启动类加载器都不能完成加载任务
                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    // 调用自身的加载功能
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            // 是否开始解析，此处resolve = false
            if (resolve) {
                // 最终调用 private native void resolveClass0(Class<?> c);
                resolveClass(c);
            }
            return c;
        }
    }
}
```







##### 自定义类加载器

通常情况下，我们都是直接使用系统类加载器，但是，有的时候，我们也需要自定义类加载器，比如应用是通过网络来传输 Java 类的字节码，为保证安全性，这些字节码经过了加密处理，这时系统类加载器就无法对其进行加载，此时就需要我们自定义类加载器。

而需要自定义类加载器，就需要了解几个API：

```java
package java.lang;


public abstract class ClassLoader {

    /**
     * 类加载的入口，提供双亲委派机制，调用了 findClass(String name)
     */
    protected Class<?> loadClass(String name, boolean resolve) {
        // .............
    }
    
    /**
     * 由类加载器子类实现获取二进制数据，
     * 调用 defineClass(String name, byte[] b, int off, int len) 
     * 如：URLClassLoader 会根据文件路径获取类文件中的二进制数据
     */
	protected Class<?> findClass(String name) {
        throw new ClassNotFoundException(name);
    }
}


/**
 * 做一些类名的校验，然后调用虚拟机底层的方法将字节码信息加载到虚拟机内存中
 */
protected final Class<?> defineClass(String name, byte[] b, int off, int len){
    // .........
}


/**
 * 执行类生命周期中的连接阶段
 */
protected final void resolveClass(Class<?> c){
    // .........
}
```

从上对 loadClass 方法来分析来看：想要自定义类加载器，那么继承 ClassLoader 类，重写 `findClass()` 即可，示例如下：

> JDK1.2之前是重写loadClass方法，但JDK1.2之后就建议自定义类加载器最好重写 `findClass()` 而不要重写 `loadClass()`，因为这样容易破坏双亲委托模式。



```java
package com.zixieqing;

import java.io.*;

/**
 * <p>
 * 自定义类加载器：核心在于对字节码文件的获取
 * </p>
 *
 * <p>@author     : ZiXieqing</p>
 */

public class MyClassLoader extends ClassLoader {

    private String root;

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        // 获取二进制数据
        byte[] ClassData = loadClassData(name);

        if (ClassData == null) {
            throw new ClassNotFoundException();
        } else {
            // 校验，将字节码信息加载进虚拟机内存
            return defineClass(name, ClassData, 0, ClassData.length);
        }
    }

    /**
     * 加载二进制数据
     * @param className Class全限定名
     */
    private byte[] loadClassData(String className) {

        // 得到Class字节码文件名
        String fileName = root + File.separatorChar + className.replace('.', File.separatorChar) + ".Class";

        try {
            InputStream ins = new FileInputStream(fileName);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 1024;
            byte[] buffer = new byte[bufferSize];
            int length = 0;
            while ((length = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, length);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    public void setRoot(String root) {
        this.root = root;
    }

    public static void main(String[] args) throws ClassNotFoundException {

        MyClassLoader myClassLoader = new MyClassLoader();
        // 设置此类加载器的加载目录
        myClassLoader.setRoot("E:\\lib\\");

        // 要加载哪个类
        // 传递的文件名是类的全限定名，以 "." 隔开，因为 defineClass() 是按这种格式进行处理的
        Class<?> testClass = myClassLoader.loadClass("com.zixq.FindClassLoader");
        System.out.println(testClass.getClassLoader());
        System.out.println(testClass.getClassLoader().getParent());
    }
}
```

结果如下：

```java
om.zixieqing.MyClassLoader@1b6d3586
sun.misc.Launcher$AppClassLoader@18b4aac2
```

1. 问题：**自定义类加载器父类怎么是AppClassLoader**？

以Jdk8为例，ClassLoader类中提供了构造方法设置parent的内容：

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109145557267-2110824427.png" alt="image-20231109145558395" />

这个构造方法由另外一个构造方法调用，其中父类加载器由 `getSystemClassLoader()` 设置，该方法返回的是AppClassLoader。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109145800286-1067637581.png" alt="image-20231109145801435" style="zoom: 50%;" />



2. 问题：**两个自定义类加载器加载相同限定名的类，会不会冲突？**

不会冲突，在同一个Java虚拟机中，只有“相同类加载器+相同的类限定名”才会被认为是同一个类。

```bash
# 在Arthas中使用下列方式查看具体的情况
sc –d 类的全限定名
```







#### Java9之后的类加载器

JDK8及之前的版本中，扩展类加载器和应用程序类加载器的源码位于rt.jar包中的s`un.misc.Launcher.java`。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109202531696-2116661804.png" alt="image-20231109202530496" style="zoom: 67%;" />

JDK9引入了module的概念，类加载器在设计上发生了变化：

1. 启动类加载器使用Java编写，位于`jdk.internal.loader.ClassLoaders`类中。

Java中的BootClassLoader继承自BuiltinClassLoader，实现从模块中找到要加载的字节码资源文件。

启动类加载器依然无法通过Java代码获取到，返回的仍然是null，保持了统一。

2. 扩展类加载器被替换成了平台类加载器（Platform Class Loader）。

平台类加载器遵循模块化方式加载字节码文件，所以继承关系从URLClassLoader变成了BuiltinClassLoader，BuiltinClassLoader实现了从模块中加载字节码文件。平台类加载器的存在更多的是为了与老版本的设计方案兼容，自身没有特殊的逻辑。





## 连接

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109171703028-1277212645.png" alt="image-20231109171701801" />



### 验证：验证被加载的类是否满足Java虚拟机规范

> 验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息遵守《Java虚拟机规范》中的约束，并且不会危害虚拟机自身的安全。

==这个阶段一般不需要程序员参与==。验证阶段大致会完成4个阶段的检验动作:

- `文件格式验证`：验证字节流是否符合Class文件格式的规范；例如: 是否以`0xCAFEBABE`开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型
- `元数据验证`：对字节码描述的信息进行语义分析(注意: 对比`Javac`编译阶段的语义分析)，以保证其描述的信息符合Java语言规范的要求；例如: 这个类是否有父类，除了`Java.lang.Object`之外
- `字节码验证`：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的
- `符号引用验证`：确保解析动作能正确执行

> 验证阶段是非常重要的，但不是必须的，它对程序运行期没有影响，
>
> **如果所引用的类经过反复验证，那么可以考虑采用`-Xverifynone`参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间**





### 准备：为静态变量分配内存 并 为其设置默认值

> 准备阶段是为静态变量（static）分配内存并设置默认值。**这些内存都将在方法区中分配**。

对于该阶段有以下几点需要注意：

1. 这时候进行内存分配的==仅包括静态变量(`static`)==，而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在Java堆中。
2. 这里所设置的默认值通常情况下是数据类型默认的零值，而不是在Java代码中被显式地赋予的值。

| 数据类型     | 默认值   |
| ------------ | -------- |
| byte         | 0        |
| short        | 0        |
| int          | 0        |
| long         | 0L       |
| boolean      | false    |
| double       | 0.0      |
| char         | ‘\u0000’ |
| 引用数据类型 | null     |

- 对基本数据类型来说，静态变量(static)和全局变量，如果不显式地对其赋值而直接使用，则系统会为其赋予默认的零值；而对于局部变量来说，在使用前必须显式地为其赋值，否则编译时不通过。
- 只被`final`修饰的常量则既可以在声明时显式地为其赋值，也可以在类初始化时显式地为其赋值，总之，在使用前必须为其显式地赋值，系统不会为其赋予默认零值。
- 同时被`static`和`final`修饰的基本数据类型的常量，准备阶段直接会将代码中的值进行赋值（即必须在声明的时候就为其显式地赋值，否则编译时不通过）。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109181450986-1215107925.png" alt="image-20231109181449857" style="zoom: 50%;" />









### 解析：将常量池中的符号引用 替换为 指向内存的直接引用

> 解析阶段主要是将常量池中的符号引用替换为直接引用。
>
> 解析动作主要针对`类`或`接口`、`字段`、`类方法`、`接口方法`、`方法类型`、`方法句柄`和`调用点`限定符7类符号引。

符号引用就是在字节码文件中使用编号来访问常量池中的内容。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109183305002-712655662.png" alt="image-20231109183303764" />

直接引用不再使用编号，而是使用内存地址来访问具体的数据。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109183446304-200246862.png" alt="image-20231109183445217" />





## 初始化：为类的静态变量赋正确值，对类进行初始化

> 初始化：为类的静态变量赋予正确的初始值，JVM负责对类进行初始化（主要对类变量进行初始化）。

初始化阶段会执行字节码文件中 ==clinit== （若是有）部分的字节码指令。clinit就是class init，即类构造方法，此类构造方法不是我们说的构造方法，不是一回事。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109201249945-69018762.png" alt="image-20231109201248802" />

**`clinit`指令在特定情况下不会出现**：

> 注：
>
> 1. 子类的初始化clinit调用之前，会先调用父类的clinit初始化方法。
> 2. 虚拟机会保证一个类的clinit方法在多线程下被同步加锁，即一个类的clinit方法只会被加载一次。

1. 无静态代码块且无静态变量赋值语句。
2. 有静态变量的声明，但是没有赋值语句。
3. 静态变量的定义使用final关键字，这类变量会在准备阶段直接进行初始化。



**在Java中对类变量进行初始值设定有两种方式**：

- 声明类变量时指定初始值
- 使用静态代码块为类变量指定初始值



**JVM初始化步骤：**

1. 若这个类还没有被加载和连接，则程序先加载并连接该类。
2. 若该类的父类还没有被初始化，则先初始化其父类。
3. 若类中有初始化语句，则系统依次执行这些初始化语句。

关于第3点：原因是在链接阶段的准备中，已经将静态变量加载到内存中了，只是初始值是数据类型的默认值而已，而这里初始化就是重新赋正确值，而这顺序就按代码顺序赋值。

如下图，虚拟机指令是按顺序执行初始化语句

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109205006247-1956213042.png" alt="image-20231109205004723" style="zoom: 67%;" />



**类初始化时机**：只有当对类的主动使用时会导致类的初始化。类的主动使用包括以下六种：

> 添加 `-XX:+TraceClassLoading` 虚拟机参数可以打印出加载并初始化的类。

1. 创建类的对象，也就是new的方式。
2. 访问某个类或接口的静态变量，或者对该静态变量赋值。注：变量是`final`修饰且等号右边是常量不会触发初始化。

```java
/**
 * 变量是 final 修饰且等号右边是常量不会触发初始化
 */

public class Initialization {
    public static final int value = 8;

    static {
        System.out.println("Initialization类被初始化了");
    }
}

class Test {
    public static void main(String[] args) {
        System.out.println(Initialization.value);
    }
}
```

3. 调用类的静态方法。
4. 反射。如：`Class.forName("com.zixieqing.JVM.Test"))` 。若使用的是下面这个API，则是否初始化取决于程序员。

```java
public static Class<?> forName(String name, boolean initialize, ClassLoader loader) {
    
}
```

5. 初始化某个类的子类，则其父类也会被初始化。
6. Java虚拟机启动时被标明为启动类的类，直接使用`Java.exe`命令来运行某个主类。





## 使用

类访问方法区内的数据结构的接口， 对象是Heap（堆）区的数据。



## 卸载

> 这个玩意儿在垃圾回收还会整。



**Java虚拟机将结束生命周期的几种情况：**

- 调用 Runtime 类 或 system 类的 `exit()`，或 Runtime 类的 `halt()`，并且 Java 安全管理器也允许这次 exit 或 halt 操作。
- 程序正常执行结束。
- 程序在执行过程中遇到了异常或错误而异常终止。
- 由于操作系统出现错误而导致Java虚拟机进程终止。





# 阿里一面：请说一下对象的创建过程

![img](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240323161817475-2015712042.jpg)    

在实例化一个对象的时候，JVM 首先会去检查目标对象是否已经被加载并初始化了。

 

如果没有，JVM 需要立刻去加载目标类，然后调用目标类的构造器完成初始化。目标类的加载是通过类加载器来实现的，主要就是把一个类加载到内存里面。然后初始化的过程，主要是对目标类里面的静态变量、成员变量、静态代码块进行初始化。 

 

当目标类被初始化后，就可以从常量池里面找到对应的类元信息，并且目标对象的大小在类加载之后就已经确定了，所以这个时候就需要为新创建的对象，根据目标对象的大小在堆内存里面分配内存空间。 内存分配的方式一般有两种，一种指针碰撞，另一种是空闲列表，JVM 会根据 Java 堆内存是否规整来决定内存分配方式。 

 

接下来，JVM 会把目标对象里面的普通成员变量初始化为零值，比如 int 类型初始化为 0，对象类型初始化为 null，（类变量在类加载的准备阶段就已经初始化过了）。 这一步操作主要是保证对象里面的实例字段，不用初始化就可以直接使用，也就是程序能够获得这些字段对应数据类型的零值。 
然后，JVM 还需要对目标对象的对象头做一些设置，比如对象所属的类元信息、对象的 GC 分代年龄、 hashcode、锁标记等等。 

 

完成这些步骤以后，对于 JVM 来说，新对象的创建工作已经完成。但是基于 Java 语言来说，对象创建才算是开始。 

接下来要做的，就是执行目标对象内部生成的 init 方法，初始化成员变量的值、执行构造块、最后执行目标对象的构造方法，完成对象的创建。 
其中，init 方法是 Java 文件编译之后在字节码文件中生成的，它是一个实例构造器，这个构造器会把语句块、变量初始化、调用父类构造器等操作组织在一起。所以调用 init 方法能够完成一系列的初始化动作。 





# JVM内存结构

> 注：不要和Java内存模型混淆了。



## 运行时数据区

> **Java虚拟机在运行Java程序过程中管理的内存区域，称之为运行时数据区**。
>
> Java 虚拟机定义了若干种程序运行期间会使用到的运行时数据区，其中有一些会随着虚拟机启动而创建，随着虚拟机退出而销毁，另外一些则是与线程一 一对应的，这些与线程一 一对应的数据区域会随着线程开始和结束而创建和销毁。
>
> **线程不共享：**
>
> - 程序计数器
> - 本地方法栈
> - Java虚拟机栈
>
> **线程共享：**
>
> - 方法区
> - 堆



![image-20231110204142543](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231111214356868-657372761.png)





### 程序计数器

> 程序计数器（**Program Counter Register**）也叫**PC**寄存器，**用来存储指向下一条字节码指令的地址，即将要执行的指令代码由执行引擎读取下一条指令**。

1. ==**程序计数器是唯一 一个在 JVM 规范中没有规定任何 内存溢出（`OutOfMemoryError` ）情况的区域**==。

> **内存溢出：** 指的是程序在使用某一块内存区域时，存放的数据需要占用的内存大小超过了虚拟机能提供的内存上限。

- 因为每个线程只存储一个固定长度的内存地址，所以程序计数器是不会发生内存溢出的。 因此：**程序员无需对程序计数器做任何处理**。

2. **在 JVM 规范中，每个线程都有它自己的程序计数器，是线程私有的，生命周期与线程的生命周期一致**。
3. 程序计数器是一块很小的内存空间，几乎可以忽略不计，也是运行速度最快的存储区域。
4. 分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

如：分支、跳转

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112142204976-197590123.png" alt="image-20231112142205561" style="zoom: 50%;" />

5. **JVM 中的 PC 寄存器是对物理 PC 寄存器的一种抽象模拟**。



> 接下来结合图理解一下程序计数器。

在加载阶段，虚拟机将字节码文件中的指令读取到内存之后，会将原文件中的偏移量转换成内存地址。每一条字节码指令都会拥有一个内存地址。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112134532660-124307120.png" alt="image-20231112134532805" />

在代码执行过程中，程序计数器会记录下一行字节码指令的地址。执行完当前指令之后，虚拟机的执行引擎根据程序计数器执行下一行指令。

> **注：**如果当前线程正在执行的是 Java 方法，程序计数器记录的是 JVM 字节码指令地址，如果是执行 native 方法，则是未指定值（undefined）。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112134852332-1705218448.png" alt="image-20231112134852723" />



1. 问题：**使用程序计数器存储字节码指令地址有什么用？为什么使用程序计数器记录当前线程的执行地址？**

因为CPU需要不停的切换各个线程，当切换回来以后，就得知道接着从哪开始继续执行，JVM的字节码解释器就需要通过改变程序计数器的值来明确下一条应该执行什么样的字节码指令。

2. 问题：**程序计数器为什么会被设定为线程私有的？**

多线程在一个特定的时间段内只会执行其中某一个线程方法，CPU会不停的做任务切换，这样必然会导致经常中断或恢复。为了能够准确的记录各个线程正在执行的当前字节码指令地址，所以为每个线程都分配了一个程序计数器，每个线程都独立计算，不会互相影响。





### 栈内存：Java虚拟机栈 与 本地方法栈

> 栈即先进后出（First In Last Out），是一种快速有效的分配存储方式，访问速度仅次于程序计数器.
>
> ==栈不存在垃圾回收问题==。



#### Java虚拟机栈（JVM Stack）

> **Java**虚拟机栈（Java Virtual Machine Stack），早期也叫 Java 栈。采用**栈**的数据结构来管理方法调用中的基本数据。
>
> 每个线程在创建的时候都会创建一个虚拟机栈，其内部保存一个个的**栈帧(Stack Frame）**，每一个Java方法的调用都使用一个栈帧来保存。

1. Java虚拟机栈主管 Java 程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回。
2. ==Java虚拟机栈是线程私有的，生命周期和线程一致==。
3. JVM 直接对虚拟机栈的操作只有两个：方法的入栈(方法的执行) 与 出栈(方法的结束)。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112165138710-1571653769.png" alt="image-20231112165137448" />

4. Java虚拟机栈如果栈帧过多，占用内存超过栈内存可以分配的最大大小就**会出现内存溢出**（内存溢出会出现`StackOverflowError`错误）。

```java
package com.zixieqing.runtime_data_area.stack;

/**
 * <p>
 * JVM内存结构：运行时数据区之Java虚拟机栈
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

public class JVM_Stack {

    /**
     * 计数器：得到当前系统的栈帧大小
     */
    private static int count = 0;

    public static void main(String[] args) {
        stackOverFlowTest();
    }

    /**
     * <p>
     * 1、测试Java虚拟机栈是否会内存溢出
     * </p>
     */
    public static void stackOverFlowTest() {
        System.out.println(++count);
        stackOverFlowTest();
    }
}



// 结果
10710
Exception in thread "main" java.lang.StackOverflowError
```

- **Java虚拟机栈默认大小**：如果我们不指定栈的大小，JVM 将创建一个具有默认大小的栈。大小取决于操作系统和计算机的体系结构。

```txt
Linux
		x86（64位）：1MB
		
Windows
		基于操作系统默认值
		
BSD
		x86（64位）：1MB
		
Solarls
		64位：1MB
```

- **自己设置栈大小**：使用虚拟机参数 `-Xss栈大小`

> **注：** HotSpot Java虚拟机对栈大小的最大值和最小值有要求
>
> - **Windows（64位）下的JDK8最小值为180k，最大值为1024m** 。
>
> 一般情况下，工作中栈的深度最多也只能到几百,不会出现栈的溢出。所以此参数可以手动指定为`-Xss256k`节省内存。

```txt
参数含义：设置线程的最大栈空间

语法：-Xss栈大小

单位：字节（默认，必须是 1024 的倍数）、k或者K(KB)、m或者M(MB)、g或者G(GB)。示例如下：
	-Xss1048576 
    -Xss1024K 
    -Xss1m
    -Xss1g



与 -Xss 类似，也可以使用 -XX:ThreadStackSize 来配置栈大小。

	格式为： -XX:ThreadStackSize=1024
```



由前面内容知道：每个线程在创建的时候都会创建一个虚拟机栈，其内部保存一个个的**栈帧(Stack Frame）**，这些栈帧对应着一个个方法的调用，那栈帧组成又是怎么样的？





##### 栈帧的组成

每个**栈帧**（Stack Frame）中存储着：

1. **局部变量表（Local Variables）**：局部变量表的作用是在运行过程中存放所有的局部变量。
2. **操作数栈（Operand Stack）**：操作数栈是栈帧中虚拟机在执行指令过程中用来存放临时数据的一块区域。
3. **帧数据（Frame Data）**：帧数据主要包含动态链接、方法出口、异常表的引用。

- **动态链接（Dynamic Linking）**：当前类的字节码指令引用了其他类的属性或者方法时，需要将符号引用（编号）转换成对应的运行时常量池中的内存地址。**动态链接就保存了编号（符号引用）到 运行时常量池的内存地址 的映射关系**。
- **方法返回地址 / 方法出口（Return Address）**：方法在正确或者异常结束时，当前栈帧会被弹出，同时程序计数器应该指向上一个栈帧中的下一条指令的地址。所以在当前栈帧中，需要存储此方法出口的地址。
- 一些附加信息：栈帧中还允许携带与 Java 虚拟机实现相关的一些附加信息，例如，对程序调试提供支持的信息，但这些信息取决于具体的虚拟机实现，所以在这里不说明这玩意儿。



###### 局部变量表（Local Variables）

> 局部变量表也被称为局部变量数组或者本地变量表。
>
> **局部变量表的作用是在方法执行过程中存放所有的局部变量**。编译成字节码文件时就可以确定局部变量表的内容。
>
> ==**局部变量表中保存的是：实例方法的this对象，方法的参数，方法体中声明的局部变量**==

1. **栈帧中的局部变量表是一个数组，数组中每一个位置称之为槽(slot)，long和double类型占用两个槽，其他类型占用一个槽。** 

> byte、short、char 在存储前被转换为int；
>
> boolean也被转换为int，0 表示 false，1 表示 true。



<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112205004543-2094783191.png" alt="image-20231112205003404" />

栈帧中的局部变量表是咋样的？数组咯，每个索引位置就是一个槽（Slot）

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112205502809-1418460258.png" alt="image-20231112205501841" style="zoom: 80%;" />

插入一个与此相关的内容：Java中的8大数据类型在虚拟机中的实现。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240124212930184-1737058423.png" alt="image-20240124212945092" />

- **问题：boolean、byte、char、short在栈上是不是存在空间浪费？**

是的，Java虚拟机采用的是空间换时间方案，在栈上不存储具体的类型，只根据slot槽进行数据的处理，浪费了一些内存空间，但是避免不同数据类型不同处理方式带来的时间开销。

同时，像long型在64位系统中占用2个slot，使用了16字节空间，但实际上在Hotspot虚拟机中，它的高8个字节没有使用，这样就满足了long型使用8个字节的需要。

2. **如果当前帧是由构造方法或实例方法创建的，那么该对象引用 this 将会存放在 index 为 0 的 Slot 处，其余的参数按照参数表顺序继续排列**。

因为实例方法需要先拿到实例对象，即代码首行有个隐藏的this，它去搞对象了。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112210402806-1619948978.png" alt="image-20231112210400680" />



3. 为了节省空间，**局部变量表中的槽（Slot）是可以被复用的**。一旦某个局部变量不再生效，当前槽就可以再次被使用。

![image-20231112212102143](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112212104648-524392593.png)





###### 操作数栈（Operand Stack）

> **操作数栈：是栈帧中虚拟机在执行指令过程中用来存放中间数据的一块区域**。如果一条指令将一个值压入操作数栈，则后面的指令可以弹出并使用该值。

1. **操作数栈，在方法执行过程中，根据字节码指令，往操作数栈中写入数据或提取数据，即入栈（push）、出栈（pop）**。

2. **如果被调用的方法带有返回值的话，其返回值将会被压入当前栈帧的操作数栈中**，并更新 PC 寄存器中下一条需要执行的字节码指令。
3. 所谓的**Java虚拟机的解释引擎是基于栈的执行引擎**，其中的栈指的就是操作数栈。

4. **每一个操作数栈在编译期就已经确定好了其所需的最大深度**，从而在执行时正确的分配内存大小。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112220642671-2003094921.png" alt="image-20231112220641722" />

那这个所需的最大深度是怎么确定的？

编译器模拟对应的字节码指令执行过程，在这个过程中最多可以存放几个数据，这个最多存放多少个数据就是操作数栈所需的最大深度。

如下图，共出现了0和1这两个数据，所以操作数栈的最大深度就是2。

![操作数栈最大深度的确定 + 操作数栈的入栈、出栈](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112222052205-149655198.gif)

> 另一个小知识点：栈顶缓存（Top-of-stack-Cashing）

HotSpot 的执行引擎采用的并非是基于寄存器的架构，但这并不代表 HotSpot VM 的实现并没有间接利用到寄存器资源，寄存器是物理 CPU 中的组成部分之一，它同时也是 CPU 中非常重要的高速存储资源。一般来说，寄存器的读/写速度非常迅速，甚至可以比内存的读/写速度快上几十倍不止，不过寄存器资源却非常有限，不同平台下的CPU 寄存器数量是不同和不规律的。寄存器主要用于缓存本地机器指令、数值和下一条需要被执行的指令地址等数据

基于栈式架构的虚拟机所使用的零地址指令更加紧凑，但完成一项操作的时候必然需要使用更多的入栈和出栈指令，这同时也就意味着将需要更多的指令分派（instruction dispatch）次数和内存读/写次数，由于操作数是存储在内存中的，因此频繁的执行内存读/写操作必然会影响执行速度。为了解决这个问题，HotSpot JVM 设计者们提出了==**栈顶缓存技术，将栈顶元素全部缓存在物理 CPU 的寄存器中，以此降低对内存的读/写次数，提升执行引擎的执行效率**==





###### 帧数据（Frame Data）

1. **动态链接（Dynamic Linking）**：当前类的字节码指令引用了其他类的属性或者方法时，需要将符号引用（编号）转换成对应的运行时常量池中的内存地址。**动态链接就保存了编号（符号引用）到 运行时常量池的内存地址 的映射关系**。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231113144513254-1121270090.png" alt="image-20231113144512701" />

> JVM 是如何执行方法调用的？

方法调用不同于方法执行，方法调用阶段的唯一任务就是确定被调用方法的版本（即调用哪一个方法），暂时还不涉及方法内部的具体运行过程，Class 文件的编译过程中不包括传统编译器中的连接步骤，一切方法调用在 Class文件里面存储的都是**符号引用**，而不是方法在实际运行时内存布局中的入口地址（**直接引用**），也就是需要在类加载阶段，甚至到运行期才能确定目标方法的直接引用。

在 JVM 中，将符号引用转换为调用方法的直接引用与方法的绑定机制有关

- **静态链接**：当一个字节码文件被装载进 JVM 内部时，如果被调用的**目标方法在编译期可知，且运行期保持不变时**，这种情况下将调用方法的符号引用转换为直接引用的过程称之为静态链接。
- **动态链接**：如果**被调用的方法在编译期无法被确定下来**，也就是说，只能在程序运行期将调用方法的符号引用转换为直接引用，由于这种引用转换过程具备动态性，因此也就被称之为动态链接。

对应的方法的绑定机制为：早期绑定（Early Binding）和晚期绑定（Late Binding）。

**绑定是一个字段、方法或者类在符号引用被替换为直接引用的过程，这仅仅发生一次**。

- **早期绑定**：**指被调用的目标方法如果在编译期可知，且运行期保持不变时**，即可将这个方法与所属的类型进行绑定，这样一来，由于明确了被调用的目标方法究竟是哪一个，因此也就可以使用静态链接的方式将符号引用转换为直接引用。
- **晚期绑定**：如果**被调用的方法在编译器无法被确定下来**，只能够在程序运行期根据实际的类型绑定相关的方法，这种绑定方式就被称为晚期绑定。



> 虚方法与非虚方法

- **非虚方法：指的是方法在编译器就确定了具体的调用版本，这个版本在运行时是不可变的**，如静态方法、私有方法、final 方法、实例构造器、父类方法都是非虚方法。
- 其他方法称为虚方法。





2. **方法返回地址 / 方法出口（Return Address）：方法在正确或者异常结束时，当前栈帧会被弹出，同时程序计数器应该指向上一个栈帧中的下一条指令的地址。所以在当前栈帧中，存放的这条指令地址就是方法出口地址**。

> 本质上，**方法的结束就是当前栈帧出栈的过程**，此时，需要恢复上层方法的局部变量表、操作数栈、将返回值压入调用者栈帧的操作数栈、设置PC寄存器值等，让调用者方法继续执行下去

细节说明一下：

- **方法正常退出时**，调用该方法的指令的下一条指令的地址即为返回地址。

一个方法的正常调用完成之后，究竟需要使用哪一个返回指令，还需要根据方法返回值的实际数据类型而定。

| 返回类型                         | 返回指令 |
| -------------------------------- | -------- |
| void、类和接口的初始化方法       | return   |
| int (boolean、byte、char、short) | ireturn  |
| long                             | lreturn  |
| float                            | freturn  |
| double                           | dreturn  |
| reference                        | areturn  |

- **方法通过异常退出的**，返回地址是要通过异常表来确定的，栈帧中一般不会保存这部分信息。

在方法执行的过程中遇到了异常，并且这个异常没有在方法内进行处理，也就是只要在本方法的异常表中没有搜索到匹配的异常处理器，就会导致方法退出，简称**异常完成出口**。

方法执行过程中抛出异常时的异常处理，存储在一个异常处理表，方便在发生异常的时候找到处理异常的代码。



> 上述内容说到了“异常表”，也顺便解释一下这个东西

**异常表：存放的是代码中异常的处理信息，包含了异常捕获的生效范围以及异常发生后跳转到的字节码指令位置**。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231113161321529-1755139183.png" alt="image-20231113161321037" style="zoom: 50%;" />

程序运行中触发异常时，Java 虚拟机会从上至下遍历异常表中的所有条目。当触发异常的字节码的索引值在某个异常表条目的监控范围内，Java 虚拟机会判断所抛出的异常和该条目想要捕获的异常是否匹配。

- 如果匹配，跳转到“跳转PC”对应的字节码位置。
- 如果遍历完都不能匹配，说明异常无法在当前方法执行时被捕获，此方法栈帧直接弹出，在上一层的栈帧中进行异常捕获的查询。

多个catch分支情况下（multi-catch的写法也一样），异常表会从上往下遍历，先捕获RuntimeException，如果捕获不了，再捕获Exception。

**finally的处理方式**：

- finally中的字节码指令会插入到try 和 catch代码块中，保证在try和catch执行之后一定会执行finally中的代码。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126152724283-1230160433.png" alt="image-20240126152740285" />

- 如果抛出的异常范围超过了Exception，比如Error或者Throwable，此时也要执行finally，所以异常表中增加了两个条目。覆盖了try和catch两段字节码指令的范围，`any`代表可以捕获所有种类的异常。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126153105591-1787471596.png" alt="image-20240126153122051" />

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126153202554-1745074822.png" alt="image-20240126153219055" />





#### 本地方法栈（Native Method Stack）

> Java虚拟机栈存储了Java方法调用时的栈帧，而**本地方法栈存储的是native本地方法的栈帧**。
>
> 它的具体做法是 `Native Method Stack` 中登记 native 方法，在 `Execution Engine` 执行时加载本地方法库，当某个线程调用一个本地方法时，它就进入了一个全新的并且不再受虚拟机限制的世界，它和虚拟机拥有同样的权限。
>
> ==本地方法栈也是线程私有的。也会发生内存溢出==。
>
> - 如果线程请求分配的栈容量超过本地方法栈允许的最大容量，Java 虚拟机将会抛出一个 `StackOverflowError` 异常
> - 如果本地方法栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的本地方法栈，那么 Java虚拟机将会抛出一个`OutofMemoryError`异常

1. 在Hotspot虚拟机中，Java虚拟机栈和本地方法栈实现上使用了同一个栈空间，即**在 Hotspot JVM 中，直接将本地方法栈和虚拟机栈合二为一了**。

2. 本地方法栈会在栈内存上生成一个栈帧，临时保存方法的参数同时方便出现异常时也把本地方法的栈信息打印出来。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231113164905429-1403218080.png" alt="image-20231113164905329" />



```java
package com.zixieqing.runtime_data_area.stack;

import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

/**
 * <p>
 * JVM -> 运行时数据区 -> 本地方法栈
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

public class NativeMethodStack {
    public static void main(String[] args) {
        try {
            // 这里并没有F盘
            FileOutputStream fos = new FileOutputStream("F:\\zixq.txt");
            fos.write(2);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}


// 结果
java.io.FileNotFoundException: F:\zixq.txt (系统找不到指定的路径。)
	at java.io.FileOutputStream.open0(Native Method)		// 这里就把Native方法信息也打印出来了
	at java.io.FileOutputStream.open(FileOutputStream.java:270)
	at java.io.FileOutputStream.<init>(FileOutputStream.java:213)
	at java.io.FileOutputStream.<init>(FileOutputStream.java:101)
	at com.zixieqing.runtime_data_area.stack.NativeMethodStack.main(NativeMethodStack.java:20)
```

3. 本地方法可以通过本地方法接口来访问虚拟机内部的运行时数据区，它甚至可以直接使用本地处理器中的寄存器，直接从本地内存的堆中分配任意数量的内存。
4. 并不是所有 JVM 都支持本地方法，因为 Java 虚拟机规范并没有明确要求本地方法栈的使用语言、具体实现方式、数据结构等，如果 JVM 产品不打算支持 native 方法，也可以无需实现本地方法栈。







### 堆区（Heap Area）

> **栈是运行时的单位，而堆是存储的单位**。
>
> - 栈解决的是程序的运行问题，即程序如何执行，或者说如何处理数据。
> - 堆解决的是数据存储的问题，即数据怎么放、放在哪。



> 一般Java程序中堆内存是空间最大的一块内存区域。**创建出来的对象都存在于堆上。几乎所有的对象实例以及数据都在这里分配内存。**
>
> ==被所有线程共享，会发生内存溢出==。
>
> - 栈上的局部变量表中，可以存放堆上对象的引用。静态变量也可以存放堆对象的引用，通过静态变量就可以实
>
>   现对象在线程之间共享。
>
> <img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231113231643195-2127233617.png" alt="image-20231113231643098" />
>
> - **Java 虚拟机规范规定，Java 堆可以是处于物理上不连续的内存空间中，只要逻辑上是连续的即可**。
> - 堆的大小可以是固定大小，也可以是可扩展的，主流虚拟机都是可扩展的（通过 `-Xmx` 和 `-Xms` 控制）。
> - **如果堆中没有完成实例分配，并且堆无法再扩展时，就会抛出 `OutOfMemoryError` 异常**。

**堆内存溢出模拟**：

```java
import java.util.ArrayList;

/**
 * <p>
 * 模拟堆内存溢出
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

public class OutOfMemoryTest {

    private static ArrayList<Object> list = new ArrayList<>();

    public static void main(String[] args) {
        while (true) {
            list.add(new byte[1024 * 1024 * 10]);
        }
    }
}


// 结果
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.zixieqing.runtime_data_area.heap.OutOfMemoryTest.main(OutOfMemoryTest.java:20)
```



为了进行高效的垃圾回收（GC后续说明），虚拟机把堆内存**逻辑上**划分成三块区域（分代的唯一理由就是优化 GC 性能）：

![JDK7](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231113225142450-1115685021.jpg)

1. **新生代 / 年轻代 （Young Generation）**：新对象和没达到一定年龄的对象都在新生代。

年轻代是所有新对象创建的地方。当填充年轻代时，执行垃圾收集，这种垃圾收集称为 **Young GC**。

年轻代被分为三个部分——伊甸园（**Eden Memory**，伊甸园-上帝创造夏娃）和两个幸存区（**Survivor Memory**，被称为 from / to 或 s0 / s1），默认比例是`8:1:1`，这个比例可以通过 `-XX:SurvivorRatio` 来配置。

- 大多数新创建的对象都位于 Eden 内存空间中。
- 当 Eden 空间被对象填充时，执行**Minor GC**，并将所有幸存者对象移动到一个幸存者空间中。
- Minor GC 检查幸存者对象，并将它们移动到另一个幸存者空间。所以每次，一个幸存者空间总是空的。
- 经过多次 GC 循环后存活下来的对象被移动到老年代。通常，这是通过设置年轻一代对象的年龄阈值来实现的，然后他们才有资格提升到老一代。



2. **老年代 / 养老区 （Old Generation）**：被长时间使用的对象，老年代的内存空间比年轻代更大（因大部分对象都是创建出来之后就使用了，之后就不再使用了，就可以回收了。如：自己封装的对象-获取订单到返回订单就基本上不用了）。

旧的一代内存包含那些经过许多轮小型 GC 后仍然存活的对象。通常，垃圾收集是在老年代内存满时执行的。老年代垃圾收集称为 主GC（Major GC），通常需要更长的时间。

大对象直接进入老年代（大对象是指需要大量连续内存空间的对象），这样做的目的是避免在 Eden 区和两个Survivor 区之间发生大量的内存拷贝。



> 默认情况下新生代和老年代的比例是 1:2，可以通过 `–XX:NewRatio` 来配置。
>
> 
>
> 若在 JDK 7 中开启了 `-XX:+UseAdaptiveSizePolicy`，JVM 会动态调整 JVM 堆中各个区域的大小以及进入老年代的年龄，此时 `–XX:NewRatio` 和 `-XX:SurvivorRatio` 将会失效，而 JDK 8 是默认开启`-XX:+UseAdaptiveSizePolicy`。
>
> 在 JDK 8中，**不要随意关闭**`-XX:+UseAdaptiveSizePolicy`，除非对堆内存的划分有明确的规划。



3. **元空间（Meta Space） / 永久代（Permanent-Generation）**：JDK1.8前是永久代，JDK1.8及之后是元空间。JDK1.8 之前是占用 JVM 内存，JDK1.8 之后直接使用物理 / 直接内存。

不管是 JDK8 之前的**永久代**，还是 JDK8 及之后的**元空间**，**都是 Java 虚拟机规范中方法区的实现**。因此：这点内容在这里不多说明，到后面方法区再进行说明。





#### 对象在堆中的生命周期

1. 在 JVM 内存模型的堆中，堆被划分为新生代和老年代 
   - 新生代又被进一步划分为 **Eden区** 和 **Survivor区**，Survivor 区由 **From Survivor** 和 **To Survivor** 组成
2. 当创建一个对象时，对象会被优先分配到新生代的 Eden 区 
   - 此时 JVM 会给对象定义一个**对象年龄计数器**（`-XX:MaxTenuringThreshold`）
3. 当 Eden 空间不足时，JVM 将执行新生代的垃圾回收（Minor GC） 
   - JVM 会把存活的对象转移到 Survivor 中，并且对象年龄 +1
   - 对象在 Survivor 中同样也会经历 Minor GC，每经历一次 Minor GC，对象年龄都会+1
4. 如果分配的对象超过了`-XX:PetenureSizeThreshold`，对象会**直接被分配到老年代**





#### 对象的分配过程

> 涉及的内容对于初识JVM的人有点超纲，后续会慢慢了解。

为对象分配内存是一件非常严谨和复杂的任务，JVM 的设计者们不仅需要考虑内存如何分配、在哪里分配等问题，并且由于内存分配算法和内存回收算法密切相关，所以还需要考虑 GC 执行完内存回收后是否会在内存空间中产生内存碎片。

1. new 的对象先放在伊甸园区，此区有大小限制。
2. 当伊甸园的空间填满时，程序又需要创建对象，JVM 的垃圾回收器将对伊甸园区进行垃圾回收（Minor GC），将伊甸园区中不再被其他对象所引用的对象进行销毁，再加载新的对象放到伊甸园区。
3. 然后将伊甸园中的剩余对象移动到幸存者 0 区。
4. 如果再次触发垃圾回收，此时上次幸存下来的放到幸存者 0 区，如果没有被回收，就会放到幸存者 1 区。
5. 如果再次经历垃圾回收，此时会重新放回幸存者 0 区，接着再去幸存者 1 区。
6. 什么时候才会去养老区呢？ 默认是 15 次回收标记。
7. 在养老区，相对悠闲，当养老区内存不足时，触发 Major GC，进行养老区的内存清理。
8. 若养老区执行了 Major GC 之后发现依然无法进行对象的保存，就会产生 OOM 异常。





#### 堆内存三个值：used、total、max

> **used**：指的是**当前已使用的堆内存**；
>
> **total**：指的是Java虚拟机**已经分配的可用堆内存**；
>
> **max**：指的是Java虚拟机可以**分配的最大堆内存**。

![image-20231114202838913](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240202221109510-2054934690.png)



#### Arthas中堆内存相关的功能

1. **堆内存used total max三个值可以通过dashboard命令看到**。

```txt
手动指定刷新频率（不指定默认5秒一次）：dashboard –i 刷新频率(毫秒)

要是只想看内存栏的数据，可以直接使用指令：memory
```



```java
package com.zixieqing.runtime_data_area.heap;

import java.io.IOException;
import java.util.ArrayList;

/**
 * <p>
 * Arthas中堆内存的相关功能
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

public class Arthas_Heap {

    public static void main(String[] args) throws IOException, InterruptedException {

        ArrayList<Object> list = new ArrayList<>();
        
        System.in.read();

        while (true) {
            list.add(new byte[1024 * 1024 * 100]);
        }
    }
}
```



<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231114203714413-1108140369.png" alt="image-20231114203712309" />



2. **随着堆中的对象增多，当total可以使用的内存即将不足时，Java虚拟机会继续分配内存给堆**。

![image-20231114204015965](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231114204016590-1777970374.png)

3. **如果堆内存不足，Java虚拟机就会不断的分配内存，total值会变大。==total最多只能与max相等==**。

![image-20231114204203055](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231114204203609-948403832.png)



> 问题：**是不是当used = max = total的时候，堆内存就溢出了？**

**不是**，堆内存溢出的判断条件比较复杂（具体内容需要到后续的垃圾回收相关内容去了解）。





#### 设置堆内存大小

> **如果不设置任何的虚拟机参数，max默认是系统内存的1/4，total默认是系统内存的1/64**。==在实际应用中一般都需要设置total和max的值==
>
> Oracle官方文档：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html
>
> **要修改堆内存大小，可以使用虚拟机参数 `-Xms值` (初始的total) 和 `–Xmx值`（max最大值）**。

Java 堆用于存储 Java 对象实例，那么堆的大小在 JVM 启动的时候就确定了，我们可以通过 `-Xmx` 和 `-Xms` 来设定

- `-Xms` 用来表示堆的起始内存，等价于 `-XX:InitialHeapSize`
- `-Xmx` 用来表示堆的最大内存，等价于 `-XX:MaxHeapSize`

> **在工作中，这两个值一般配置的都是相同的值**
>
> - 好处：这样在程序启动之后可使用的总内存就是最大内存，而无需向java虚拟机再次申请，减少了申请并分配内存时间上的开销，同时也不会出现内存过剩之后堆收缩的情况。

PS：如果堆的内存大小超过 `-Xmx` 设定的最大内存， 就会抛出 `OutOfMemoryError` 异常

```txt
语法：-Xmx值 -Xms值

单位：字节（默认，必须是 1024 的倍数）、k或者K(KB)、m或者M(MB)、g或者G(GB)

限制：Xmx必须大于 2 MB，Xms必须大于1MB

示例：
-Xms6291456
-Xms6144k
-Xms6m
-Xmx83886080
-Xmx81920k
-Xmx80m
```

可以通过代码获取到我们的设置值：

```java
public static void main(String[] args) {

  // 返回 JVM 堆大小
  long initalMemory = Runtime.getRuntime().totalMemory() / 1024 /1024;
  // 返回 JVM 堆的最大内存
  long maxMemory = Runtime.getRuntime().maxMemory() / 1024 /1024;

  System.out.println("-Xms : "+initalMemory + "M");
  System.out.println("-Xmx : "+maxMemory + "M");

  System.out.println("系统内存大小：" + initalMemory * 64 / 1024 + "G");
  System.out.println("系统内存大小：" + maxMemory * 4 / 1024 + "G");
}
```



> 拓展点：为什么Arthas中显示的heap堆大小与设置的值不一样？
>
> PS：测试自行在IDEA中设置下面内容的Java虚拟机参数，然后启动Arthas，不断添加对象，在Arthas中输入`memory`参数进行对比查看。

```bash
-Xms1g -Xmx1g
```

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231114211409212-284352653.png" alt="image-20231114211407634" />

原因：**arthas中**的heap堆内存使用了JMX技术的内存获取方式，这种方式与垃圾回收器有关，**计算的是可以分配对象的内存**，而不是整个内存。

即：虽然设置了多少内存，但有一点内存是不会用到的，也就是JMX技术会把这部分内存去掉，不申请这部分内存，因为这点内存也放不下一个新对象，因此申请了也是浪费。







### 方法区（Method Area）

> 方法区只是 JVM 规范中定义的一个概念。并没有规定如何去实现它，不同的厂商有不同的实现。
>
> **永久代（PermGen）是 Hotspot 虚拟机特有的概念， Java8 的时候被 元空间**取代了，永久代和元空间都是方法区的落地实现方式。
>
> ==方法区是线程共享的，并且也有内存溢出==
>
> - PS：如果方法区域中的内存不能用于满足分配请求，则 Java 虚拟机抛出 `OutOfMemoryError`。



HotSpot虚拟机中：

- 永久代是HopSpot虚拟机中才有的概念。
- JDK1.6及之前，方法区的实现方式是永久代，是在堆区中（运行时常量池[里面的逻辑包含了字符串常量池]、静态变量就存储在这个永久代中）
- JDK1.7，方法区的实现方式还是永久代，也还在堆区中，但逐步“去永久代”，将字符串常量池、静态变量放到了堆区中（java.lang.Class对象）
- JDK1.8及之后，取消永久代，方法区的实现方式变为了堆+元空间，元空间位于操作系统维护的直接内存中，默认情况下只要不超过操作系统承受的上限就可以一直分配内存。
  - PS：此时，类型信息、字段、方法、常量保存在元空间中，字符串常量池、静态变量还存储在堆区中（java.lang.Class对象）





#### Java与直接内存

> 直接内存（Direct Memory）并不在《Java虚拟机规范》中存在，所以并不属于Java运行时的内存区域。

在 JDK 1.4 中引入了 NIO 机制，使用了直接内存，主要为了解决以下两个问题：

1. Java堆中的对象如果不再使用要回收，回收时会影响对象的创建和使用。
2. 以前，IO操作比如读文件，需要先把文件读入直接内存（缓冲区）再把数据复制到Java堆中。现在，直接放入直接内存即可，同时Java堆上维护直接内存的引用，减少了数据复制的开销。写文件也是类似的思路。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231117222445137-274627799.png" alt="image-20231117222444427" />



> **要创建直接内存上的数据，可以使用`java.nio.ByteBuffer`。** 
>
> 语法：`ByteBuffer directBuffer = ByteBuffer.allocateDirect(size);`
>
> 注意：也会抛`OutOfMemoryError`。

```java
package com.zixieqing.runtime_data_area;

import java.io.IOException;
import java.nio.ByteBuffer;
import java.util.ArrayList;
import java.util.List;

/**
 * <p>
 * Java使用直接内存Direct Memory
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

public class DirectMemory {

    /**
     * SIZE = 100mb
     */
    private static int SIZE = 1024 * 1025 * 100;

    private static List<ByteBuffer> LIST = new ArrayList<>();

    private static int COUNT = 0;

    public static void main(String[] args) throws IOException, InterruptedException {

        System.in.read();

        while (true) {
            ByteBuffer byteBuffer = ByteBuffer.allocateDirect(SIZE);
            LIST.add(byteBuffer);
            System.out.println("COUNT = " + (++COUNT));
            Thread.sleep(5000);
        }
    }
}
```



> Arthas的memory命令可以查看直接内存大小，属性名direct。

![image-20231117225342871](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231117225344541-1658102154.png)

> 如果需要手动调整直接内存的大小，可以使用 `-XX:MaxDirectMemorySize=值`

默认不设置该参数情况下，JVM 自动选择最大分配的大小。示例如下：

```bash
-XX:MaxDirectMemorySize=1m
-XX:MaxDirectMemorySize=1024k
-XX:MaxDirectMemorySize=1048576
```





#### 方法区的内部结构

> 说的是HotSpot虚拟机，且是JDK1.8及之后。

主要包含三部分内容：

1. **类的元信息**：保存所有类的基本信息。一般称之为**InstanceKlass对象。**在类的加载阶段完成。这一点就是前面一开始玩的字节码文件内容

![image-20231117170921007](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231117170922195-650785069.png)

2. **运行时常量池**：字节码文件中通过编号查表的方式找到常量，这种常量池称为静态常量池。当常量池加载到内存中之后，可以通过内存地址快速的定位到常量池中的内容，这种常量池称为运行时常量池。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231117214933506-211615087.png" alt="image-20231117214931503" />

常量池表（Constant Pool Table）是 Class 文件的一部分，用于存储编译期生成的各种字面量和符号引用，**这部分内容将在类加载后存放到方法区的运行时常量池中**。

> 为什么需要常量池？
>
> - 一个 Java 源文件中的类、接口，编译后产生一个字节码文件。而 Java 中的字节码需要数据支持，通常这种数据会很大以至于不能直接存到字节码里，换另一种方式，可以存到常量池，这个字节码包含了指向常量池的引用。在动态链接的时候用到的就是运行时常量池



3. **字符串常量池（**StringTable**）**：JDK7后，在堆中，存储的是在代码中定义的常量字符串内容。比如“123” 这个123就会被放入字符串常量池。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231117215324003-945968567.png" alt="image-20231117215323048" />

> JDK6中，`String.intern()` 方法会把第一次遇到的字符串实例复制到永久代的字符串常量池中，返回的也是永久代里面这个字符串实例的引用。
>
> JDK7及之后中，由于字符串常量池在堆上，所以 `String.intern()` 方法会把第一次遇到的字符串的引用放入字符串常量池。

如下图：上为JDK6的结果，下为JDK7及之后中的结果

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231117221921037-171634504.png" alt="image-20231117221920037" />

早期设计时（JDK1.6及之前），字符串常量池是属于运行时常量池的一部分，他们存储的位置也是一致的（堆中的永久代空间），后续做出了调整，大意图如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231117215459171-718250975.png" alt="image-20231117215458355" />





#### 设置方法区大小

1. JDK1.7及之前，通过下列参数设置永久代空间大小。

```txt
-XX:PermSize=值 和 -xx:MaxPermSize=值
```

2. JDK1.8及之后，通过下列参数设置元空间大小。

```txt
-XX:MetaspaceSize=值 和 -XX:MaxMetaspaceSize=值
```

- 默认值依赖于平台。Windows 下，`-XX:MetaspaceSize` 是 21M，`-XX:MaxMetaspacaSize` 的值是 -1，即没有限制。
- 与永久代不同，如果不指定大小，默认情况下，虚拟机会耗尽所有的可用系统内存。如果元数据发生溢出，虚拟机一样会抛出异常 `OutOfMemoryError:Metaspace`。
- `-XX:MetaspaceSize` ：到达这个值后触发Full GC。对于一个 64 位的服务器端 JVM 来说，其默认的 `-XX:MetaspaceSize` 的值为20.75MB，这就是初始的高水位线，一旦触及这个水位线，Full GC 将会被触发并卸载没用的类（即这些类对应的类加载器不再存活），然后这个高水位线将会重置，新的高水位线的值取决于 GC 后释放了多少元空间。如果释放的空间不足，那么在不超过 `MaxMetaspaceSize`时，适当提高该值。如果释放空间过多，则适当降低该值。
- 如果初始化的高水位线设置过低，上述高水位线调整情况会发生很多次，通过垃圾回收的日志可观察到 Full GC 多次调用。为了避免频繁 GC，建议将 `-XX:MetaspaceSize` 设置为一个相对较高的值。





### 对象在堆中的内存布局

> 对象在堆中的内存布局（对象在堆中如何存储的），指的是对象在堆中存放时的各个组成部分

主要分为以下几个部分：

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240124203647336-456379244.png" alt="image-20240124203702225" />

1. **标记字段（Mark Word）**：保存对象自身的运行时数据，`HashCode`、`GC Age`、`锁标记位`、`是否为偏向锁`等。

> 这标记字段中GC Age是用的4 bit来存储的，而4 bit能存储的最大值是15，所以就知道为什么GC Age最大是15，而不是32或其他值了。

标记字段相对比较复杂。在不同的对象状态（有无锁、是否处于垃圾回收的标记中）下存放的内容是不同的。同时在64位（又分为是否开启指针压缩）、32位虚拟机中的布局都不同。

以64位开启指针压缩：JDK8默认开启指针压缩

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240124201118067-576985215.png" alt="image-20240124201132092" />

64位不开启指针压缩，只是将Cms使用这部分弃用:

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240124201955847-1092348088.png" alt="image-20240124202010607" />



**查看对象布局的方式**：JOL工具。

JOL是用于分析 JVM 中对象布局的一款专业工具。工具中使用 Unsafe、JVMTI 和 Serviceability Agent (SA)等虚拟机技术来打印实际的对象内存布局。

- 依赖

```xml
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.9</version>
</dependency>
```

- 打印对象内存布局

```java
System.out.println(ClassLayout.parseInstance(对象).toPrintable());
```



2. **元数据的指针（Klass pointer）**：指向方法区中保存的InstanceKlass对象。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240124203539107-653199281.png" alt="image-20240124203553856" />



> 指针压缩：可以使用`-XX:-UseCompressedOops`关闭。

在64位的Java虚拟机中，Klass Pointer以及对象数据中的对象引用都需要占用8个字节，为了减少这部分的内存使用量，64 位 Java 虚拟机使用指针压缩技术，将堆中原本 8个字节的 指针压缩成 4个字节。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240124202645267-384439322.png" alt="image-20240124202700148" />

**指针压缩的思想**：将寻址的单位放大。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240124202826495-667850049.png" alt="image-20240124202841509" />

如上图所示，原来按编号1去寻址，能拿到1字节开始的数据；现在按1去寻址，就可以拿到8个字节开始的数据。所以结果是原来按1字节去寻址，现在可以按8字节寻址。

因此优点是：这样将编号当成地址，就可以用更小的内存访问更多的数据。

但是也会引发两个问题：

- **需要进行内存对齐**：指的是将对象的内存占用填充至"8字节的倍数"。存在空间浪费（对于Hotspot来说不存在，即便不开启指针压缩，也需要进行内存对齐） 

- **寻址大小仅仅能支持2^35^ 个字节（32GB，如果超过32GB指针压缩会自动关闭）**：不用压缩指针，应该是2^64^ = 16EB，用了压缩指针就变成了8（字节） ： 2^3^ * 2^32^ = 2^35^。



3. **内存对齐填充**：将对象的内存占用填充至"8字节的倍数"。主要目的是为了解决并发情况下CPU缓存失效的问题。

**不采用内存对齐**：会造成多个对象在同一缓存行中，如下图A对象与B对象

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240124203837660-1958693400.png" alt="image-20240124203852982" />

CPU缓存修改A对象并将其写回内存，但另一个线程B读取的是B对象，由于A、B对象在同一缓存行，A对象的数据写回内存了，但B对象可没有，所以线程B就会读取到B对象的假数据。

**采用内存对齐**：同一个缓存行中不会出现不同对象的属性。在并发情况下，如果让A对象一个缓存行失效，是不会影响到B对象的缓存行的。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240124204245449-168910782.png" alt="image-20240124204300424" />



> 内存对齐涉及的小知识：字段重排列。在Hotspot中，要求每个属性的偏移量Offset（字段地址 – 起始地址）必须是字段长度的N倍。如果不满足要求，会尝试使用内存对齐，通过在属性之间插入一块对齐区域达到目的。

如下图中（JOL打印），Student类中的id属性类型为long，那么偏移量就必须是8的倍数

![image-20240124204710320](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240124204655010-473016754.png)

> 这里有个小注意点（字段重排列通俗解释）：为了保证偏移量是8的倍数（这里的8说的是上图的long，类型不同该值不同），所以会进行字段顺序的重排列。
>
> 如：上图id在代码中声明可能是在前面（即age的位置），但age的offset是12，不满足8的倍数，所以将字段id重排列，放到offset为16处（即和字段age换位了）。

如下图，name字段是引用类型占用8个字节（关闭了指针压缩），因为Offset必须是8的倍数，所以在age和name之间插入了4个字节的空白区域。28 + 4 = 32，是8的倍数。

![image-20240124204930241](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240124204915282-354279928.png)

> 提一嘴儿：子类继承自父类的属性，那么子类中会有和父类顺序一致的属性的偏移量，并且子类中属性的偏移量是先按父类属性偏移量的顺序排列好，再排列子类本身的属性偏移量。

```java
import org.openjdk.jol.info.ClassLayout;

public class A {
    long l;
    long m;
    String name;
}

class B extends A{
    int a;
    int b;
}

class Test {
    public static void main(String[] args) {
        System.out.println(ClassLayout.parseInstance(new B()).toPrintable());
    }
}
```

![image-20240124211139355](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240124211125168-1279416241.png)





### 栈中的数据要保存到堆上 或 从堆中加载到栈上时怎么处理？

1. **堆中的数据加载到栈上**：由于栈上的空间大于或者等于堆上的空间，所以直接处理，但是需要注意下"符号位"。

- 无符号，低位复制，高位补0

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240124214117490-1692618746.png" alt="image-20240124214132699" style="zoom: 50%;" />

- 有符号，低位复制，高位非负则补0，负则补1 

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240124214257149-1984047281.png" alt="image-20240124214312504" style="zoom: 50%;" />



2. **栈中的数据要保存到堆上**：由于堆上存储空间较小，需要将高位去掉（boolean比较特殊，只取低位的最后一位保存）。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240124214742778-1963005210.png" alt="image-20240124214757551" />





> 小测试：通过字节码指令修改方式将iconst1修改为iconst2和iconst3会得到截然不同的结果。这个就涉及到上面所说的堆 -> 栈；栈 -> 堆，以及boolean的存储方式。
>
> - 字节码指令：IDEA中可以安装ASM Bytecode Outline插件生成类的ASM代码，也可以用其他方式。

```java
package com.zixieqing;

/**
 * <p>
 * 栈中数据保存到堆中、堆中数据保存到栈中。iconst2 -> 2 = 10	iconst3 -> 3 = 11
 * boolean：1为true、0为false		boolean取低位最后一位
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

public class StackStore {
    static boolean a;	// 堆上

    public static void main(String[] args) {
        a = true;	// 栈上

        if (a) {
            System.out.println("a为true");
        } else {
            System.out.println("a为false");
        }

        if (a == true) {
            System.out.println("a为true");
        } else {
            System.out.println("a为false");
        }
    }
}
```







### 方法调用原理

> 方法调用的本质是通过字节码指令的执行，能在栈上创建栈帧，并执行调用方法中的字节码执行。以`invoke`开头的字节码指令的作用就是找到字节码指令并执行。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240125215605430-377793222.png" alt="image-20240125215351842" />

在JVM中，一共有五个字节码指令可以执行方法调用：

1. **invokestatic**：调用静态方法。
2. **invokespecial**：调用对象的private方法、构造方法，以及使用 super 关键字调用父类实例的方法、构造方法，以及所实现接口的默认方法。
3. **invokevirtual**：调用对象的非private方法。
4. **invokeinterface**：调用接口对象的方法。
5. **invokedynamic**：用于调用动态方法，主要应用于lambda表达式中，机制极为复杂了解即可



> Invoke指令执行时，需要找到方法区中instanceKlass中保存的方法相关的字节码信息。但是方法区中有很多类，每一个类又包含很多个方法，怎么精确地定位到方法的位置呢？

1. **静态绑定**：指的是方法第一次调用时，符号引用被替换成内存地址的直接引用。

编译期间，invoke指令会携带一个参数符号引用，引用到常量池中的方法定义。方法定义中包含了类名 + 方法名 + 返回值 + 参数。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240125220644981-457065500.png" alt="image-20240125220700536" />

适用于处理静态方法、私有方法、或者使用final修饰的方法，因为这些**方法不能被继承之后重写**。即：**invokestatic**、**invokespecial**、**final修饰的invokevirtual**指令。



2. **动态绑定**：非static、非private、非final的方法，**有可能存在子类重写方法**，那么就需要通过动态绑定来完成方法地址绑定的工作。

动态绑定是基于方法表来完成的，invokevirtual使用了虚方法表（vtable），invokeinterface使用了接口方法表(itable)，整体思路类似。

> 虚方法表，本质上是一个数组，记录了方法的内存地址。每个类中都有一个虚方法表，子类方法表中包含父类方法表中的所有方法；子类如果重写了父类方法，则使用子类自己的方法的地址进行替换。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240125221404584-279902103.png" alt="image-20240125221419659" />

使用invokevirtual和虚方法表来解释整个过程：产生invokevirtual调用时，先根据对象头中的类型指针找到方法区中InstanceKlass对象，获得虚方法表，再根据虚方法表找到对应的地方，获得方法的地址，最后调用方法。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240125221737532-1546994555.png" alt="image-20240125221753257" />





## 执行引擎

> 执行引擎：执行本地已经编译好的方法，如虚拟机提供的C++方法。
>
> 包括：即时编译器（JIT，即Just-in-time）、解释器、垃圾回收器等。



<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231119165759216-474366881.png" alt="image-20231110204142543" />

### 自动垃圾回收

> 在C/C++这类没有自动垃圾回收机制的语言中，一个对象如果不再使用，需要手动释放，否则就会出现内存泄漏。我们称这种释放对象的过程为垃圾回收，而需要程序员编写代码进行回收的方式为手动回收。 
>
> - PS：内存泄漏指的是不再使用的对象在系统中未被回收，内存泄漏的积累可能会导致内存溢出。

Java中为了简化对象的释放，引入了自动垃圾回收（Garbage Collection简称GC）机制。**虚拟机通过垃圾回收器来对不再使用的对象完成自动的回收**，**垃圾回收器主要负责对堆上的内存进行回收**。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231119170950627-1731223678.png" alt="image-20231119170949432" />

> 自动垃圾回收与手动垃圾回收的优缺点。

1. **自动垃圾回收**：自动根据对象是否使用由虚拟机来回收对象。

- 优点：降低程序员实现难度、降低对象回收bug的可能性。
- 缺点：程序员无法控制内存回收的及时性。

2. **手动垃圾回收**：由程序员编程实现对象的删除。

- 优点：回收及时性高，由程序员把控回收的时机。
- 缺点：编写不当容易出现悬空指针、重复释放、内存泄漏等问题。





### 方法区的回收

> **线程不共享的部分（程序计数器、Java虚拟机栈、本地方法栈），都是伴随着线程的创建而创建，线程的销毁而销毁**。而方法的栈帧在执行完方法之后就会自动弹出栈并释放掉对应的内存，因此不需要对这部分区域进行垃圾回收。
>
> 
>
> **由前面对方法区的了解可知：方法区中能回收的内容主要就是常量池中废弃的常量和不再使用的类型**。
>
> 开发中此类场景一般很少出现，主要在如 OSGi、JSP 的热部署等应用场景中。每个jsp文件对应一个唯一的类加载器，当一个jsp文件修改了，就直接卸载这个jsp类加载器。重新创建类加载器，重新加载jsp文件。

在前面类的生命周期中，最后一步是卸载（unloading），而判定一个类可以被卸载。需要**同时满足**下面三个条件：

> 可以使用虚拟机参数 `-verbose:class` 或 `-XX:+TraceClassLoading` 、`-XX:+TraceClassUnloading` 查看类加载和卸载信息。
>
> `-XnoClassgc` 参数可以关闭类的GC / 控制是否对类进行卸载。在垃圾收集时类对象不会被回收，会被认为总是存活的，这将导致存放类对象的内存被持续占用，如果不谨慎使用，将可能导致OOM。

1. **此类所有实例对象都已经被回收**。在堆中不存在任何该类的实例对象以及子类对象。 

2. **加载该类的类加载器已经被回收**。

3. **该类对应的 `java.lang.Class` 对象没有在任何地方被引用**。

```java
package com.zixieqing.execution_engine.method_area_collection;

import com.zixieqing.class_and_classloader.FindClassLoader;

/**
 * <p>
 * 卸载（unloading）类“同时满足”的三个条件
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

public class UnloadingConditions {

    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, InterruptedException {

        ClassLoader classLoader = FindClassLoader.class.getClassLoader();
        Class<?> clazz = classLoader.loadClass("com.zixieqing.class_and_classloader.FindClassLoader");
        Object obj = clazz.newInstance();
        // 1、该类所有实例对象都被回收   反例证明：将obj这个对象放到其他地方去，让其被持有则不会不会被卸载了
        obj = null;

        // 2、加载该类的类加载器已经被回收     反例证明：将此类加载器其他持有就不会被卸载
        classLoader = null;

        // 3、该类对应的 java.lang.Class 对象没有在任何地方被引用     反例证明：该对象被其他持有
        clazz = null;

        // 手动触发垃圾回收
        // 不一定会立即回收垃圾，仅仅是向Java虚拟机发送一个垃圾回收的请求，具体是否需要执行垃圾回收Java虚拟机会自行判断
        System.gc();
    }
}
```



> 补充：关于 `finalize()` 方法

`finalize()` 类似 C++ 的析构函数，用来做关闭外部资源等工作。但是 try-finally 等方式可以做的更好，并且该方法运行代价高昂，不确定性大，无法保证各个对象的调用顺序，因此最好不要使用。

当一个对象可被回收时，如果需要执行该对象的 `finalize()` 方法，那么就有可能通过在该方法中让对象重新被引用，从而实现自救，自救只能进行一次，如果回收的对象之前调用了 `finalize()` 方法自救，后面回收时不会调用 `finalize()` 方法。





### 堆回收

#### 如何判断堆上的对象是否可被回收？

> Java中的对象是否能被回收，是根据对象是否被引用来决定的。如果对象被引用了，说明该对象还在使用，不允许被回收。

![image-20231210163809458](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231210163812081-1703581880.png)

而判断对象是否有引用有两种判断方法：引用计数法和可达性分析法。



##### 引用计数法

> 引用计数法会为每个对象维护一个引用计数器，当对象被引用时加1，取消引用（或引用失效）时减1，引用计数为 0 的对象可被回收。

引用计数法的优点是实现简单，C++中的智能指针就采用了引用计数法，但是它也存在缺点，主要有两点：

1. 每次引用和取消引用都需要维护计数器，对系统性能会有一定的影响。
2. 存在循环引用问题，此时引用计数器永远不为 0，导致无法对它们进行回收。所谓循环引用就是当A引用B，B同时引用A时会出现对象无法回收的问题。

![image-20231210171559057](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231210171600079-1092784022.png)



##### 可达性分析法

> 可达性分析将对象分为两类：垃圾回收的根对象（GC Root）和普通对象，对象与对象之间存在引用关系。通过 GC Roots 作为起始点进行搜索，能够到达到的对象都是存活的，不可达的对象可被回收。
>
> **Java 虚拟机使用的是可达性分析算法来判断对象是否可以被回收。**
>
> ==注：可达性算法中描述的对象引用，一般指的是强引用（另外几种引用后续会说明）。==即是GCRoot对象对普通对象有引用关系，只要这层关系存在，普通对象就不会被回收。

![image](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231210172002410-6587695.png)

**哪些对象被称之为GC Root对象？**

1. 线程Thread对象，引用线程栈帧中的方法参数、局部变量等。
2. 系统类加载器加载的java.lang.Class对象，引用类中的静态变量。

![image-20231210172758467](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231210172759222-1322912237.png)

3. 监视器对象，用来保存同步锁synchronized关键字持有的对象。

![image-20231210172859422](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231210172900410-1547384601.png)

4. 本地方法调用时使用的全局对象。





**在 Java 中 GC Roots 一般包含哪些内容？**

1. 虚拟机栈中引用的对象。
2. 本地方法栈中引用的对象。
3. 方法区中类静态属性引用的对象。
4. 方法区中的常量引用的对象。





##### 引用类型

> 无论是通过引用计算算法判断对象的引用数量，还是通过可达性分析算法判断对象是否可达，判定对象是否可被回收都与引用有关。共有5种引用类型（由强到弱）：强引用、软引用、弱引用、虚引用、终结器引用



###### 强引用

> 强引用：指的是GCRoot对象对普通对象有引用关系，即由可达性分析法判断，只要这层关系存在，普通对象就不会被回收。

创建强引用的方式：“可以使用” new 的方式来创建强引用。PS：别钻牛角尖，来个 `new SoftReference<Object>(obj);`

```java
Object obj = new Object();
```



###### 软引用

> 软引用相对于强引用是一种比较弱的引用关系。
>
> 如果一个对象只有软引用关联到它，则当程序内存不足时，就会将此软引用中的数据进行回收。

在JDK 1.2版之后提供了SoftReference类来实现软引用，软引用常用于缓存中。

```java
Object obj = new Object();
SoftReference<Object> sf = new SoftReference<Object>(obj);
obj = null;  // 使对象只被软引用关联
```

**软引用的执行过程如下：**

1. 将对象使用软引用包装起来，**`new SoftReference<对象类型>(对象)`**。
2. 内存不足时，虚拟机尝试进行垃圾回收。
3. 如果垃圾回收仍不能解决内存不足的问题，回收软引用中的对象。
4. 如果依然内存不足，抛出OutOfMemory异常。



> 问题：软引用中的对象如果在内存不足时回收，SoftReference对象本身也需要被回收。如何知道哪些SoftReference对象需要回收？

SoftReference提供了一套队列机制：

1. 软引用创建时，通过构造器传入引用队列。
2. 在软引用中包含的对象被回收时，该软引用对象会被放入引用队列。
3. 通过代码遍历引用队列，将SoftReference的强引用删除。

![image-20231210215049848](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231210215051853-1875548629.png)

软引用应用场景：缓存示例

![image-20231210215151456](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231210215152458-1613924896.png)





###### 弱引用

> 弱引用的整体机制和软引用基本一致，区别在于弱引用包含的对象在垃圾回收时，不管内存够不够都会直接被回收。**即：被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前。**

JDK 1.2版之后提供了WeakReference类来实现弱引用，弱引用主要在ThreadLocal中使用。

```java
Object obj = new Object();
WeakReference<Object> wf = new WeakReference<Object>(obj);
obj = null;
```





###### 虚引用和终结器引用

> ==这两种引用在常规开发中是不会使用的。==

1. **虚引用**：也叫幽灵引用/幻影引用，不能通过虚引用对象获取到包含的对象。虚引用唯一的用途是当对象被垃圾回收器回收时可以接收到对应的通知。Java中使用PhantomReference实现了虚引用，

直接内存中为了及时知道直接内存对象不再使用，从而回收内存，使用了虚引用来实现。

```java
Object obj = new Object();
PhantomReference<Object> pf = new PhantomReference<Object>(obj);
obj = null;
```



2. **终结器引用**：指的是在对象需要被回收时，终结器引用会关联对象并放置在Finalizer类中的引用队列中，在稍后由一条FinalizerThread线程从队列中获取对象，然后执行对象的finalize方法，在对象第二次被回收时，该对象才真正的被回收。在这个过程中可以在finalize方法中再将自身对象使用强引用关联上，但是不建议这样做。







#### 垃圾回收算法

> 垃圾回收算法核心思想：
>
> 1. 找到内存中存活的对象。
> 2. 释放不再存活对象的内存，使得程序能再次利用这部分空间。



##### 判断GC算法是否优秀的标准

> Java垃圾回收过程会通过单独的GC线程来完成，但是不管使用哪一种GC算法，都会有部分阶段需要停止所有的用户线程。这个过程被称之为Stop The World简称STW，如果STW时间过长则会影响用户的使用。

1. **吞吐量**

吞吐量指的是 CPU 用于执行用户代码的时间 与 CPU 总执行时间的比值，即：吞吐量 = 执行用户代码时间 /（执行用户代码时间 + GC时间）。吞吐量数值越高，垃圾回收的效率就越高。

2. **最大暂停时间**

最大暂停时间指的是所有在垃圾回收过程中的STW时间最大值。如下图中，黄色部分的STW就是最大暂停时间，显而易见上图比下图拥有更少的最大暂停时间。最大暂停时间越短，用户使用系统时受到的影响就越短。

如下图就是上优下劣：

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211213638104-737794451.png" alt="image-20231211213636743" />

3. **堆使用效率**

不同垃圾回收算法，对堆内存的使用方式是不同的。比如标记清除算法，可以使用完整的堆内存。而复制算法会将堆内存一分为二，每次只能使用一半内存。从堆使用效率上来说，标记清除算法要优于复制算法。



> 三种评价标准：**堆使用效率、吞吐量，以及最大暂停时间不可兼得**。
>
> 一般来说，堆内存越大，最大暂停时间就越长。想要减少最大暂停时间，就会降低吞吐量。
>
> **不同的垃圾回收算法，适用于不同的场景**。





##### GC算法：标记-清除算法

> 一句话概括就是：标记存活对象，删除未标记对象。
>
> ==标记-清除算法可以使用完整的堆内存==。



![image-20231211214713125](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211214713898-653683505.png)



标记清除算法的核心思想分为两个阶段：

1. 标记阶段：将所有存活的对象进行标记。Java中使用可达性分析算法，从GC Root开始通过引用链遍历出所有存活对象。

2. 清除阶段：从内存中删除没有被标记（也就是非存活）对象。

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211214316392-219611098.png" alt="image-20231211214315968" />

优点：实现简单，只需要在第一阶段给每个对象维护标志位，第二阶段删除对象即可。

缺点：

1. 会产生内存碎片化问题。

由于内存是连续的，所以在对象被删除之后，内存中会出现很多细小的可用内存单元。如果我们需要的是一个比较大的空间，很有可能这些内存单元的大小过小无法进行分配。

2. 分配速度慢。

由于内存碎片的存在，需要维护一个空闲链表，极有可能发生每次需要遍历到链表的最后才能获得合适的内存空间。

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211214658988-1178393094.png" alt="image-20231211214658763" />



##### GC算法：标记-整理算法

> 一句话概括就是：让所有存活的对象都向堆内存的一端移动，然后直接清理掉“端边界以外”的内存。
>
> 标记整理算法也叫标记压缩算法，是对标记清理算法中容易产生内存碎片问题的一种解决方案。
>
> ==标记-整理算法可以使用整个堆内存==。



![image-20231211215213871](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211215213945-1314935861.png)

核心思想分为两个阶段：

1. 标记阶段：将所有存活的对象进行标记。Java中使用可达性分析算法，从GC Root开始通过引用链遍历出所有存活对象。

2. 整理阶段：将存活对象移动到堆的一端。清理掉非存活对象的内存空间。

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211215325857-1121448194.png" alt="image-20231211215325960" />

优点：

1. 内存使用率高：整个堆内存都可以使用。
2. 不会产生内存碎片化问题：在整理阶段可以将对象往内存的一侧进行移动，剩下的空间都是可以分配对象的有效空间。

缺点：

1. 整理阶段的效率不高：因为要去找存活和非存活对象，然后进行相应内存位置移动，这里又涉及对象引用问题，所以会造成整体性能不佳。

如：Lisp2整理算法就需要对整个堆中的对象搜索3次。当然也有优化整理阶段的算法，如Two-Finger、表格算法、ImmixFC等高效的整理算法来提升此阶段性能。





##### GC算法：复制算法

> ==复制算法每次只能使用一半堆内存==。



![image-20231211220534975](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211220535483-1449601383.png)

复制算法的核心思想：

1. 准备两块空间From空间和To空间，每次在对象分配阶段，只能使用其中一块空间（From空间）。
2. 在垃圾回收GC阶段，将From中存活对象复制到To空间。
3. 将两块空间的From和To名字互换。



优点：

1. 吞吐量高：复制算法只需要遍历一次存活对象复制到To空间即可。比标记-整理算法少了一次遍历过程，因而性能较好，但不如标记-清除算法，因标记-清除算法不需要进行对象的移动。
2. 不会发生内存碎片化问题：复制算法在复制之后就会将对象按照顺序放入To内存，所以对象以外的区域是可用空间，因此不会产生内存碎片化问题。

缺点：

1. 内存使用率低：每次只能让一半的内存空间来供创建对象使用。





##### GC算法：分代收集算法

> 主流的JVM（如：HotSpot）采用的就是此种算法。
>
> 一般将堆分为新生代和老年代：
>
> - 新生代分为伊甸园（Eden）区、两个幸存（Survivor ）区——被称为 from / to 或 s0 / s1。三者默认比例是8:1:1。
> - 新生代使用: 复制算法
> - 老年代使用: 标记 - 清除 或者 标记 - 整理 算法



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211222724175-209132338.png" alt="image-20231211222722551" />

1. 分代回收时，创建出来的对象，首先会被放入Eden伊甸园区。
2. 随着对象在Eden区越来越多，如果Eden区满，新创建的对象已经无法放入，就会触发年轻代的GC，称为Minor GC 或 Young GC。
3. Minor GC会把eden中和From需要回收的对象回收，把没有回收的对象放入To区。
4. 接下来，S0会变成To区，S1变成From区。当eden区满时再往里放入对象，依然会发生Minor GC。
5. 此时会回收eden区和S1(from)中的对象，并把eden和from区中剩余的对象放入S0。

> 注意：每次Minor GC中都会为对象记录他的年龄（或者叫回收标记次数），默认值为0（默认值和垃圾回收器有关），每次GC完加1。JVM中此值最大为15。

6. 如果Minor GC后对象的年龄达到阈值（最大值为15），对象就会被晋升至老年代。
7. 当老年代中空间不足，无法放入新的对象时，先尝试minor gc，如果还是不足，就会触发Full GC。

> 问题：为什么老年代空间不足，需要先尝试minor gc，即年轻代回收？

因为第6步中年轻代中的对象不是一定是年龄达到15才会进入老年代。年轻代空间满了，此时有些对象年龄可能是小于15的，但为了腾出可用空间，这部分对象也可能会被丢进老年代。

8. 如果Full GC依然无法回收掉老年代的对象，那么当对象继续放入老年代时，就会抛出Out Of Memory异常。



> 关于伊甸园（Eden）区有一个小知识点：TLAB （Thread Local Allocation Buffer）

- 从内存模型而不是垃圾回收的角度，对 Eden 区域继续进行划分，JVM 为每个线程分配了一个私有缓存区域，它包含在 Eden 空间内。
- 多线程同时分配内存时，使用 TLAB 可以避免一系列的非线程安全问题，同时还能提升内存分配的吞吐量，因此我们可以将这种内存分配方式称为**快速分配策略**。
- OpenJDK 衍生出来的 JVM 大都提供了 TLAB 设计。



> 为什么要有 TLAB ?

- 堆区是线程共享的，任何线程都可以访问到堆区中的共享数据。
- 由于对象实例的创建在 JVM 中非常频繁，因此在并发环境下从堆区中划分内存空间是线程不安全的。
- 为避免多个线程操作同一地址，需要使用加锁等机制，进而影响分配速度。

尽管不是所有的对象实例都能够在 TLAB 中成功分配内存，但 JVM 确实是将 TLAB 作为内存分配的首选。

在程序中，可以通过 `-XX:UseTLAB` 设置是否开启 TLAB 空间。

默认情况下，TLAB 空间的内存非常小，仅占有整个 Eden 空间的 1%，我们可以通过 `-XX:TLABWasteTargetPercent` 设置 TLAB 空间所占用 Eden 空间的百分比大小。

一旦对象在 TLAB 空间分配内存失败时，JVM 就会尝试着通过使用加锁机制确保数据操作的原子性，从而直接在 Eden 空间中分配内存。





> Arthas查看分代之后的内存情况

1. 在JDK8中，添加 `-XX:+UseSerialGC` 参数使用分代回收的垃圾回收器，运行程序。
2. 使用 `memory` 命令查看内存，显示出三个区域的内存情况。

测试时需要的JVM参数参考：JDK 1.8(版本不同有些参数会无效) + 添加 `-XX:+UseSerialGC` 参数

| 参数                                        | 参数含义                                                     | 示例                                                         |
| ------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| -Xms                                        | 设置堆的最小 / 初始 大小（相当于前面说的total）。<br />必须是1024的倍数且大于1MB。 | 设置为6MB的写法：<br />-Xms6291456<br />-Xms6144k<br />-Xms6m |
| -Xmx                                        | 设置最大堆的大小（相当于前面说的max）。<br />必须是1024倍数且大于2MB。 | 设置为80 MB的写法：<br />-Xmx83886080<br />-Xmx81920k<br />-Xmx80m |
| -Xmn                                        | 新生代的大小。<br />默认为整个堆的1 / 3                      | 设置256 MB的写法：<br />-Xmn256m<br />-Xmn262144k<br />-Xmn268435456 |
| -XX:SurvivorRatio                           | 伊甸园区和幸存区的比例，默认为8。<br />如：新生代1g内存，伊甸园区800MB，S0和S1各100MB | 比例调整为4的写法：<br />-XX:SurvivorRatio=4                 |
| -XX:+PrintGCDetails<br />或<br />verbose:gc | 打印GC日志                                                   |                                                              |





#### 垃圾回收器

> 垃圾回收器（Garbage collector，即GC）是垃圾回收算法的具体实现。
>
> **除G1之外**，**其他**垃圾回收器**必须成对组合**进行使用（如下图）。
>
> 年轻代回收都是复制算法（包括G1），老年代回收的算法不同。



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212221114589-1260192870.png" alt="image-20231212221113679" />



> 记忆方式：
>
> 1. JDK8及之前
>
> - 关注暂停时间：ParNew + CMS（CMS在使用中需测试，因CMS在回收老年代时可能会影响用户线程）。
> - 关注吞吐量：Parallel Scavenge + Parallel Old（此组合为JDK8默认）。
> - ~~较大堆且关注暂停时间（JDK8之前不建议用）：G1~~。PS：JDK8最新版算是成熟的G1，故其实可以直接使用。
>
> 2. JDK9之后：G1（默认）。生产环境中也建议使用。。





##### 垃圾回收器：Serial 与 Serial Old

###### 新生代：Serial

> Serial是一种**单线程串行回收**年轻代的垃圾回收器，采用的是**复制算法**。
>
> 垃圾回收线程进行GC时，会让用户线程进入等待，GC执行完了才会进行用户线程（即STW）。
>
> **适用场景**：Java编写的客户端程序 或者 硬件配置有限（服务器不多）的场景。或者直接说 Client 模式下的场景。
>
> - PS：Serial 收集器收集几十兆甚至一两百兆的新生代停顿时间可以控制在一百多毫秒以内，只要不是太频繁，这点停顿是可以接受的。



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212215638312-1741193841.png" alt="image-20231212215637335" />

**优点**：单CPU处理器下吞吐量非常出色。

**缺点**：多CPU下吞吐量不如其他垃圾回收器，堆如果偏大会让用户线程处于长时间的等待。





###### 老年代：Serial Old

> Serial Old是Serial垃圾回收器的老年代版本，**单线程串行回收**，采用的是**标记-整理算法**。
>
> 开启的方式：使用虚拟机参数 `-XX:+UseSerialGC` 即：新生代、老年代都使用串行回收器。
>
> 垃圾回收线程进行GC时，会让用户线程进入等待，GC执行完了才会进行用户线程。
>
> **适用场景**：与Serial垃圾回收器搭配使用 或者 在CMS特殊情况下使用（CMS时会说明）。



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212220704050-431305960.png" alt="image-20231212220703204" />

优缺点和Serial一样。

**优点**：单CPU处理器下吞吐量非常出色。

**缺点**：多CPU下吞吐量不如其他垃圾回收器，堆如果偏大会让用户线程处于长时间的等待。





##### 垃圾回收器：ParNew 与 CMS

###### 新生代：ParNew

> ParNew垃圾回收器本质上是对Serial在多CPU下的优化，但：JDK9之后不建议使用了。
>
> 使用**多线程进行垃圾回收**，采用的是**复制算法**。
>
> - PS：默认开启的线程数量与 CPU 数量相同，可以使用 `-XX:ParallelGCThreads` 参数来设置线程数。
>
> 开启方式：使用参数 `-XX:+UseParNewGC` 即：新生代使用ParNew（并行）回收器， 老年代使用串行回收器（即Serial Old）。
>
> 垃圾回收线程进行GC时，会让用户线程进入等待，GC执行完了才会进行用户线程。
>
> **适用场景**：JDK8及之前的版本中，与老年代垃圾回收器CMS搭配使用。



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212221645408-144737065.png" alt="image-20231212221644577" />

**优点**：多CPU处理器下停顿时间较短。

**缺点**：吞吐量和停顿时间不如G1，所以在JDK9之后不建议使用。



###### 老年代：CMS(Concurrent Mark Sweep)

> CMS垃圾回收器关注的是系统的暂停时间。
>
> **允许用户线程和垃圾回收线程在某些步骤中同时执行**，减少了用户线程的等待时间。采用的是**标记-清除算法**。
>
> 开启方式：使用参数 `XX:+UseConcMarkSweepGC`。即新生代使用并行（ParNew），老年代使用CMS。
>
> 初识标记 与 重新标记阶段 会让用户线程进入等待，GC执行完了才会进行用户线程。
>
> **适用场景**：大型的互联网系统中用户请求数据量大、频率高的场景。如订单接口、商品接口等。



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212222406985-474363136.png" alt="image-20231212222405963" />

**CMS执行步骤**：

1. 初始标记（Initial Mark）：用极短的时间标记出GC Roots能直接关联到的对象（可达性分析法）。 
2. 并发标记（Concurrent Mark）：标记所有的对象，用户线程不需要暂停。

这里采用了一个并发标记算法，学名叫三色标记法，G1垃圾回收器也采用的是这个算法，所以G1时再说明这个算法。

并发阶段运行时的线程数可以通过参数 `-XX:ConcGCThreads` （默认值为0）设置，由系统计算得出。即：CMS存在的线程资源争抢问题的解决方式。

```txt
计算公式为：
	(-XX:ParallelGCThreads定义的线程数 + 3) / 4


ParallelGCThreads		是STW停顿之后的并行线程数		上一节 新生代：ParNew 已说过该参数
	
	ParallelGCThreads是由处理器核数决定的：
		1、当cpu核数小于8时，ParallelGCThreads = CPU核数
		2、否则 ParallelGCThreads = 8 + (CPU核数 – 8 ) * 5 / 8
```

3. 重新标记（Remark）：由于并发标记阶段有些对象会发生变化，故存在错标、漏标等情况，因此需要重新标记。
4. 并发清理（Clean）：清理非存活对象，用户线程不需要暂停。





> CMS存在的问题

缺点：

1. CMS使用了标记-清除算法，在垃圾收集结束之后会出现大量的内存碎片，CMS会在Full GC时进行碎片的整理。这样会导致用户线程暂停，可以使用参数 ` -XX:CMSFullGCsBeforeCompaction=N` （默认0）调整N次Full GC之后再整理。
2. 无法处理在并发清理过程中产生的“浮动垃圾”，不能做到完全的垃圾回收。

“浮动垃圾”的原因：并发清理时用户线程会产生一些很快就使用、后续不用的对象，而这些对象在这次的GC中无法回收，只能等到下次GC回收，这部分垃圾就是“浮动垃圾”。

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212224114901-1389920984.png" alt="image-20231212224114153" />

3. 如果老年代内存不足无法分配对象，CMS就会退化成Serial Old单线程回收老年代（即：前面说的特殊情况会调用Serial Old）。









##### 垃圾回收器：Parallel Scavenge 与 Parallel Old

###### 新生代：Parallel Scavenge（PS）

> **Parallel Scavenge是JDK8默认的年轻代垃圾回收器**。
>
> **多线程并行回收**，关注的是系统的吞吐量。具备**自动调整堆内存大小**的特点，采用的是**复制算法**。
>
> - 自动调整堆内存：即不需要手动指定新生代的大小(`-Xmn`)、Eden 和 Survivor 区的比例（`-XX:SurvivorRatio` ）、晋升老年代对象年龄等细节参数了。虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量。
>
> - 可以使用参数 `-XX:+UseAdaptiveSizePolicy` 让垃圾回收器根据吞吐量和最大停顿毫秒数自动调整内存大小。
>
> 开启方式：`-XX:+UseParallelGC` 或 `-XX:+UseParallelOldGC` 就可以使用Parallel Scavenge + Parallel Old这种组合。 
>
> - Oracle官方建议在使用这个组合时，**不要设置堆内存的最大值**，垃圾回收器会根据最大暂停时间和吞吐量自动调整内存大小。
>
> 垃圾回收线程进行GC时，会让用户线程进入等待，GC执行完了才会进行用户线程。
>
> **适用场景**：后台任务，不需要与用户交互，并且容易产生大量的对象。如大数据的处理，大文件导出。



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212225111952-1559169926.png" alt="image-20231212225110859" />



**优点**：吞吐量高，而且手动可控。为了提高吞吐量，虚拟机会动态调整堆的参数。

- 可以使用参数 `-XX:GCTimeRatio=n` 设置吞吐量为n。

```txt
用户线程执行时间 = n / (n + 1)
```

**缺点**：不能保证单次的停顿时间。

- 可以使用参数 `-XX:MaxGCPauseMillis=n` 设置每次垃圾回收时的最大停顿毫秒数。



> 证明JDK8默认采用了Parallel Scavenge垃圾回收器

1. CMD中输入下列参数即可

```bash
# 打印出启动过程中使用的所有虚拟机参数
C:\Users\zixq\Desktop> java -XX:+PrintCommandLineFlags -version

打印出的关键结果：

-XX:InitialHeapSize=264767296
-XX:MaxHeapSize=4236276736
-XX:+PrintCommandLineFlags
-XX:+UseCompressedClassPointers 
-XX:+UseCompressedOops 
-XX:-UseLargePagesIndividualAllocation 
-XX:+UseParallelGC		# 此处：已经加了使用 Parallel Scavenge 垃圾回收器的虚拟机参数
```

2. 也可在Arthas中查看：随便写段代码，使用 `System.in.read();` 定住，不添加任何虚拟机参数启动。示例如下：

```java
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * <p>
 * 测试JDK8的垃圾回收器（GC）是什么
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

public class JDK8_GC_Test {

    public static void main(String[] args) throws IOException {
        List<Object> list = new ArrayList<>();

        int count = 0;
        while (true) {
            System.in.read();
            System.out.println("++count = " + ++count);
            // 1M
            list.add(new byte[1024 * 1024]);
        }
    }
}
```

Arthas命令：

```bash
dashboard -n 1
```

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212233212383-537452935.png" alt="image-20231212233209157" />





###### 老年代：Parallel Old（PO）

> Parallel Old是为Parallel Scavenge收集器设计的老年代版本，JDK8默认的老年代垃圾回收器。
>
> 利用**多线程并发收集**，采用**标记-整理算法**。
>
> 开启方式：`-XX:+UseParallelGC` 或 `-XX:+UseParallelOldGC` 就可以使用Parallel Scavenge + Parallel Old这种组合。 
>
> 垃圾回收线程进行GC时，会让用户线程进入等待，GC执行完了才会进行用户线程。





<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212233406186-1725335491.png" alt="image-20231212233405285" />



**优点**：并发收集，在多核CPU下效率较高。

**缺点**：暂停时间会比较长。





##### 垃圾回收器：G1

> G1（Garbage First）是在Java7 update 4之后引入的一个新的垃圾回收器，引入分区的思路，弱化了分代的概念。
>
> JDK9之后默认的垃圾回收器是G1垃圾回收器。
>
> 堆被分为新生代和老年代，其它收集器进行收集的范围都是整个新生代或老年代，而 G1 可以**直接对新生代和老年代一起回收（Mixed GC）**，采用**标记-复制算法**。
>
> 开启方式：使用参数 `-XX:+UseG1GC` 
>
> 适用场景：JDK8最新版本、JDK9之后建议默认使用。



Parallel Scavenge关注吞吐量，允许用户设置最大暂停时间 ，但是会减少年轻代可用空间的大小。

CMS关注暂停时间，但是吞吐量方面会下降。

而G1设计目标就是将上述两种垃圾回收器的优点融合：

1. 支持巨大的堆空间回收，并有较高的吞吐量。
2. 支持多CPU并行垃圾回收。
3. 允许用户设置最大暂停时间。



优点：对比较大的堆如超过6G的堆回收时，延迟可控不会产生内存碎片，并发标记的SATB算法效率高.

缺点：JDK8之前还不够成熟。





###### G1的内存划分

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213161803495-1937292118.png" alt="image-20231213161802465" />

G1是将整个堆会被划分成多个大小相等的区域，称之为Region（默认将整堆划分为2048个分区），区域不要求是物理连续的（逻辑连续即可）。分为Eden、Survivor、Old区。

> 问题：每个Region的大小是怎么计算来的？

1. 虚拟机自行计算的方式

```txt
每个Region的大小 = 堆空间大小 / 2048m


如：堆空间大小 = 4G，则每个Region的大小 = （1024 * 4）/ 2048 = 2m
```

2. 程序员手动配置的方式

```bash
可通过参数 -XX:G1HeapRegionSize=32m 指定

	其中32m指定每个region大小为32M，Region size必须是2的指数幂，取值范围从1M到32M
```



> 分区，即分Region，还会牵扯到一个小知识点：本地分配缓冲 Lab（Local allocation buffer）

由于分区的思想，每个线程均可以"认领"某个分区用于线程本地的内存分配，而不需要顾及分区是否连续。因此，每个应用线程和GC线程都会独立地使用分区，进而减少同步时间，提升GC效率，这个分区称为**本地分配缓冲区(Lab)**。其中：

1. 应用线程可以独占一个本地缓冲区（TLAB，详情见前面的 [GC算法：分代收集算法](#GC算法：分代收集算法) ） 来创建对象，而大部分都会落入Eden区域（巨型对象或分配失败除外），因此TLAB的分区属于Eden空间。
2. 而每次垃圾收集时，每个GC线程同样可以独占一个本地缓冲区(GCLAB)用来转移对象，每次回收会将对象复制到Suvivor空间或老年代空间；对于从Eden / Survivor空间晋升（Promotion）到Survivor / 老年代空间的对象，同样由GC独占的本地缓冲区进行操作，该部分称为**晋升本地缓冲区(PLAB)**





###### G1垃圾回收的方式

G1垃圾回收有两种方式：

1. 年轻代回收：Young GC（Minor GC）

- Young GC：回收Eden区和Survivor区中不用的对象。会导致STW。采用复制算法。
- G1中可以通过参数 `-XX:MaxGCPauseMillis=n` （默认200ms） 设置每次垃圾回收时的最大暂停时间毫秒数，G1垃圾回收器会“尽可能地”保证暂停时间（即软实时：尽可能在此时限内完成垃圾回收）。



2. 混合回收（年轻代+老年代）：Mixed GC。

- Mixed GC：回收所有年轻代和部分老年代的对象以及大对象区。采用标记-复制整理算法



###### 年轻代回收：Young GC执行流程

> 下列流程也有个专业名字：年轻代收集集合 CSet of Young Collection

1. **新创建的对象会存放在Eden区。当G1判断年轻代区不足（max默认60%），无法分配对象时需要回收时会执行Young GC。** 

> max指的是：年轻代内存超过整个堆内存的60%。

**注意**：部分对象如果大小达到甚至超过Region的一半，会直接放入老年代，这类老年代被称为Humongous【巨型】区。

如堆内存是4G，那么每个Region是2M，而只要一个大对象超过了1M就被放入Humongous区，如果对象过大会横跨多个Region。

G1内部做了一个优化，一旦发现没有引用指向巨型对象，则可直接在年轻代收集周期中被回收。

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213165452814-586753045.png" alt="image-20231213165452618" />

从上图也可知：巨型对象会独占一个、或多个连续分区，其中

- 第一个分区被标记为开始巨型(StartsHumongous)，相邻连续分区被标记为连续巨型(ContinuesHumongous)。
- 由于无法享受Lab带来的优化，并且确定一片连续的内存空间需要整堆扫描，因此确定巨型对象开始位置的成本非常高，如果可以，应用程序应避免生成巨型对象。



2. **标记出Eden和Survivor区域中的存活对象。**
3. **根据配置的最大暂停时间选择“某些Region区域”，将这些区域中存活对象复制到一个新的Survivor区中（对象年龄【存活次数】+1），然后清空这些区域。**

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213214316495-2110152344.png" alt="image-20231213171509169" />

> 这里的小细节是：Eden分区存活的对象将被拷贝到Survivor分区；原有Survivor分区存活的对象，将根据任期阈值(tenuring threshold)分别晋升到PLAB、新的survivor分区和老年代分区中，而原有的年轻代分区将被整体回收掉。



> 问题1：所谓的“某些Region区域“是怎么选择的？

- G1在进行Young GC的过程中会去记录每次垃圾回收时每个Eden区和Survivor区的平均耗时，以作为下次回收时的参考依据。

- 然后根据配置的最大暂停时间【`-XX:MaxGCPauseMillis=n` （默认200ms）】就能计算出本次回收时最多能回收多少个Region区域。

如：`-XX:MaxGCPauseMillis=n`（默认200ms），每个Region回收耗时40ms，那么这次回收最多只能回收200 / 40 = 5，但选5个可能会超出设置的最大暂停时间，所以只选择4个Region进行回收。



> 问题2：此步要维护对象年龄（年龄+1）的原因是什么？
>
> - 答案：辅助判断老化(tenuring)对象晋升时是到Survivor分区还是到老年代分区。

- 年轻代收集首先先将晋升对象内存大小总和、对象年龄信息维护到年龄表中。
- 再根据年龄表、Survivor内存大小、Survivor填充容量 `-XX:TargetSurvivorRatio` (默认50%)、最大任期阈值`-XX:MaxTenuringThreshold` (默认15)，计算出一个恰当的任期阈值，凡是超过任期阈值的对象都会被晋升到老年代。



4. **后续Young GC时与之前相同，只不过Survivor区中存活对象会被搬运到另一个Survivor区。**当某个存活对象的年龄到达阈值（默认15），将被放入老年代。



> 关于上述年轻代回收有一个小知识点：收集集合（CSet）
>
> - 收集集合(CSet)：代表每次GC暂停时回收的一系列目标分区。

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213213240528-1259651546.jpg" alt="img" />



在任意一次收集暂停中，CSet所有分区都会被释放，内部存活的对象都会被转移到分配的空闲分区中。

因此无论是年轻代收集，还是混合收集，工作的机制都是一致的。年轻代收集CSet只容纳年轻代分区，而混合收集会通过启发式算法，在老年代候选回收分区中，筛选出回收收益最高的分区添加到CSet中。

> 候选老年代分区的CSet准入条件：可以通过活跃度阈值 `-XX:G1MixedGCLiveThresholdPercent` (默认85%)进行设置，从而拦截那些回收开销巨大的对象；同时，每次混合收集可以包含候选老年代分区，可根据CSet对堆的总大小占比 `-XX:G1OldCSetRegionThresholdPercent` (默认10%)设置数量上限。

由上述可知，G1的收集都是根据CSet进行操作的，年轻代收集与混合收集没有明显的不同，最大的区别在于两种收集的触发条件。





###### 卡表（Card Table） 与 已记忆集合（RSet）

> 问题：Region内部又有什么？Card与RSet
>
> - 这个问题就是想问：G1的年轻代回收的原理（卡片Card + 卡表 Crad Table + 已记忆集合 RSet ）。
>
> G1年轻代回收（Young GC）的执行流程请看：[年轻代回收Young GC执行流程](#年轻代回收Young GC执行流程) 

在前面G1年轻代回收（Young GC）时需要标记“存活对象”，请问：如果产生了跨代引用，我们怎么知道它算不算存活对象？如老年代对象引用了年轻代对象。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126211503882-1260837965.png" alt="image-20240126211516748" />

方式一：从GC Root开始，扫描所有对象，如果年轻代对象在引用链上，就标记为存活。这种需要遍历引用链上的所有对象，效率太低，所以不可取。

方式二：维护一个详细的表，记录哪个对象被哪个老年代引用了。在年轻代中被引用的对象，不进行回收。此方式看似可行，但如果对象太多这张表会占用很大的内存空间。存在错标的情况。可相对来说比方式一要靠谱一点。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240129152356200-628575224.png" alt="image-20240126212047426" />

所以，对方式二进行优化：

1. 优化一：只记录Region被哪些对象引用了。这种引用详情表称为**记忆集 RememberedSet（简称RS或RSet）：是一种记录了从非收集区域对象 引用 收集区域对象的这些关系**的数据结构。扫描时将记忆集中的对象也加入到GC Root中，就可以根据引用链判断哪些对象需要回收了。

这样通过使用RSet，就在做可达性分析时就可以避免全堆扫描。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126212746408-601645056.png" alt="image-20240126212802913" />



虽然引入了RSet，但是还有存在一个问题：如果区域中引用对象很多，还是占用很多内存。因此继续优化。



2. 优化二：将所有区域（Region）中的内存按一定大小划分成很多个块，每个块进行编号（即卡片【Card 或 Card Page】）。**记忆集(RSet)中只记录对块（Card）的引用关系（现在才是RSet真正记录的东西）**。如果一个块中有多个对象，只需要引用一次，减少了内存开销。

在串行和并行收集器中，GC通过整堆扫描，来确定对象是否处于可达路径中。

然而G1为了避免STW式的整堆扫描，在每个分区记录了一个已记忆集合(RSet)，内部类似一个反向指针，记录引用分区内对象的卡片索引（见下述Card与Card Table）。当要回收该分区时，通过扫描分区的RSet，来确定引用本分区内的对象是否存活，进而确定本分区内的对象存活情况。

> 这个RSet有个注意点：并非所有的引用都需要记录在RSet中，只有老年代的分区可能会有RSet记录，这些分区称为**拥有RSet分区(an RSet’s owning region)**

- 如果一个分区确定需要扫描，那么无需RSet也可以无遗漏地得到引用关系，那么引用源自本分区的对象，当然不用落入RSet中；

- G1 GC每次都会对年轻代进行整体收集，因此引用源自年轻代的对象，也不需要在RSet中记录。



> G1对内存的使用以分区(Region)为单位，而对对象的分配则以卡片(Card)为单位。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126214003903-2074758626.png" alt="image-20240126214020277" />

> 每个分区（Region）内部被分成了若干个大小为**512 Byte的卡片(Card 或 Card Page)**，标识堆内存最小可用粒度。

所有分区的卡片将会记录在全局卡片表（Global Card Table，是一个字节数组）中，分配的对象会占用物理上连续的若干个卡片，当查找对分区内对象的引用时便可通过记录卡片来查找该引用对象。

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126225458654-716061640.jpg)

**每次对内存的回收，本质都是对指定分区的卡片进行处理**。

> 每个分区（Region）都拥有一个自己的卡表（Card Table），如果产生了跨代引用（老年代引用年轻代），此时这个Region对应的卡表上就会将字节内容进行修改。
>
> - 上述的全局卡片表(Global Card Table)这个字节数组的一个元素就是这里的某个Region的卡表。
> - 卡表会占用一定的内存空间，如：堆大小是1G时，卡表大小为 1024 MB / 512 = 2MB。
> - 卡表的主要作用是生成记忆集（RSet）。

JDK8源码中0代表被引用了称为脏卡。这样就可以标记出当前Region被老年代中的哪些部分引用了。那么要**生成记忆集就比较简单了，只需要遍历整个卡表，找到所有脏卡。**

0代码脏卡，其他数字代表意思如下：

![image-20240126220459277](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126220442509-1435370812.png)

产生跨代引用（老年代引用年轻代），Region对应的卡表上就会将字节内容进行修改：

![image-20240126225909633](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126225852949-1588912682.png)



年轻代回收标记时，会将记忆集中的对象也加入到GC Root对象中，进行扫描并标记其引用链上的对象。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126221219943-573110583.png" alt="image-20240126221236071" />



> 问题：RSet的内部又有什么？有个你先人板板包批龙，人都问麻了>_>
>
> - 答案：PRT（ Per Region Table ）

RSet在内部使用Per Region Table（PRT）记录分区的引用情况。由于RSet的记录要占用分区的空间，如果一个分区非常"受欢迎"，那么RSet占用的空间会上升，从而降低分区的可用空间。G1应对这个问题采用了改变RSet的密度的方式，在PRT中将会以三种模式记录引用：

- 稀少：直接记录引用对象的卡片索引。
- 细粒度：记录引用对象的分区索引。
- 粗粒度：只记录引用情况，每个分区对应一个比特位。

由上可知，粗粒度的PRT只是记录了引用数量，需要通过整堆扫描才能找出所有引用，因此扫描速度也是最慢的。





###### RSet的维护：写屏障 与 并发优化线程

> 本节内容参考自：[@pdai：Java垃圾回收器 - G1详解](https://www.pdai.tech/md/java/jvm/java-jvm-gc-g1.html)



> 由于不能整堆扫描，又需要计算分区确切的活跃度，因此，G1需要一个增量式的完全标记并发算法，通过维护RSet，得到准确的分区引用信息。在G1中，RSet的维护主要来源两个方面：写屏障(Write Barrier) 和 并发优化线程(Concurrence Refinement Threads)。

首先介绍一下栅栏(Barrier)的概念。**栅栏是指在原生代码片段中，当某些语句被执行时，栅栏代码也会被执行。**

> G1主要在赋值语句中，使用写前屏障(Pre-Write Barrrier)和写后屏障(Post-Write Barrrier)。事实上，写屏障的指令序列开销非常昂贵，应用吞吐量也会根据栅栏复杂度而降低。

栅栏代码示意：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127133014932-1259183453.jpg)



> 写屏障（Write Barrier）：在执行引用关系建立的代码执行后插入一段指令，完成卡表的维护工作。损失的性能一般大约在5%~10%之间。

- 每次引用类型数据写操作时，都会产生一个Write Barrier【写屏障/写栅栏】暂时中断操作；
- 然后检查将要写入的引用指向的对象是否和该引用类型数据在不同的Region（其他收集器：检查老年代对象是否引用了新生代对象）；
  - 如果不同，通过cardTable把相关引用信息记录到引用指向对象的所在Region对应的Remembered Set中；
  - 当进行垃圾收集时，在GC根节点的枚举范围加入Remembered Set；就可以保证不进行全局扫描，也不会有遗漏。



> 写前屏障（Pre-Write Barrrier）

即将执行一段赋值语句时，等式左侧对象将修改引用到另一个对象，那么等式左侧对象原先引用的对象所在分区将因此丧失一个引用，那么JVM就需要在赋值语句生效之前，记录丧失引用的对象。JVM并不会立即维护RSet，而是通过批量处理，在将来RSet更新(见下述“起始快照算法（SATB）”)。

> 写后屏障（Post-Write Barrrier）：G1使用写后屏障技术，在执行引用关系建立的代码执行后插入一段指令，完成卡表的维护工作。

当执行一段赋值语句后，等式右侧对象获取了左侧对象的引用，那么等式右侧对象所在分区的RSet也应该得到更新。同样为了降低开销，写后栅栏发生后，RSet也不会立即更新，同样只是记录此次更新日志，在将来批量处理(见下述“并发优化线程（Concurrence Refinement Threads）“）。



> 起始快照算法 Snapshot at the beginning (SATB)：一句概括 生成对象图，变更则记录STAB日志或缓冲区，并发标记时扫描引用更新RSet
>
> - Taiichi Tuasa贡献的增量式完全并发标记算法。
> - 主要针对标记-清除垃圾收集器的并发标记阶段，非常适合G1的分区块的堆结构，同时解决了CMS的主要烦恼：重新标记暂停时间长带来的潜在风险。

- SATB会创建一个对象图，相当于堆的逻辑快照，从而确保并发标记阶段所有的垃圾对象都能通过快照被鉴别出来。

- 当赋值语句发生时，应用将会改变了它的对象图，那么JVM需要记录被覆盖的对象。因此写前栅栏会在引用变更前，将值记录在SATB日志或缓冲区中。每个线程都会独占一个SATB缓冲区，初始有256条记录空间。
- 当空间用尽时，线程会分配新的SATB缓冲区继续使用，而原有的缓冲区则加入全局列表中。
- 最终在并发标记阶段，并发标记线程（Concurrent Marking Threads）在标记的同时，还会定期检查和处理全局缓冲区列表的记录，然后根据标记位图分片的标记位，扫描引用字段来更新RSet。此过程又称为并发标记 / SATB写前栅栏。



> 并发标记线程（Concurrent Marking Threads）

并发标记位图过程：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127135900266-1941287621.png)

> 要标记存活的对象，每个分区都需要创建位图(Bitmap)信息来存储标记数据，来确定标记周期内被分配的对象。
>
> - G1采用了两个位图Previous Bitmap、Next Bitmap 来存储标记数据。

Previous位图存储上次的标记数据，Next位图在标记周期内不断变化更新，同时Previous位图的标记数据也越来越过时，

当标记周期结束后Next位图便替换Previous位图，成为上次标记的位图。同时，每个分区通过顶部开始标记(TAMS)来记录已标记过的内存范围。同样的，G1使用了两个顶部开始标记Previous TAMS(PTAMS)、Next TAMS(NTAMS)记录已标记的范围。

在并发标记阶段，G1会根据参数`-XX:ConcGCThreads`(默认GC线程数的1/4，即`-XX:ParallelGCThreads/4`)分配并发标记线程(Concurrent Marking Threads)，进行标记活动。每个并发线程一次只扫描一个分区，并通过"手指"指针的方式优化获取分区。并发标记线程是爆发式的，在给定的时间段拼命干活，然后休息一段时间，再拼命干活。

每个并发标记周期，在初始标记STW的最后，G1会分配一个空的Next位图和一个指向分区顶部(Top)的NTAMS标记。Previous位图记录的上次标记数据上次的标记位置，即PTAMS，在PTAMS与分区底部(Bottom)的范围内，所有的存活对象都已被标记。那么，在PTAMS与Top之间的对象都将是隐式存活(Implicitly Live)对象。在并发标记阶段，Next位图吸收了Previous位图的标记数据，同时每个分区都会有新的对象分配，则Top与NTAMS分离，前往更高的地址空间。在并发标记的一次标记中，并发标记线程将找出NTAMS与PTAMS之间的所有存活对象，将标记数据存储在Next位图中。同时，在NTAMS与Top之间的对象即成为已标记对象。如此不断地更新Next位图信息，并在清除阶段与Previous位图交换角色。



> 并发优化线程（Concurrence Refinement Threads）：

G1中使用基于Urs Hölzle的快速写栅栏，将栅栏开销缩减到2个额外的指令。栅栏将会更新一个card table type的结构来跟踪代间引用。

当赋值语句发生后，写后栅栏会先通过G1的过滤技术判断是否是跨分区的引用更新，并将跨分区更新对象的卡片加入缓冲区序列，即更新日志缓冲区或脏卡片队列。与SATB类似，一旦日志缓冲区用尽，则分配一个新的日志缓冲区，并将原来的缓冲区加入全局列表中。

并发优化线程(Concurrence Refinement Threads)，只专注扫描日志缓冲区记录的卡片来维护更新RSet，线程最大数目可通过`-XX:G1ConcRefinementThreads`(默认等于`-XX:ParellelGCThreads`)设置。并发优化线程永远是活跃的，一旦发现全局列表有记录存在，就开始并发处理。如果记录增长很快或者来不及处理，那么通过阈值`-X:G1ConcRefinementGreenZone/-XX:G1ConcRefinementYellowZone/-XX:G1ConcRefinementRedZone`，G1会用分层的方式调度，使更多的线程处理全局列表。如果并发优化线程也不能跟上缓冲区数量，则Mutator线程(Java应用线程)会挂起应用并被加进来帮助处理，直到全部处理完。因此，必须避免此类场景出现。





> 更详细地G1年轻代回收的过程

通过前面的铺垫之后，现更详细地在G1年轻代的回收过程如下：

1. Root扫描，将所有的静态变量、局部变量扫描出来。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127142421479-975717425.png" alt="image-20240127142438781" />

2. 处理脏卡队列中的没有处理完的信息，更新记忆集的数据，此阶段完成后，记忆集中包含了所有老年代对当前Region的引用关系。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127142528660-1280114003.png" alt="image-20240127142545962" />

3. 标记存活对象。记忆集中的对象会加入到GC Root对象集合中，在GC Root引用链上的对象也会被标记为存活对象。
4. 根据设定的最大停顿时间，选择本次收集的区域，称之为回收集合（Collection Set）。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127142705568-14811188.png" alt="image-20240127142723038" />

5. 复制对象：将标记出来的对象复制到新的区中，将年龄+1，如果年龄到达15则晋升到老年代。老的区域内存直接清空。
6. 处理软、弱、虚、终结器引用，以及JNI中的弱引用。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127142827310-460433325.png" alt="image-20240127142844824" />









###### 混合收集：Mixed GC

> 回收所有年轻代和部分老年代的对象以及大对象区。采用的算法：标记-复制整理。
>
> **触发时机**：经过多次的回收之后(上述Young GC执行流程)，会出现很多老年代区（Old），此时总堆占有率达到阈值时（`-XX:InitiatingHeapOccupancyPercent` 默认45%）会触发混合回收Mixed GC。

为了满足暂停目标，G1可能不能一口气将所有的候选分区收集掉，因此G1可能会产生连续多次的混合收集与应用线程交替执行，每次STW的混合收集与年轻代收集过程相类似。

为了确定包含到年轻代收集集合CSet的老年代分区，JVM通过参数混合周期的最大总次数`-XX:G1MixedGCCountTarget`(默认8)、堆废物百分比-`XX:G1HeapWastePercent`(默认5%)。通过候选老年代分区总数与混合周期最大总次数，确定每次包含到CSet的最小分区数量；根据堆废物百分比，当收集达到参数时，不再启动新的混合收集。而每次添加到CSet的分区，则通过计算得到的GC效率进行安排。



**Mixed GC的回收流程**：

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213170847310-294288465.png" alt="image-20231213170846871" />



1. **初始标记（initial mark）：暂停所有用户线程（停段时间很短）。标记Gc Roots引用的对象为存活**。
2. **并发标记（concurrent mark）：会和用户线程一起执行。将第一步中标记的对象引用的对象，标记为存活**。这里和Region维护的Remebered Set挂钩（请看：[卡表（Card Table） 与 已记忆集合（RSet）](#卡表（Card Table） 与 已记忆集合（RSet）)）。

> CMS和G1在并发标记时使用的是同一个算法：三色标记法
>
> 使用黑灰白三种颜色标记对象。
>
> - 黑色（存活）：当前对象在GC Root引用链上，其自身与其引用对象都已标记完成；
> - 灰色（待处理）：当前对象在GC Root引用链上，其自身被标记，其引用的对象未标记；
> - 白色（可回收）：当前对象不在GC ROOT引用链上，是未标记。



![img](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213220758112-1592549812.gif)



**执行流程如下：**

- GC 开始前所有对象都是白色。
- GC 一开始，所有根（GC Root）能够直达的对象被压到栈中，待搜索，此时颜色是灰色（即：放入灰色队列）。
- 然后灰色对象依次从栈中取出搜索子对象，子对象也会被涂为灰色，入栈。当其所有的子对象都涂为灰色之后，该对象被涂为黑色。
- 当 GC 结束之后，灰色对象将全部没了（即对象标记处理完成），**剩下黑色的为存活对象**，白色的为垃圾。



上面的三色标记法的黑色、白色就是使用了位图（BitMap）来实现的：

如：8个字节使用1个bit来标识标记的内容，黑色为1，白色为0，灰色不会体现在位图中，会单独放入一个队列中。如果对象超过8个字节，仅仅使用第一个bit位处理。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127151107339-1280015919.png" alt="image-20240127151124362" />



3. **最终标记（remark或者Finalize Marking）：暂停所有用户线程。标记一些引用改变而漏标的对象**。

> 和CMS的区别在这步：Mixed GC在这里不管新创建 和 不再关联的对象（等待下一轮回收该回收的）。

**小细节**：为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程的 Remembered Set Logs 里，最终标记阶段需要把 Remembered Set Logs 的数据合并到 Remembered Set 中。

> 问题：漏标问题是怎么产生的？

三色标记法存在一个很严重的问题：由于用户线程可能同时在修改对象的引用关系，就会出现漏标的情况。

如：用户线程执行了 B.c = null；将B到C的引用去除了。同时执行了A.c = c; 添加了A到C的引用。这就产生了严重问题。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127152106425-734763295.png" alt="image-20240127152123800" />

换言之就是：在最终标记（remark或者Finalize Marking）过程中，黑色指向了白色，如果不对黑色重新扫描，则会漏标。会把白色对象当作没有新引用指向从而回收掉。

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213222630479-1600594513.png" alt="img" />

并发标记过程中，Mutator删除了所有从灰色到白色的引用，会产生漏标。此时白色对象应该被回收。

产生漏标问题的条件有两个：

- 黑色对象指向了白色对象。
- 灰色对象指向白色对象的引用消失。

所以要解决上述三色标记法带来的漏标问题，打破两个条件之一即可：

- **跟踪黑指向白的增加** incremental update：增量更新，关注引用的增加。把黑色重新标记为灰色，下次重新扫描属性。CMS采用该方法。
- **记录灰指向白的消失** SATB（snapshot at the beginning 初始快照算法）：关注引用的删除。当灰–>白消失时，要把这个 引用 推到GC的栈中，保证白还能被GC扫描到。G1采用该方法。



SATB（snapshot at the beginning 起始快照算法）：

- 标记开始时创建一个快照，记录当前所有对象，标记过程中新生成的对象直接标记为黑色。
- 采用写前屏障（Post-Write-Barrier）技术，在引用赋值前（如B.c = null之前），将之前引用的对象c放入SATB待处理队列中。SATB队列每个线程都有一个，最终标记阶段会汇总到一个总的SATB队列中，然后逐一处理。

> SATB队列中的对象，默认按照存活处理，同时要处理他们引用的对象。
>
> SATB的缺点：是在本轮清理时可能会将不存活的对象标记成存活对象，产生了一些所谓的浮动垃圾，等到下一轮清理时才能回收.

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127153339227-403686653.png" alt="image-20240127153356491" />



> 问题：为什么G1采用SATB而不用incremental update？

因为采用incremental update把黑色重新标记为灰色后，之前扫描过的还要再扫描一遍，效率太低。G1有RSet与SATB相配合。Card Table里记录了RSet，RSet里记录了其他对象指向自己的引用，这样就不需要再扫描其他区域，只要扫描RSet就可以了。

也就是说 灰色–>白色 引用消失时，如果没有 黑色–>白色，引用会被push到堆栈，下次扫描时拿到这个引用，由于有RSet的存在，不需要扫描整个堆去查找指向白色的引用，效率比较高，SATB配合RSet浑然天成。



4. **并发清理（cleanup）：与用户线程一起执行，STW很长。将存活对象复制到别的Region**。使用复制算法的目的是为了不产生内存碎片。



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213172021793-1104993078.png" alt="image-20231213172021831" style="zoom: 67%;" />

- 根据最终标记的结果，可以计算出每一个区域的垃圾对象占用内存大小，根据停顿时间，选择复制效率最高（垃圾对象最多）的几个区域。
- 复制时先复制GC Root直接引用的对象，然后再复制其他对象。

> G1对老年代的清理：会选择存活度最低的区域来进行回收，这样可以保证回收效率最高，这也是G1（Garbage first）名称的由来：G1的1（first）指的就是存活度最低的区域。
>

- 回收老的区域，如果外部有其他区域对象引用了转移对象，也需要重新设置引用关系

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127154448656-1529057910.png" alt="image-20240127154505904" />





> 顺便再提一下和这里相关的：Full GC 整堆收集，是单线程+标记-整理算法，会导致用户线程的暂停。
>
> 建议：尽量保证应用的堆内存有一定多余的空间。

**触发时机**：

1. 如果上面的清理过程中发现没有足够的空Region存放转移的对象（大对象、长期存活的对象进入老年代，导致老年代空间不足），会出现Full GC。

避免这种情况引起Full GC的方式：

一是：尽量不要创建过大的对象以及数组。

二是：通过参数 `-Xmn` 调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。

- PS：但得注意，这个参数的修改需要经过大量测试（所有接口、所有场景的测试），因为在实际场景中，接口响应时间、创建对象的大小、程序内部的定时任务等等这些不确定因素都会影响该值，所以理论计算并不能准确地得到该值，若要变动该值则需要进行大量测试【G1垃圾回收器中就尽量不要设置该值，因G1会动态调整年轻代大小】。

三是：通过参数 `-XX:MaxTenuringThreshold` 调大对象进入老年代的年龄，让对象在新生代多存活一段时间。

- PS：此值即最大晋升阈值，默认值为15。对象年龄大于此值之后会进入老年代。在JVM中有一个“动态年龄判断机制”，将从小到大的对象占据的空间加起来，若大于survivor区域的50%，则把等于或大于该年龄的对象丢入到老年代。因此：此值个人建议别轻易去修改。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240111210921546-1601564586.png" alt="image-20240111210925222" />

> 此处相关的小知识：Concurrent Mode Failure 并发模式失败
>
> - 执行 CMS GC 的过程中同时有对象要放入老年代（CMS的垃圾清理线程和用户线程并行执行），而此时老年代空间不足(可能是 GC 过程中浮动垃圾过多导致暂时性的空间不足)，便会报 Concurrent Mode Failure 错误，并触发 Full GC。
> - 此模式一旦产生，那么Java虚拟机就会使用Serial Old单线程进行Full GC回收老年代，从而出现长时间停顿（STW）。

2. 调用 `System.gc()`。不建议的方式。因为调用此方法只是建议虚拟机执行 Full GC，但是虚拟机不一定会真正去执行。
3. 使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC。

**内存空间分配担保**：在发生 Minor GC 之前，虚拟机先检查老年代最大可用的连续空间是否大于新生代所有对象总空间。

- 如果条件成立的话，那么 Minor GC 可以确认是安全的。

- 如果不成立的话：

JDK 6 Update 24之前规则：虚拟机会查看 `-XX:HandlePromotionFailure=true/false`设置值是否允许担保失败

- 如果允许，那么就会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小

  - 如果大于，将尝试着进行一次 Minor GC；

  - 如果小于，或者 HandlePromotionFailure 设置不允许冒险，那么就要进行一次 Full GC。

**JDK 6 Update 24之后的规则**：`-XX:HandlePromotionFailure=true/false`配置不会再影响到虚拟机的空间分配担保策略。所以此时是只要老年代的连续空间大于新生代对象空间总大小 或者 历次晋升的平均大小，就会进行 Minor GC，否则将进行 Full GC。







##### 关于Minor GC、Major GC、Full GC的说明

> 尽量别让Major GC / Full GC触发。

JVM 在进行 GC 时，并非每次都对堆内存（新生代、老年代；方法区）区域一起回收的，大部分时候回收的都是指新生代。

针对 HotSpot VM 的实现，它里面的 GC 按照回收区域又分为两大类：部分收集（Partial GC），整堆收集（Full GC）。

1. 部分收集（Partial GC）：不是完整收集整个 Java 堆的垃圾收集其中又分为： 

- 新生代收集（Minor GC/Young GC）：只是新生代的垃圾收集。
- 老年代收集（Major GC/Old GC）：只是老年代的垃圾收集。
  - 目前，只有 CMS GC 会有单独收集老年代的行为。
  - 很多时候 Major GC 会和 Full GC 混合使用，需要具体分辨是老年代回收还是整堆回收。

- 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集。
  - 目前只有 G1 GC 会有这种行为。

2. 整堆收集（Full GC）：收集整个 Java 堆和方法区的垃圾。



> 年轻代GC（Minor GC）触发机制

- 当年轻代空间不足时，就会触发MinorGC，这里的年轻代满指的是Eden代满，Survivor满不会引发GC。（每次Minor GC会清理年轻代的内存）
- 因为Java对象大多都具备朝生夕灭的特性.，所以Minor GC非常频繁，一般回收速度也比较快。这一定义既清晰又易于理解。
- Minor GC会引发STW，暂停其它用户的线程，等垃圾回收结束，用户线程才恢复运行。



> 老年代GC（Major GC / Full GC）触发机制 

- 指发生在老年代的GC，对象从老年代消失时，我们说 “Major GC” 或 “Full GC” 发生了。

> 出现了Major Gc，经常会伴随至少一次的Minor GC（非绝对，在Paralle1 Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程）。
>
> ==也就是在老年代空间不足时，会先尝试触发Minor Gc。如果之后空间还不足，则触发Full GC（Major GC）==。

- Major GC的速度一般会比Minor GC慢10倍以上，STW的时间更长。
- 如果Major GC后，内存还不足，就报OOM了。



>  Full GC触发机制：Full GC 是开发或调优中尽量要避免的。这样暂停时间会短一些。因为Full GC是单线程+标记-整理，单线程会STW会长。

触发Full GC执行的情况有如下几种：

1. 调用`System.gc()`时，系统建议执行Full GC，但是不必然执行。
2. 老年代空间不足。前面G1说的触发时机基本都可统称为这个原因。

- 补充一种情况：由Eden区、survivor space0（From Space）区向survivor space1（To Space）区复制时，对象大小 大于 To Space可用内存，则把该对象转存到老年代，且老年代的可用内存 小于 该对象大小。

3. 方法区空间不足。





### 即时编译器（JIT）

> 本节内容可以直接去看 [@美团技术团队：Java即时编译器原理解析及实践](https://tech.meituan.com/2020/10/22/java-jit-practice-in-meituan.html) 的内容，里面的内容更丰富、更全面，看完都不需要看我这里的内容了，因我这里的内容也是在此基础上调整的。
>
> 作者：昊天 珩智 薛超



> 在Java中，JIT（Just-In-Time）即时编译器是一项用来提升应用程序代码执行效率的技术。字节码指令被 Java 虚拟机解释执行，如果有一些指令执行频率高，称之为热点代码，这些字节码指令则被JIT即时编译器编译成机器码同时进行一些优化，最后保存在内存中，将来执行时直接读取就可以运行在计算机硬件上了

在HotSpot中，有三款即时编译器，C1、C2和Graal。

> 可使用Java虚拟机参数：`-Xin` 关闭即时编译器。

**C1编译效率比C2快，但是优化效果不如C2。所以C1适合优化一些执行时间较短的代码，C2适合优化服务端程序中长期执行的代码**

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126181132447-1349497379.png" alt="image-20240126181148496" />

JDK7之后，采用了分层编译的方式，在JVM中C1和C2会一同发挥作用，分层编译将整个优化级别分成了5个等级：

> `-XX:TieredStopAtLevel=1`  分层编译下只使用1层C1进行编译

下列保存内容的原因：为了执行C2即时编译器的需要，从而让C2更好地优化。

| 等级 | 使用的组件   | 描述                                         | 保存的内容                                                   | 性能打分（1 - 5） |
| ---- | ------------ | -------------------------------------------- | ------------------------------------------------------------ | ----------------- |
| 0    | 解释器       | 解释执行；<br />记录方法调用次数及循环次数   | 无                                                           | 1                 |
| 1    | C1即时编译器 | C1完整优化                                   | 优化后的机器码                                               | 4                 |
| 2    | C1即时编译器 | C1完整优化；<br />记录方法调用次数及循环次数 | 优化后的机器码；<br />部分额外信息：方法调用次数及循环次数   | 3                 |
| 3    | C1即时编译器 | C1完整优化；<br />记录所有额外信息           | 优化后的机器码；<br />所有额外信息：分支跳转次数、类型转换等 | 2                 |
| 4    | C2即时编译器 | C2完整优化                                   | 优化后的机器码                                               | 5                 |



C1即时编译器和C2即时编译器都有独立的线程去进行处理，内部会保存一个队列，队列中存放需要编译的任务。一般即时编译器是针对方法级别来进行优化的，当然也有对循环进行优化的设计。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126182759360-1584501806.png" alt="image-20240126182815705" />



> 看C1和C2是如何进行协作的？一句话概括就是：先有C1执行并收集C2需要的数据，然后C2执行，C1忙碌时交给C2执行，C2忙碌时交给C1执行。

1. 先由C1执行过程中收集所有运行中的信息，方法执行次数、循环执行次数、分支执行次数等等，然后等待执行次数触发阈值（分层即时编译由JVM动态计算）之后，进入C2即时编译器进行深层次的优化

![image-20240126183016012](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126182959734-1606283573.png)

2. 方法字节码执行数目过少，先收集信息，JVM判断C1和C2优化性能差不多，那之后转为不收集信息，由C1直接进行优化

![image-20240126183058113](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126183041312-475011996.png)

3. C1线程都在忙碌的情况下（队列中编译任务很多），直接由C2进行优化。

![image-20240126183144693](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126183127948-717225589.png)

4. C2线程忙碌时，先由2层C1编译收集一些基础信息，多运行一会儿，然后再交由3层C1处理，由于3层C1处理效率不高，所以尽量减少这一层停留时间（C2忙碌着，一直收集也没有意义），最后C2线程不忙碌了再交由C2进行处理。

![image-20240126183527117](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126183510624-199248474.png)





#### JIT优化手段：方法内联 与 逃逸分析

> 可以借助JIT Watch工具来查看（会涉及汇编语言）：https://github.com/AdoptOpenJDK/jitwatch/tree/1.4.2
>
> - 注：以目前测试的结果来看，版本号最好用上面这个，因为比这版本高的会出现一些问题，如乱码之类的。



##### 方法内联

> 方法内联（Method Inline）：方法体中的字节码指令直接复制到调用方的字节码指令中，节省了创建栈帧的开销

方法内联是字节码层面，这里便于理解使用源码：

![image-20240126184047964](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126184031485-787401721.png)

> 方法内联的限制与条件

并不是所有的方法都可以内联，有一定的限制：

1. 方法编译之后的字节码指令总大小 < 35字节，可以直接内联。（通过 `-XX:MaxInlineSize=值` 控制）。
2. 方法编译之后的字节码指令总大小 < 325字节，并且是一个热方法。（通过- `XX:FreqInlineSize=值` 控制）。
3. 方法编译生成的机器码不能大于1000字节。（通过 `-XX:InlineSmallCode=值` 控制） 。
4. 一个接口的实现必须小于3个，如果大于三个就不会发生内联。



编译器的大部分优化都是在方法内联的基础上。所以一般来说，内联的方法越多，生成代码的执行效率越高。但是对于即时编译器来说，内联的方法越多，编译时间也就越长，程序达到峰值性能的时刻也就比较晚。

可以通过虚拟机参数 `-XX:MaxInlineLevel` 调整内联的层数，以及1层的直接递归调用（可以通过虚拟机参数 `-XX:MaxRecursiveInlineLevel` 调整）。一些常见的内联相关的参数如下表所示：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126185134246-4703152.png)







##### 逃逸分析

> 逃逸分析（Escape Analysis）：指的是如果JIT发现在方法内创建的对象不会被外部引用，那么就可以采用锁消除、标量替换等方式进行优化。
>
> 通过逃逸分析，Java Hotspot 编译器能够分析出一个新的对象的引用的使用范围，从而决定是否要将这个对象分配到堆上。
>
> - 在 JDK 6u23 版本之后，HotSpot 中默认就已经开启了逃逸分析。
> - 如果使用较早版本，可以通过`-XX"+DoEscapeAnalysis`显式开启。

逃逸分析的基本行为就是分析对象动态作用域：

- 当一个对象在方法中被定义后，对象只在方法内部使用，则认为没有发生逃逸
- 当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸，例如作为调用参数传递到其他地方中，称为方法逃逸

如下代码：因为sb通过return返回了，那么sb就有可能会被其他方法所改变，这就发生了逃逸。

```java
public static StringBuffer craeteStringBuffer(String s1, String s2) {
   StringBuffer sb = new StringBuffer();
   sb.append(s1);
   sb.append(s2);
   return sb;
}
```

所以就可以优化：

```java
public static String createStringBuffer(String s1, String s2) {
   StringBuffer sb = new StringBuffer();
   sb.append(s1);
   sb.append(s2);
   return sb.toString();
}
```





> 锁消除：指的是如果对象被判断不会逃逸出去，那么该对象就不存在并发访问问题，对象上的锁处理都不会执行，从而提高性能。

比如如下代码写法：

```java
synchronized (new Test()) {
    
}
```

当然，从上述代码也可以看出，锁消除优化在真正的工作代码中并不常见，一般加锁的对象都是支持多线程去访问的，除非程序员自己太拉胯。





> 标量替换：在Java虚拟机中，对象中的基本数据类型称为标量，引用的其他对象称为聚合量。**标量替换指的是如果方法中的对象不会逃逸，那么其中的标量就可以直接在栈上分配**。
>
> - 逃逸分析真正对性能优化比较大的方式是标量替换。
> - 常见栈上分配的场景：成员变量赋值、方法返回值、实例引用传递。
> - 通过 `-XX:+EliminateAllocations` 可以开启标量替换，`-XX:+PrintEliminateAllocations` 查看标量替换情况。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240126191709942-1728266767.png" alt="image-20240126191725807" />





因此：根据JIT即时编器优化代码的特性，在编写代码时可以注意以下几个事项，可以让代码执行时拥有更好的性能：

1. 尽量编写比较小的方法，让方法内联可以生效。
2. 高频使用的代码，特别是第三方依赖库甚至是JDK中的，如果内容过度复杂是无法内联的，可以自行实现一个特定的优化版本。
3. 注意下接口的实现数量，尽量不要超过2个，否则会影响内联的处理。
4. 高频调用的方法中创建对象临时使用，尽量不要让对象逃逸。







# ZGC详解

> 本节内容转载于：[@美团技术团队：新一代垃圾回收器ZGC的探索与实践](https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html)，ZGC调优实践之前的内容我做了调整。
>
> 作者：王东 王伟



## ZGC概述

> ZGC（The Z Garbage Collector）是JDK 11中推出的一款低延迟垃圾回收器，堆大小对STW的时间基本没有影响。它的设计目标包括：

- 停顿时间不超过10ms；
- 停顿时间不会随着堆的大小，或者活跃对象的大小而增加（对程序吞吐量影响小于15%）；
- 支持8MB~4TB级别的堆（未来支持16TB）

从设计目标来看，我们知道ZGC适用于**大内存低延迟**服务的内存管理和回收。本文主要介绍ZGC在低延时场景中的应用和卓越表现，文章内容主要分为四部分：

1. **GC之痛**：介绍实际业务中遇到的GC痛点，并分析CMS收集器和G1收集器停顿时间瓶颈；
2. **ZGC原理**：分析ZGC停顿时间比G1或CMS更短的本质原因，以及背后的技术原理；
3. **ZGC调优实践**：重点分享对ZGC调优的理解，并分析若干个实际调优案例；
4. **升级ZGC效果**：展示在生产环境应用ZGC取得的效果

## GC之痛

> 很多低延迟高可用Java服务的系统可用性经常受GC停顿的困扰。
>
> GC停顿指垃圾回收期间STW（Stop The World），当STW时，所有应用线程停止活动，等待GC停顿结束

在G1垃圾回收器中，STW时间的主要来源是在转移阶段：

1. 初始标记：STW，采用三色标记法标记从GC Root可直达的对象。 STW时间极短。
2. 并发标记：并发执行，对存活对象进行标记。
3. 最终标记：STW，处理SATB相关的对象标记。 STW时间极短。
4. 清理：STW，如果区域中没有任何存活对象就直接清理。 STW时间极短 。
5. 转移：将存活对象复制到别的区域。 STW时间较长

G1停顿时间的瓶颈主要是标记-复制中的转移阶段STW。为什么转移阶段不能和标记阶段一样并发执行呢？主要是G1未能解决转移过程中准确定位对象地址的问题。



ZGC的执行流程（待完善）:

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127201017481-1159846916.png" alt="image-20240127201034141" />



> ZGC转移时需要停顿的主要原因

在转移时，能不能让用户线程和GC线程同时工作呢？考虑下面的问题：

- 转移完之后，需要将A对对象的引用更改为新对象的引用。但是在更改前，执行A.c.count = 2，此时更改的是转移前对象中的属性。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127201520869-1003500965.png" alt="image-20240127201538295" />

- 更改引用之后, A引用了转移之后的对象，此时获取A.c.count发现属性值依然是1。这样就产生了问题。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127201637225-1564181107.png" alt="image-20240127201654842" />

所以G1为了解决问题，在转移过程中需要进行用户线程的停止。ZGC和Shenandoah GC解决了这个问题，让转移过程也能够并发执行。

> Shenandoah GC就是通过G1源代码改造而来的，通过修改对象头的方式来完成并发转移，使用的核心技术：前向指针+读前屏障。Shenandoah GC有两个版本：
>
> - 1.0版本（不稳定）存在于JDK8和JDK11中，此版本是直接在对象的前8个字节增加了前向指针
>   - 读数据：读前屏障，判断前向指针指向的是自己还是转移后的对象，然后进行操作；
>   - 写数据：写前屏障，判断Mark Work中的GC状态，若GC状态为0说明处于GC过程中，直接写入、不为0则根据GC状态值确认当前处于垃圾回收的哪个阶段，让用户线程执行垃圾回收相关的任务。
> - 后续的JDK版本中均使用2.0版本，优化前向指针的位置，仅转移阶段将其放入了Mark Word中。

Shenandoah GC执行流程：

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240128131804097-1786384664.png" alt="image-20240128131821173" />





## ZGC原理

### 全并发的ZGC

> 与CMS中的ParNew和G1类似，**ZGC也采用标记-复制算法**，不过ZGC对该算法做了重大改进：**ZGC在标记、转移和重定位阶段几乎都是并发**的，这是ZGC实现停顿时间小于10ms目标的最关键原因

ZGC垃圾回收周期如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193241007-1758742276.png)

ZGC只有三个STW阶段：**初始标记**，**再标记**，**初始转移**。其中：

- 初始标记和初始转移分别都只需要扫描所有GC Roots，其处理时间和GC Roots的数量成正比，一般情况耗时非常短；
- 再标记阶段STW时间很短，最多1ms，超过1ms则再次进入并发标记阶段。即，ZGC几乎所有暂停都只依赖于GC Roots集合大小，停顿时间不会随着堆的大小或者活跃对象的大小而增加。与ZGC对比，G1的转移阶段完全STW的，且停顿时间随存活对象的大小增加而增加。

### ZGC关键技术

ZGC通过着色指针和读屏障技术，解决了转移过程中准确访问对象的问题，实现了并发转移。大致原理描述如下：并发转移中“并发”意味着GC线程在转移对象的过程中，应用线程也在不停地访问对象。假设对象发生转移，但对象地址未及时更新，那么应用线程可能访问到旧地址，从而造成错误。而在ZGC中，应用线程访问对象将触发“读屏障”，如果发现对象被移动了，那么“读屏障”会把读出来的指针更新到对象的新地址上，这样应用线程始终访问的都是对象的新地址。那么，JVM是如何判断对象被移动过呢？就是利用对象引用的地址，即着色指针。下面介绍着色指针和读屏障技术细节。



#### 读屏障

> 读屏障（Load Barrier）是JVM向应用代码插入一小段代码的技术来实现转移后对象的获取。当应用线程从堆中读取对象引用时，就会执行这段代码。如果对象指向的不是转移后的对象，用户线程会将引用指向转移后的对象。
>
> - 需要注意的是：仅“从堆中读取对象引用”才会触发这段代码。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127202934889-1971185671.png" alt="image-20240127202952494" />



<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127203153283-1271166381.png" alt="image-20240127203209670" />





仅“从堆中读取对象引用”才会触发插入的代码：

```java
Object o = obj.FieldA   // 从堆中读取引用，需要加入屏障
<Load barrier>
Object p = o  // 无需加入屏障，因为不是从堆中读取引用
o.dosomething() // 无需加入屏障，因为不是从堆中读取引用
int i =  obj.FieldB  //无需加入屏障，因为不是对象引用
```

ZGC中读屏障的代码作用：在对象标记和转移过程中，用于确定对象的引用地址是否满足条件，并作出相应动作。



#### 着色指针

> 着色指针（Colored Pointers）是一种将信息存储在指针中的技术。

ZGC仅支持64位系统，它把64位虚拟地址空间划分为多个子空间，如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193302710-961555534.png)

其中，[0~4TB) 对应Java堆，[4TB ~ 8TB) 称为M0地址空间，[8TB ~ 12TB) 称为M1地址空间，[12TB ~ 16TB) 预留未使用，[16TB ~ 20TB) 称为Remapped空间。

当应用程序创建对象时，首先在堆空间申请一个虚拟地址，但该虚拟地址并不会映射到真正的物理地址。ZGC同时会为该对象在M0、M1和Remapped地址空间分别申请一个虚拟地址，且这三个虚拟地址对应同一个物理地址，但这三个空间在同一时间有且只有一个空间有效（就算颜色位不同，虚拟地址相同，最终指向的还是同一个物理地址）。ZGC之所以设置三个虚拟地址空间，是因为它使用“空间换时间”思想，去降低GC停顿时间。“空间换时间”中的空间是虚拟空间，而不是真正的物理空间。后续章节将详细介绍这三个空间的切换过程。

与上述地址空间划分相对应：

1. ZGC实际仅使用64位地址空间的第0~41位，用于表示对象的地址（Obejct Address）；
2. 而第42~45位存储元数据，即这中间4位为颜色位，每一位只能存放0或1，并且同一时间只有其中一位是1。

- 终结位（Finalizable）：为1时表示只能通过终结器访问；
- 重映射位(Remapped)：转移完之后，对象的引用关系已经完成变更。
- Marked0 和 Marked1：标记可达对象。

> 留个问题：为什么要使用两个Marked（即Marked0 和 Marked1），而不是一个？

3. 第47~63位固定为0，这18位未使用（Unused）。

ZGC将对象存活信息存储在42~45位中，这与传统的垃圾回收并将对象存活信息放在对象头中完全不同。

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193319151-158480141.png)



### ZGC的内存划分

> 在ZGC中，与G1垃圾回收器一样将堆内存划分成很多个区域，这些内存区域被称之为Zpage。

Zpage分成三类大中小，管控粒度比G1更细，这样更容易去控制停顿时间：

1. 小区域：2M，只能保存256KB内的对象。
2. 中区域：32M，保存256KB – 4M的对象。
3. 大区域：只保存一个大于4M的对象。





### ZGC执行流程

已完善版（未分代ZGC，即：JDK11 - JDK20）：

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127213828604-1515629765.png" alt="image-20240127213845798" />



1. **初始标记阶段：标记Gc Roots直接引用的对象为存活**。对象数量不多，所以停顿时间非常短。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127214340317-1021950629.png" alt="image-20240127214357356" />

每个阶段都需要为当前阶段定义一个颜色，当前阶段为标记阶段（Mark），所以使用Marked0（上图举例定义的颜色为红色），初始时指针的颜色位都是0。

现在需要将GC Roots能直接关联到的对象里面的颜色位（上述Marked0红色）对应的值由0改为1，

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127220150701-2004412505.png" alt="image-20240127220207735" />

这样就对GC Roots直接关联的对象的指针进行了着色，并且也就将GC Roots直接关联的对象（上图1、2、4）标记为了“存活对象”。



2. **并发标记阶段：遍历所有对象，标记可以到达的每一个对象是否存活**。用户线程使用读屏障，如果发现对象没有完成标记也会帮忙进行标记。

顺着初始标记阶段中标记的GC Roots直接关联的对象，找出这些对象引用的对象标记为“存活对象”（如下图 2 -> 5 -> 8），并使用当前阶段定义的颜色（标记阶段，如前面定义的Marked0红色）对指针进行着色。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127222745797-1506218358.png" alt="image-20240127222802389" />

因为是并发，所以用户线程会活动，假如就是上图用户线程通过4对象访问5对象，这时用户线程就会使用“读屏障”，判断4 -> 5的指针是否为当前阶段的颜色（如上图定义的Marked0红色），如果不是，那么用户线程也会帮忙进行标记（即：将 4 -> 5的指针颜色标记为Marked0红色），从而提升整个并发标记阶段的效率。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127223427986-2143402747.png" alt="image-20240127223444999" />



3. **并发处理阶段：选择需要转移的Zpage，并创建转移表，用于记录转移前对象和转移后对象地址**。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127223932626-1814249870.png" alt="image-20240127223948978" />

- **转移开始阶段**：转移GC Root直接关联的对象，不转移的对象remapped值设置成1，避免重复进行判断。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127224421368-254525344.png" alt="image-20240127224437980" />

对象1、2不需要进行转移，所以将GC Roots对象到1、2的指针着色，颜色为当前阶段定义的颜色（如上图举例定义的Remapped绿色）；并且将不转移对象的指针对应的颜色位（Remapped）设置为1。此时用户线程若去访问对象1、2的指针就会认为1、2对象已经完成了转移的处理，直接使用即可。

对于对象4，因它属于转移对象，所以就会将对象4复制到转移后的Zpage中，并且在转移映射表中记录该对象转移前的地址和转移后的地址：

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127225839358-1674831038.png" alt="image-20240127225857048" />

注意：上图GC Roots -> 4'的指针颜色为Remapped绿色，它的Remapped会被设置为1，而4' -> 5还是红色是因为4'指向的对象5是转移前的对象。



- **并发转移阶段**：将剩余对象转移到新的ZPage中，转移之后将两个对象的地址记入转移映射表。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127230423366-889751208.png" alt="image-20240127230440947" />

转移完之后，转移前的ZPage就可以清空了，转移表需要保留下来。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127230544363-2045378330.png" alt="image-20240127230601993" />

从上图也可看出：问题就出来了，虽然对象转移了，但它们的指针还没完全处理完。此时，如果用户线程访问4`对象引用的5对象：

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127231132527-1406590847.png" alt="image-20240127231149822" />

就会发现4' -> 5还是旧的引用（即还停留在Marked0阶段），那么会通过读屏障，利用转移映射表找到对象5转移后的地址5'，从而将4对5的引用进行重置，修改为对5`的引用，同时将remap标记为1代表已经重新映射完成。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127231827115-106516123.png" alt="image-20240127231844237" />

并发转移阶段结束之后，这一轮的垃圾回收就结束了，但其实并没有完成所有指针的重映射工作，这个工作会放到下一阶段，与下一阶段的标记阶段一起完成（因为都需要遍历整个对象图）。



第二轮垃圾回收：

1. **初始标记阶段：同样地，沿着GC Root标记对象。**

此阶段又要定义当前阶段的颜色，并且得需要和第一次标记阶段区分开来，所以就需要使用到Marked1了，如下举例Marked1蓝色：

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127232602361-986047317.png" alt="image-20240127232619450" />

将GC Roots直接关联的对象指针颜色改为当前阶段定义的颜色，并将该指针的颜色位修改为1.



2. **并发标记阶段：如果Marked为1代表上一轮的重映射还没有完成，先完成重映射从转移表中找到老对象转移后的新对象，再进行标记。如果Remap为1，只需要进行标记**。

沿着GC Roots直观关联的对象，找其引用的对象，此时扫描的指针颜色就会出现多种：前面Remapped定义使用的颜色 和 Marked0定义使用的颜色

- 发现是Remapped的颜色（Remap为1），则值进行标记即可（弄为当前阶段Marked1颜色，且其Marked1设为1）；
- 发现是Marked0的颜色（Marked0为1），代表上一轮的重映射还没有完成（还停留在Marked0阶段，未从Marked0 -> Remapped），那么就需要通过转移映射表找到老对象转移后的新对象，进行标记（指针弄为当前阶段Marked1的颜色，且其Marked1设为1）。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127234212755-2019541786.png" alt="image-20240127234230158" />

> 这里也就解答了为什么要使用两个Marked（即Marked0 和 Marked1）？
>
> - 因为一个用来表示当前这一轮的垃圾回收的标记阶段；而另一个用来表示上一轮垃圾回收的标记阶段，这两个不断交替使用。



3. **并发处理阶段：将转移映射表删除，释放内存空间**。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127234748690-1433243061.png" alt="image-20240127234806066" />



- **并发转移阶段 – 并发问题**：如果用户线程在帮忙转移时，GC线程也发现这个对象需要复制，那么就会去尝试写入转移映射表，如果发现映射表中已经有相同的老对象，直接放弃。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127235118952-1841161502.png" alt="image-20240127235136278" />





> 分代ZGC的设计

在JDK21之后，ZGC设计了年轻代和老年代，这样可以让大部分对象在年轻代回收，减少老年代的扫描次数，同样可以提升一定的性能。同时，年轻代和老年代的垃圾回收可以并行执行。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127235338426-1311305143.png" alt="image-20240127235356152" />

分代之后的着色指针将原来的8字节保存地址的指针拆分成了三部分：

1. 46位用来表示对象地址，最多可以表示64TB的地址空间。
2. 中间的12位为颜色位。
3. 最低4位和最高2位未使用

整个分代之后的读写屏障、着色指针的移位使用都变的异常复杂，仅作了解即可

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240127235445539-601075388.png" alt="image-20240127235503461" />





## ZGC调优实践

> ZGC不是“银弹”，需要根据服务的具体特点进行调优。网络上能搜索到实战经验较少，调优理论需自行摸索，我们在此阶段也耗费了不少时间，最终才达到理想的性能。本文的一个目的是列举一些使用ZGC时常见的问题，帮助大家使用ZGC提高服务可用性。

### 调优基础知识

#### 理解ZGC重要配置参数

> 以我们服务在生产环境中ZGC参数配置为例，说明各个参数的作用：

重要参数配置样例：

```java
-Xms10G -Xmx10G 
-XX:ReservedCodeCacheSize=256m -XX:InitialCodeCacheSize=256m 
-XX:+UnlockExperimentalVMOptions -XX:+UseZGC 
-XX:ConcGCThreads=2 -XX:ParallelGCThreads=6 
-XX:ZCollectionInterval=120 -XX:ZAllocationSpikeTolerance=5 
-XX:+UnlockDiagnosticVMOptions -XX:-ZProactive 
-Xlog:safepoint,Classhisto*=trace,age*,gc*=info:file=/opt/logs/logs/gc-%t.log:time,tid,tags:filecount=5,filesize=50m 
```

- `-Xms -Xmx`：堆的最大内存和最小内存，这里都设置为10G，程序的堆内存将保持10G不变。
- `-XX:ReservedCodeCacheSize -XX:InitialCodeCacheSize`：设置CodeCache的大小， JIT编译的代码都放在CodeCache中，一般服务64m或128m就已经足够。我们的服务因为有一定特殊性，所以设置的较大，后面会详细介绍。
- `-XX:+UnlockExperimentalVMOptions -XX:+UseZGC`：启用ZGC的配置。
- `-XX:ConcGCThreads`：并发回收垃圾的线程默认是总核数的12.5%，8核CPU默认是1调大后GC变快，但会占用程序运行时的CPU资源，吞吐会受到影响。
- `-XX:ParallelGCThreads`：STW阶段使用线程数，默认是总核数的60%。
- `-XX:ZCollectionInterval`：ZGC发生的最小时间间隔，单位秒。
- `-XX:ZAllocationSpikeTolerance`：ZGC触发自适应算法的修正系数，默认2，数值越大，越早的触发ZGC。
- `-XX:+UnlockDiagnosticVMOptions -XX:-ZProactive`：是否启用主动回收，默认开启，这里的配置表示关闭。
- `-Xlog`：设置GC日志中的内容、格式、位置以及每个日志的大小。

#### 理解ZGC触发时机

> 相比于CMS和G1的GC触发机制，ZGC的GC触发机制有很大不同。ZGC的核心特点是并发，GC过程中一直有新的对象产生。如何保证在GC完成之前，新产生的对象不会将堆占满，是ZGC参数调优的第一大目标。因为在ZGC中，当垃圾来不及回收将堆占满时，会导致正在运行的线程停顿，持续时间可能长达秒级之久。

ZGC有多种GC触发机制，总结如下：

- 阻塞内存分配请求触发：当垃圾来不及回收，垃圾将堆占满时，会导致部分线程阻塞。我们应当避免出现这种触发方式。日志中关键字是“Allocation Stall”。
- 基于分配速率的自适应算法：最主要的GC触发方式，其算法原理可简单描述为”ZGC根据近期的对象分配速率以及GC时间，计算出当内存占用达到什么阈值时触发下一次GC”。自适应算法的详细理论可参考彭成寒《新一代垃圾回收器ZGC设计与实现》一书中的内容。通过ZAllocationSpikeTolerance参数控制阈值大小，该参数默认2，数值越大，越早的触发GC。我们通过调整此参数解决了一些问题。日志中关键字是“Allocation Rate”。
- 基于固定时间间隔：通过ZCollectionInterval控制，适合应对突增流量场景。流量平稳变化时，自适应算法可能在堆使用率达到95%以上才触发GC。流量突增时，自适应算法触发的时机可能会过晚，导致部分线程阻塞。我们通过调整此参数解决流量突增场景的问题，比如定时活动、秒杀等场景。日志中关键字是“Timer”。
- 主动触发规则：类似于固定间隔规则，但时间间隔不固定，是ZGC自行算出来的时机，我们的服务因为已经加了基于固定时间间隔的触发机制，所以通过-ZProactive参数将该功能关闭，以免GC频繁，影响服务可用性。 日志中关键字是“Proactive”。
- 预热规则：服务刚启动时出现，一般不需要关注。日志中关键字是“Warmup”。
- 外部触发：代码中显式调用System.gc()触发。 日志中关键字是“System.gc()”。
- 元数据分配触发：元数据区不足时导致，一般不需要关注。 日志中关键字是“Metadata GC Threshold”。

#### 理解ZGC日志

一次完整的GC过程，需要注意的点已在图中标出

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193419389-759244267.png)

注意：该日志过滤了进入安全点的信息。正常情况，在一次GC过程中还穿插着进入安全点的操作。

GC日志中每一行都注明了GC过程中的信息，关键信息如下：

- **Start**：开始GC，并标明的GC触发的原因。上图中触发原因是自适应算法。
- **Phase-Pause Mark Start**：初始标记，会STW。
- **Phase-Pause Mark End**：再次标记，会STW。
- **Phase-Pause Relocate Start**：初始转移，会STW。
- **Heap信息**：记录了GC过程中Mark、Relocate前后的堆大小变化状况。High和Low记录了其中的最大值和最小值，我们一般关注High中Used的值，如果达到100%，在GC过程中一定存在内存分配不足的情况，需要调整GC的触发时机，更早或者更快地进行GC。
- **GC信息统计**：可以定时的打印垃圾收集信息，观察10秒内、10分钟内、10个小时内，从启动到现在的所有统计信息。利用这些统计信息，可以排查定位一些异常点。

日志中内容较多，关键点已用红线标出，含义较好理解，更详细的解释大家可以自行在网上查阅资料。

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193431941-1029405740.png)

#### 理解ZGC停顿原因

我们在实战过程中共发现了6种使程序停顿的场景，分别如下：

- **GC时，初始标记**：日志中Pause Mark Start。
- **GC时，再标记**：日志中Pause Mark End。
- **GC时，初始转移**：日志中Pause Relocate Start。
- **内存分配阻塞**：当内存不足时线程会阻塞等待GC完成，关键字是”Allocation Stall”。

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193448417-314887730.png)

- **安全点**：所有线程进入到安全点后才能进行GC，ZGC定期进入安全点判断是否需要GC。先进入安全点的线程需要等待后进入安全点的线程直到所有线程挂起。
- **dump线程、内存**：比如jstack、jmap命令。

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193458908-1286881725.png)

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193506953-1288434960.png)

### 调优案例

我们维护的服务名叫Zeus，它是美团的规则平台，常用于风控场景中的规则管理。规则运行是基于开源的表达式执行引擎[Aviator](https://github.com/killme2008/aviator)。Aviator内部将每一条表达式转化成Java的一个类，通过调用该类的接口实现表达式逻辑。

Zeus服务内的规则数量超过万条，且每台机器每天的请求量几百万。这些客观条件导致Aviator生成的类和方法会产生很多的ClassLoader和CodeCache，这些在使用ZGC时都成为过GC的性能瓶颈。接下来介绍两类调优案例。

> **第一类：内存分配阻塞，系统停顿可达到秒级**

#### 案例一：秒杀活动中流量突增，出现性能毛刺

**日志信息**：对比出现性能毛刺时间点的GC日志和业务日志，发现JVM停顿了较长时间，且停顿时GC日志中有大量的“Allocation Stall”日志。

**分析**：这种案例多出现在“自适应算法”为主要GC触发机制的场景中。ZGC是一款并发的垃圾回收器，GC线程和应用线程同时活动，在GC过程中，还会产生新的对象。GC完成之前，新产生的对象将堆占满，那么应用线程可能因为申请内存失败而导致线程阻塞。当秒杀活动开始，大量请求打入系统，但自适应算法计算的GC触发间隔较长，导致GC触发不及时，引起了内存分配阻塞，导致停顿。

**解决方法：**

（1）开启”基于固定时间间隔“的GC触发机制：-XX:ZCollectionInterval。比如调整为5秒，甚至更短。
（2）增大修正系数-XX:ZAllocationSpikeTolerance，更早触发GC。ZGC采用正态分布模型预测内存分配速率，模型修正系数ZAllocationSpikeTolerance默认值为2，值越大，越早的触发GC，Zeus中所有集群设置的是5。

#### 案例二：压测时，流量逐渐增大到一定程度后，出现性能毛刺

**日志信息**：平均1秒GC一次，两次GC之间几乎没有间隔。

**分析**：GC触发及时，但内存标记和回收速度过慢，引起内存分配阻塞，导致停顿。

**解决方法**：增大-XX:ConcGCThreads， 加快并发标记和回收速度。ConcGCThreads默认值是核数的1/8，8核机器，默认值是1。该参数影响系统吞吐，如果GC间隔时间大于GC周期，不建议调整该参数。

> **第二类：GC Roots 数量大，单次GC停顿时间长**

#### 案例三： 单次GC停顿时间30ms，与预期停顿10ms左右有较大差距

**日志信息**：观察ZGC日志信息统计，“Pause Roots ClassLoaderDataGraph”一项耗时较长。

**分析**：dump内存文件，发现系统中有上万个ClassLoader实例。我们知道ClassLoader属于GC Roots一部分，且ZGC停顿时间与GC Roots成正比，GC Roots数量越大，停顿时间越久。再进一步分析，ClassLoader的类名表明，这些ClassLoader均由Aviator组件生成。分析Aviator源码，发现Aviator对每一个表达式新生成类时，会创建一个ClassLoader，这导致了ClassLoader数量巨大的问题。在更高Aviator版本中，该问题已经被修复，即仅创建一个ClassLoader为所有表达式生成类。

**解决方法**：升级Aviator组件版本，避免生成多余的ClassLoader。

#### 案例四：服务启动后，运行时间越长，单次GC时间越长，重启后恢复

**日志信息**：观察ZGC日志信息统计，“Pause Roots CodeCache”的耗时会随着服务运行时间逐渐增长。

**分析**：CodeCache空间用于存放Java热点代码的JIT编译结果，而CodeCache也属于GC Roots一部分。通过添加-XX:+PrintCodeCacheOnCompilation参数，打印CodeCache中的被优化的方法，发现大量的Aviator表达式代码。定位到根本原因，每个表达式都是一个类中一个方法。随着运行时间越长，执行次数增加，这些方法会被JIT优化编译进入到Code Cache中，导致CodeCache越来越大。

**解决方法**：JIT有一些参数配置可以调整JIT编译的条件，但对于我们的问题都不太适用。我们最终通过业务优化解决，删除不需要执行的Aviator表达式，从而避免了大量Aviator方法进入CodeCache中。

值得一提的是，我们并不是在所有这些问题都解决后才全量部署所有集群。即使开始有各种各样的毛刺，但计算后发现，有各种问题的ZGC也比之前的CMS对服务可用性影响小。所以从开始准备使用ZGC到全量部署，大概用了2周的时间。在之后的3个月时间里，我们边做业务需求，边跟进这些问题，最终逐个解决了上述问题，从而使ZGC在各个集群上达到了一个更好表现。

## 升级ZGC效果

### 延迟降低

> TP(Top Percentile)是一项衡量系统延迟的指标：TP999表示99.9%请求都能被响应的最小耗时；TP99表示99%请求都能被响应的最小耗时。

在Zeus服务不同集群中，ZGC在低延迟（TP999 < 200ms）场景中收益较大：

- **TP999**：下降12~142ms，下降幅度18%~74%。
- **TP99**：下降5~28ms，下降幅度10%~47%。

超低延迟（TP999 < 20ms）和高延迟（TP999 > 200ms）服务收益不大，原因是这些服务的响应时间瓶颈不是GC，而是外部依赖的性能。

### 吞吐下降

对吞吐量优先的场景，ZGC可能并不适合例如，Zeus某离线集群原先使用CMS，升级ZGC后，系统吞吐量明显降低究其原因有二：

- 第一，ZGC是单代垃圾回收器，而CMS是分代垃圾回收器单代垃圾回收器每次处理的对象更多，更耗费CPU资源；
- 第二，ZGC使用读屏障，读屏障操作需耗费额外的计算资源。

## 总结

ZGC作为下一代垃圾回收器，性能非常优秀。ZGC垃圾回收过程几乎全部是并发，实际STW停顿时间极短，不到10ms。这得益于其采用的着色指针和读屏障技术。

Zeus在升级JDK 11+ZGC中，通过将风险和问题分类，然后各个击破，最终顺利实现了升级目标，GC停顿也几乎不再影响系统可用性。

最后推荐大家升级ZGC，Zeus系统因为业务特点，遇到了较多问题，而风控其他团队在升级时都非常顺利。欢迎大家加入“ZGC使用交流”群。

## 参考文献

- [ZGC官网](https://wiki.openJDK.Java.net/display/zgc/Main)
- 彭成寒.《新一代垃圾回收器ZGC设计与实现》. 机械工业出版社, 2019.
- [从实际案例聊聊Java应用的GC优化](https://tech.meituan.com/2017/12/29/JVM-optimize.html)
- [Java Hotspot G1 GC的一些关键技术](https://tech.meituan.com/2016/09/23/g1.html)

作者简介

- 王东，美团信息安全资深工程师
- 王伟，美团信息安全技术专家

更多优秀文章推荐

- https://www.jianshu.com/p/664e4da05b2c







# Java 内存模型（JMM）详解

> 声明：本章节转载自 Info 上 [深入理解Java内存模型](https://www.infoq.cn/article/java_memory_model/)。PDF文档下载 [深入理解Java内存模型【程晓明】](https://github.com/zixq-stack/Java-Note/blob/master/Java/JVM%EF%BC%9AJava%E8%99%9A%E6%8B%9F%E6%9C%BA/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B%E3%80%90%E7%A8%8B%E6%99%93%E6%98%8E%E3%80%91.pdf)
>
> 作者：程晓明。
>
> 说明：这篇文章对JMM讲得很清楚了，大致分三部分：重排序与顺序一致性；三个同步原语（lock，volatile，final）的内存语义；重排序规则及在处理器中的实现；Java内存模型的设计，及其与处理器内存模型和顺序一致性内存模型的关系。

### [#](#基础) 基础

#### [#](#并发编程模型的分类) 并发编程模型的分类

在并发编程中，我们需要处理两个关键问题：线程之间如何通信及线程之间如何同步（这里的线程是指并发执行的活动实体）。通信是指线程之间以何种机制来交换信息。在命令式编程中，线程之间的通信机制有两种：共享内存和消息传递。

在共享内存的并发模型里，线程之间共享程序的公共状态，线程之间通过写 - 读内存中的公共状态来隐式进行通信。在消息传递的并发模型里，线程之间没有公共状态，线程之间必须通过明确的发送消息来显式进行通信。

同步是指程序用于控制不同线程之间操作发生相对顺序的机制。在共享内存并发模型里，同步是显式进行的。程序员必须显式指定某个方法或某段代码需要在线程之间互斥执行。在消息传递的并发模型里，由于消息的发送必须在消息的接收之前，因此同步是隐式进行的。

Java 的并发采用的是共享内存模型，Java 线程之间的通信总是隐式进行，整个通信过程对程序员完全透明。如果编写多线程程序的 Java 程序员不理解隐式进行的线程之间通信的工作机制，很可能会遇到各种奇怪的内存可见性问题。

#### [#](#java-内存模型的抽象) Java 内存模型的抽象

在 Java 中，所有实例域、静态域和数组元素存储在堆内存中，堆内存在线程之间共享（本文使用“共享变量”这个术语代指实例域，静态域和数组元素）。局部变量（Local variables），方法定义参数（Java 语言规范称之为 formal method parameters）和异常处理器参数（exception handler parameters）不会在线程之间共享，它们不会有内存可见性问题，也不受内存模型的影响。

Java 线程之间的通信由 Java 内存模型（本文简称为 JMM）控制，JMM 决定一个线程对共享变量的写入何时对另一个线程可见。从抽象的角度来看，JMM 定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了该线程以读 / 写共享变量的副本。本地内存是 JMM 的一个抽象概念，并不真实存在。它涵盖了缓存，写缓冲区，寄存器以及其他的硬件和编译器优化。Java 内存模型的抽象示意图如下：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153114681-566819713.png)

从上图来看，线程 A 与线程 B 之间如要通信的话，必须要经历下面 2 个步骤：

- 首先，线程 A 把本地内存 A 中更新过的共享变量刷新到主内存中去。
- 然后，线程 B 到主内存中去读取线程 A 之前已更新过的共享变量。

下面通过示意图来说明这两个步骤：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153128397-156174190.png)

如上图所示，本地内存 A 和 B 有主内存中共享变量 x 的副本。假设初始时，这三个内存中的 x 值都为 0。线程 A 在执行时，把更新后的 x 值（假设值为 1）临时存放在自己的本地内存 A 中。当线程 A 和线程 B 需要通信时，线程 A 首先会把自己本地内存中修改后的 x 值刷新到主内存中，此时主内存中的 x 值变为了 1。随后，线程 B 到主内存中去读取线程 A 更新后的 x 值，此时线程 B 的本地内存的 x 值也变为了 1。

从整体来看，这两个步骤实质上是线程 A 在向线程 B 发送消息，而且这个通信过程必须要经过主内存。JMM 通过控制主内存与每个线程的本地内存之间的交互，来为 java 程序员提供内存可见性保证。

#### [#](#重排序) 重排序

在执行程序时为了提高性能，编译器和处理器常常会对指令做重排序。重排序分三种类型：

- 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
- 指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction-Level Parallelism， ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
- 内存系统的重排序。由于处理器使用缓存和读 / 写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

从 java 源代码到最终实际执行的指令序列，会分别经历下面三种重排序：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153142813-1995039118.png)

上述的 1 属于编译器重排序，2 和 3 属于处理器重排序。这些重排序都可能会导致多线程程序出现内存可见性问题。对于编译器，JMM 的编译器重排序规则会禁止特定类型的编译器重排序（不是所有的编译器重排序都要禁止）。对于处理器重排序，JMM 的处理器重排序规则会要求 java 编译器在生成指令序列时，插入特定类型的内存屏障（memory barriers，intel 称之为 memory fence）指令，通过内存屏障指令来禁止特定类型的处理器重排序（不是所有的处理器重排序都要禁止）。

JMM 属于语言级的内存模型，它确保在不同的编译器和不同的处理器平台之上，通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供一致的内存可见性保证。

#### [#](#处理器重排序与内存屏障指令) 处理器重排序与内存屏障指令

现代的处理器使用写缓冲区来临时保存向内存写入的数据。写缓冲区可以保证指令流水线持续运行，它可以避免由于处理器停顿下来等待向内存写入数据而产生的延迟。同时，通过以批处理的方式刷新写缓冲区，以及合并写缓冲区中对同一内存地址的多次写，可以减少对内存总线的占用。虽然写缓冲区有这么多好处，但每个处理器上的写缓冲区，仅仅对它所在的处理器可见。这个特性会对内存操作的执行顺序产生重要的影响：处理器对内存的读 / 写操作的执行顺序，不一定与内存实际发生的读 / 写操作顺序一致！为了具体说明，请看下面示例：

```java
// Processor A
a = 1; //A1  
x = b; //A2

// Processor B
b = 2; //B1  
y = a; //B2

// 初始状态：a = b = 0；处理器允许执行后得到结果：x = y = 0
```

假设处理器 A 和处理器 B 按程序的顺序并行执行内存访问，最终却可能得到 x = y = 0 的结果。具体的原因如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153156655-2068587440.png)

这里处理器 A 和处理器 B 可以同时把共享变量写入自己的写缓冲区（A1，B1），然后从内存中读取另一个共享变量（A2，B2），最后才把自己写缓存区中保存的脏数据刷新到内存中（A3，B3）。当以这种时序执行时，程序就可以得到 x = y = 0 的结果。

从内存操作实际发生的顺序来看，直到处理器 A 执行 A3 来刷新自己的写缓存区，写操作 A1 才算真正执行了。虽然处理器 A 执行内存操作的顺序为：A1->A2，但内存操作实际发生的顺序却是：A2->A1。此时，处理器 A 的内存操作顺序被重排序了（处理器 B 的情况和处理器 A 一样，这里就不赘述了）。

这里的关键是，由于写缓冲区仅对自己的处理器可见，它会导致处理器执行内存操作的顺序可能会与内存实际的操作执行顺序不一致。由于现代的处理器都会使用写缓冲区，因此现代的处理器都会允许对写 - 读操做重排序。

下面是常见处理器允许的重排序类型的列表：

|           | Load-Load | Load-Store | Store-Store | Store-Load | 数据依赖 |
| --------- | --------- | ---------- | ----------- | ---------- | -------- |
| sparc-TSO | N         | N          | N           | Y          | N        |
| x86       | N         | N          | N           | Y          | N        |
| ia64      | Y         | Y          | Y           | Y          | N        |
| PowerPC   | Y         | Y          | Y           | Y          | N        |

上表单元格中的“N”表示处理器不允许两个操作重排序，“Y”表示允许重排序。

从上表我们可以看出：常见的处理器都允许 Store-Load 重排序；常见的处理器都不允许对存在数据依赖的操作做重排序。sparc-TSO 和 x86 拥有相对较强的处理器内存模型，它们仅允许对写 - 读操作做重排序（因为它们都使用了写缓冲区）。

- ※注 1：sparc-TSO 是指以 TSO(Total Store Order) 内存模型运行时，sparc 处理器的特性。
- ※注 2：上表中的 x86 包括 x64 及 AMD64。
- ※注 3：由于 ARM 处理器的内存模型与 PowerPC 处理器的内存模型非常类似，本文将忽略它。
- ※注 4：数据依赖性后文会专门说明。

为了保证内存可见性，java 编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序。JMM 把内存屏障指令分为下列四类：

| 屏障类型            | 指令示例                   | 说明                                                         |
| ------------------- | -------------------------- | ------------------------------------------------------------ |
| LoadLoad Barriers   | Load1; LoadLoad; Load2     | 确保 Load1 数据的装载，之前于 Load2 及所有后续装载指令的装载。 |
| StoreStore Barriers | Store1; StoreStore; Store2 | 确保 Store1 数据对其他处理器可见（刷新到内存），之前于 Store2 及所有后续存储指令的存储。 |
| LoadStore Barriers  | Load1; LoadStore; Store2   | 确保 Load1 数据装载，之前于 Store2 及所有后续的存储指令刷新到内存。 |
| StoreLoad Barriers  | Store1; StoreLoad; Load2   | 确保 Store1 数据对其他处理器变得可见（指刷新到内存），之前于 Load2 及所有后续装载指令的装载。 |

StoreLoad Barriers 会使该屏障之前的所有内存访问指令（存储和装载指令）完成之后，才执行该屏障之后的内存访问指令。

StoreLoad Barriers 是一个“全能型”的屏障，它同时具有其他三个屏障的效果。现代的多处理器大都支持该屏障（其他类型的屏障不一定被所有处理器支持）。执行该屏障开销会很昂贵，因为当前处理器通常要把写缓冲区中的数据全部刷新到内存中（buffer fully flush）。

#### [#](#happens-before) happens-before

##### happens-before八大规则

1. **单一线程规则（Single Thread rule）**：在一个线程内，在程序前面的操作先行发生于后面的操作。
2. **管道锁定规则（Monitor Lock Rule）**：一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。
3. **Volatile变量规则（Volatile Variable Rule）**：对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。
4. **线程启动规则（Thread Start Rule）**：Thread 对象的 `start()` 方法调用先行发生于此线程的每一个动作。
5. **线程加入规则（Thread Join Rule）**：Thread 对象的结束先行发生于 `join()` 方法返回。
6. **线程中断规则（Thread Interruption Rule）**：对线程 `interrupt()` 方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过 `interrupted()` 方法检测到是否有中断发生。
7. **对象终结规则（Finalizer Rule）**：一个对象的初始化完成(构造函数执行结束)先行发生于它的 `finalize()` 方法的开始。
8. **传递性（Transitivity）**：如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。



从 JDK5 开始，Java 使用新的 JSR -133 内存模型（本文除非特别说明，针对的都是 JSR- 133 内存模型）。JSR-133 提出了 happens-before 的概念，通过这个概念来阐述操作之间的内存可见性。如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在 happens-before 关系。这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。 与程序员密切相关的 happens-before 规则如下：

- 程序顺序规则：一个线程中的每个操作，happens- before 于该线程中的任意后续操作。
- 监视器锁规则：对一个监视器锁的解锁，happens- before 于随后对这个监视器锁的加锁。
- volatile 变量规则：对一个 volatile 域的写，happens- before 于任意后续对这个 volatile 域的读。
- 传递性：如果 A happens- before B，且 B happens- before C，那么 A happens- before C。

注意，两个操作之间具有 happens-before 关系，并不意味着前一个操作必须要在后一个操作之前执行！happens-before 仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前（the first is visible to and ordered before the second）。happens- before 的定义很微妙，后文会具体说明 happens-before 为什么要这么定义。

happens-before 与 JMM 的关系如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153215030-1308950993.png)

如上图所示，一个 happens-before 规则通常对应于多个编译器重排序规则和处理器重排序规则。对于 java 程序员来说，happens-before 规则简单易懂，它避免程序员为了理解 JMM 提供的内存可见性保证而去学习复杂的重排序规则以及这些规则的具体实现。

### [#](#重排序-1) 重排序

#### [#](#数据依赖性) 数据依赖性

如果两个操作访问同一个变量，且这两个操作中有一个为写操作，此时这两个操作之间就存在数据依赖性。数据依赖分下列三种类型：

| 名称   | 代码示例     | 说明                           |
| ------ | ------------ | ------------------------------ |
| 写后读 | a = 1;b = a; | 写一个变量之后，再读这个位置。 |
| 写后写 | a = 1;a = 2; | 写一个变量之后，再写这个变量。 |
| 读后写 | a = b;b = 1; | 读一个变量之后，再写这个变量。 |

上面三种情况，只要重排序两个操作的执行顺序，程序的执行结果将会被改变。

前面提到过，编译器和处理器可能会对操作做重排序。编译器和处理器在重排序时，会遵守数据依赖性，编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序。

注意，这里所说的数据依赖性仅针对单个处理器中执行的指令序列和单个线程中执行的操作，不同处理器之间和不同线程之间的数据依赖性不被编译器和处理器考虑。

#### [#](#as-if-serial-语义) as-if-serial 语义

as-if-serial 语义的意思指：不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。编译器，runtime 和处理器都必须遵守 as-if-serial 语义。

为了遵守 as-if-serial 语义，编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果。但是，如果操作之间不存在数据依赖关系，这些操作可能被编译器和处理器重排序。为了具体说明，请看下面计算圆面积的代码示例：

```java
double pi  = 3.14;    //A
double r   = 1.0;     //B
double area = pi * r * r; //C
```

上面三个操作的数据依赖关系如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153231775-360196760.png)

如上图所示，A 和 C 之间存在数据依赖关系，同时 B 和 C 之间也存在数据依赖关系。因此在最终执行的指令序列中，C 不能被重排序到 A 和 B 的前面（C 排到 A 和 B 的前面，程序的结果将会被改变）。但 A 和 B 之间没有数据依赖关系，编译器和处理器可以重排序 A 和 B 之间的执行顺序。下图是该程序的两种执行顺序：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153241257-234552626.png)

as-if-serial 语义把单线程程序保护了起来，遵守 as-if-serial 语义的编译器，runtime 和处理器共同为编写单线程程序的程序员创建了一个幻觉：单线程程序是按程序的顺序来执行的。as-if-serial 语义使单线程程序员无需担心重排序会干扰他们，也无需担心内存可见性问题。

#### [#](#程序顺序规则) 程序顺序规则

根据 happens- before 的程序顺序规则，上面计算圆的面积的示例代码存在三个 happens- before 关系：

- A happens- before B；
- B happens- before C；
- A happens- before C；

这里的第 3 个 happens- before 关系，是根据 happens- before 的传递性推导出来的。

这里 A happens- before B，但实际执行时 B 却可以排在 A 之前执行（看上面的重排序后的执行顺序）。在第一章提到过，如果 A happens- before B，JMM 并不要求 A 一定要在 B 之前执行。JMM 仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前。这里操作 A 的执行结果不需要对操作 B 可见；而且重排序操作 A 和操作 B 后的执行结果，与操作 A 和操作 B 按 happens- before 顺序执行的结果一致。在这种情况下，JMM 会认为这种重排序并不非法（not illegal），JMM 允许这种重排序。

在计算机中，软件技术和硬件技术有一个共同的目标：在不改变程序执行结果的前提下，尽可能的开发并行度。编译器和处理器遵从这一目标，从 happens- before 的定义我们可以看出，JMM 同样遵从这一目标。

#### [#](#重排序对多线程的影响) 重排序对多线程的影响

现在让我们来看看，重排序是否会改变多线程程序的执行结果。请看下面的示例代码：

```java
class ReorderExample {
    int a = 0;
    boolean flag = false;

    public void writer() {
        a = 1;                   //1
        flag = true;             //2
    }

    Public void reader() {
        if (flag) {                //3
            int i =  a * a;        //4
            ……
        }
    }
}
```

flag 变量是个标记，用来标识变量 a 是否已被写入。这里假设有两个线程 A 和 B，A 首先执行 writer() 方法，随后 B 线程接着执行 reader() 方法。线程 B 在执行操作 4 时，能否看到线程 A 在操作 1 对共享变量 a 的写入?

答案是：不一定能看到。

由于操作 1 和操作 2 没有数据依赖关系，编译器和处理器可以对这两个操作重排序；同样，操作 3 和操作 4 没有数据依赖关系，编译器和处理器也可以对这两个操作重排序。让我们先来看看，当操作 1 和操作 2 重排序时，可能会产生什么效果? 请看下面的程序执行时序图：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153258466-2041374938.png)

如上图所示，操作 1 和操作 2 做了重排序。程序执行时，线程 A 首先写标记变量 flag，随后线程 B 读这个变量。由于条件判断为真，线程 B 将读取变量 a。此时，变量 a 还根本没有被线程 A 写入，在这里多线程程序的语义被重排序破坏了！

※注：本文统一用红色的虚箭线表示错误的读操作，用绿色的虚箭线表示正确的读操作。

下面再让我们看看，当操作 3 和操作 4 重排序时会产生什么效果（借助这个重排序，可以顺便说明控制依赖性）。下面是操作 3 和操作 4 重排序后，程序的执行时序图：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153310727-2103553273.png)

在程序中，操作 3 和操作 4 存在控制依赖关系。当代码中存在控制依赖性时，会影响指令序列执行的并行度。为此，编译器和处理器会采用猜测（Speculation）执行来克服控制相关性对并行度的影响。以处理器的猜测执行为例，执行线程 B 的处理器可以提前读取并计算 a*a，然后把计算结果临时保存到一个名为重排序缓冲（reorder buffer ROB）的硬件缓存中。当接下来操作 3 的条件判断为真时，就把该计算结果写入变量 i 中。

从图中我们可以看出，猜测执行实质上对操作 3 和 4 做了重排序。重排序在这里破坏了多线程程序的语义！

在单线程程序中，对存在控制依赖的操作重排序，不会改变执行结果（这也是 as-if-serial 语义允许对存在控制依赖的操作做重排序的原因）；但在多线程程序中，对存在控制依赖的操作重排序，可能会改变程序的执行结果。

### [#](#顺序一致性) 顺序一致性

#### [#](#数据竞争与顺序一致性保证) 数据竞争与顺序一致性保证

当程序未正确同步时，就会存在数据竞争。java 内存模型规范对数据竞争的定义如下：

- 在一个线程中写一个变量，
- 在另一个线程读同一个变量，
- 而且写和读没有通过同步来排序。

当代码中包含数据竞争时，程序的执行往往产生违反直觉的结果（前一章的示例正是如此）。如果一个多线程程序能正确同步，这个程序将是一个没有数据竞争的程序。

JMM 对正确同步的多线程程序的内存一致性做了如下保证：

- 如果程序是正确同步的，程序的执行将具有顺序一致性（sequentially consistent）-- 即程序的执行结果与该程序在顺序一致性内存模型中的执行结果相同（马上我们将会看到，这对于程序员来说是一个极强的保证）。这里的同步是指广义上的同步，包括对常用同步原语（lock，volatile 和 final）的正确使用。

#### [#](#顺序一致性内存模型) 顺序一致性内存模型

顺序一致性内存模型是一个被计算机科学家理想化了的理论参考模型，它为程序员提供了极强的内存可见性保证。顺序一致性内存模型有两大特性：

- 一个线程中的所有操作必须按照程序的顺序来执行。 +（不管程序是否同步）所有线程都只能看到一个单一的操作执行顺序。在顺序一致性内存模型中，每个操作都必须原子执行且立刻对所有线程可见。 顺序一致性内存模型为程序员提供的视图如下：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153325852-100921933.png)

在概念上，顺序一致性模型有一个单一的全局内存，这个内存通过一个左右摆动的开关可以连接到任意一个线程。同时，每一个线程必须按程序的顺序来执行内存读 / 写操作。从上图我们可以看出，在任意时间点最多只能有一个线程可以连接到内存。当多个线程并发执行时，图中的开关装置能把所有线程的所有内存读 / 写操作串行化。

为了更好的理解，下面我们通过两个示意图来对顺序一致性模型的特性做进一步的说明。

假设有两个线程 A 和 B 并发执行。其中 A 线程有三个操作，它们在程序中的顺序是：A1->A2->A3。B 线程也有三个操作，它们在程序中的顺序是：B1->B2->B3。

假设这两个线程使用监视器来正确同步：A 线程的三个操作执行后释放监视器，随后 B 线程获取同一个监视器。那么程序在顺序一致性模型中的执行效果将如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153339094-1330816608.png)

现在我们再假设这两个线程没有做同步，下面是这个未同步程序在顺序一致性模型中的执行示意图：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153348555-947991834.png)

未同步程序在顺序一致性模型中虽然整体执行顺序是无序的，但所有线程都只能看到一个一致的整体执行顺序。以上图为例，线程 A 和 B 看到的执行顺序都是：B1->A1->A2->B2->A3->B3。之所以能得到这个保证是因为顺序一致性内存模型中的每个操作必须立即对任意线程可见。

但是，在 JMM 中就没有这个保证。未同步程序在 JMM 中不但整体的执行顺序是无序的，而且所有线程看到的操作执行顺序也可能不一致。比如，在当前线程把写过的数据缓存在本地内存中，且还没有刷新到主内存之前，这个写操作仅对当前线程可见；从其他线程的角度来观察，会认为这个写操作根本还没有被当前线程执行。只有当前线程把本地内存中写过的数据刷新到主内存之后，这个写操作才能对其他线程可见。在这种情况下，当前线程和其它线程看到的操作执行顺序将不一致。

#### [#](#同步程序的顺序一致性效果) 同步程序的顺序一致性效果

下面我们对前面的示例程序 ReorderExample 用监视器来同步，看看正确同步的程序如何具有顺序一致性。

请看下面的示例代码：

```java
class SynchronizedExample {
    int a = 0;
    boolean flag = false;

    public synchronized void writer() {
        a = 1;
        flag = true;
    }

    public synchronized void reader() {
        if (flag) {
            int i = a;
            ……
        }
    }
}
```

上面示例代码中，假设 A 线程执行 writer() 方法后，B 线程执行 reader() 方法。这是一个正确同步的多线程程序。根据 JMM 规范，该程序的执行结果将与该程序在顺序一致性模型中的执行结果相同。下面是该程序在两个内存模型中的执行时序对比图：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153404232-1999093534.png)

在顺序一致性模型中，所有操作完全按程序的顺序串行执行。而在 JMM 中，临界区内的代码可以重排序（但 JMM 不允许临界区内的代码“逸出”到临界区之外，那样会破坏监视器的语义）。JMM 会在退出监视器和进入监视器这两个关键时间点做一些特别处理，使得线程在这两个时间点具有与顺序一致性模型相同的内存视图（具体细节后文会说明）。虽然线程 A 在临界区内做了重排序，但由于监视器的互斥执行的特性，这里的线程 B 根本无法“观察”到线程 A 在临界区内的重排序。这种重排序既提高了执行效率，又没有改变程序的执行结果。

从这里我们可以看到 JMM 在具体实现上的基本方针：在不改变（正确同步的）程序执行结果的前提下，尽可能的为编译器和处理器的优化打开方便之门。

#### [#](#未同步程序的执行特性) 未同步程序的执行特性

对于未同步或未正确同步的多线程程序，JMM 只提供最小安全性：线程执行时读取到的值，要么是之前某个线程写入的值，要么是默认值（0，null，false），JMM 保证线程读操作读取到的值不会无中生有（out of thin air）的冒出来。为了实现最小安全性，JVM 在堆上分配对象时，首先会清零内存空间，然后才会在上面分配对象（JVM 内部会同步这两个操作）。因此，在以清零的内存空间（pre-zeroed memory）分配对象时，域的默认初始化已经完成了。

JMM 不保证未同步程序的执行结果与该程序在顺序一致性模型中的执行结果一致。因为未同步程序在顺序一致性模型中执行时，整体上是无序的，其执行结果无法预知。保证未同步程序在两个模型中的执行结果一致毫无意义。

和顺序一致性模型一样，未同步程序在 JMM 中的执行时，整体上也是无序的，其执行结果也无法预知。同时，未同步程序在这两个模型中的执行特性有下面几个差异：

- 顺序一致性模型保证单线程内的操作会按程序的顺序执行，而 JMM 不保证单线程内的操作会按程序的顺序执行（比如上面正确同步的多线程程序在临界区内的重排序）。这一点前面已经讲过了，这里就不再赘述。
- 顺序一致性模型保证所有线程只能看到一致的操作执行顺序，而 JMM 不保证所有线程能看到一致的操作执行顺序。这一点前面也已经讲过，这里就不再赘述。
- JMM 不保证对 64 位的 long 型和 double 型变量的读 / 写操作具有原子性，而顺序一致性模型保证对所有的内存读 / 写操作都具有原子性。

第 3 个差异与处理器总线的工作机制密切相关。在计算机中，数据通过总线在处理器和内存之间传递。每次处理器和内存之间的数据传递都是通过一系列步骤来完成的，这一系列步骤称之为总线事务（bus transaction）。总线事务包括读事务（read transaction）和写事务（write transaction）。读事务从内存传送数据到处理器，写事务从处理器传送数据到内存，每个事务会读 / 写内存中一个或多个物理上连续的字。这里的关键是，总线会同步试图并发使用总线的事务。在一个处理器执行总线事务期间，总线会禁止其它所有的处理器和 I/O 设备执行内存的读 / 写。下面让我们通过一个示意图来说明总线的工作机制：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153423006-114753786.png)

如上图所示，假设处理器 A，B 和 C 同时向总线发起总线事务，这时总线仲裁（bus arbitration）会对竞争作出裁决，这里我们假设总线在仲裁后判定处理器 A 在竞争中获胜（总线仲裁会确保所有处理器都能公平的访问内存）。此时处理器 A 继续它的总线事务，而其它两个处理器则要等待处理器 A 的总线事务完成后才能开始再次执行内存访问。假设在处理器 A 执行总线事务期间（不管这个总线事务是读事务还是写事务），处理器 D 向总线发起了总线事务，此时处理器 D 的这个请求会被总线禁止。

总线的这些工作机制可以把所有处理器对内存的访问以串行化的方式来执行；在任意时间点，最多只能有一个处理器能访问内存。这个特性确保了单个总线事务之中的内存读 / 写操作具有原子性。

在一些 32 位的处理器上，如果要求对 64 位数据的读 / 写操作具有原子性，会有比较大的开销。为了照顾这种处理器，java 语言规范鼓励但不强求 JVM 对 64 位的 long 型变量和 double 型变量的读 / 写具有原子性。当 JVM 在这种处理器上运行时，会把一个 64 位 long/ double 型变量的读 / 写操作拆分为两个 32 位的读 / 写操作来执行。这两个 32 位的读 / 写操作可能会被分配到不同的总线事务中执行，此时对这个 64 位变量的读 / 写将不具有原子性。

当单个内存操作不具有原子性，将可能会产生意想不到后果。请看下面示意图：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153434834-76545725.png)

如上图所示，假设处理器 A 写一个 long 型变量，同时处理器 B 要读这个 long 型变量。处理器 A 中 64 位的写操作被拆分为两个 32 位的写操作，且这两个 32 位的写操作被分配到不同的写事务中执行。同时处理器 B 中 64 位的读操作被拆分为两个 32 位的读操作，且这两个 32 位的读操作被分配到同一个的读事务中执行。当处理器 A 和 B 按上图的时序来执行时，处理器 B 将看到仅仅被处理器 A“写了一半“的无效值。

### [#](#总结) 总结

#### [#](#处理器内存模型) 处理器内存模型

顺序一致性内存模型是一个理论参考模型，JMM 和处理器内存模型在设计时通常会把顺序一致性内存模型作为参照。JMM 和处理器内存模型在设计时会对顺序一致性模型做一些放松，因为如果完全按照顺序一致性模型来实现处理器和 JMM，那么很多的处理器和编译器优化都要被禁止，这对执行性能将会有很大的影响。

根据对不同类型读 / 写操作组合的执行顺序的放松，可以把常见处理器的内存模型划分为下面几种类型：

- 放松程序中写 - 读操作的顺序，由此产生了 total store ordering 内存模型（简称为 TSO）。
- 在前面 1 的基础上，继续放松程序中写 - 写操作的顺序，由此产生了 partial store order 内存模型（简称为 PSO）。
- 在前面 1 和 2 的基础上，继续放松程序中读 - 写和读 - 读操作的顺序，由此产生了 relaxed memory order 内存模型（简称为 RMO）和 PowerPC 内存模型。

注意，这里处理器对读 / 写操作的放松，是以两个操作之间不存在数据依赖性为前提的（因为处理器要遵守 as-if-serial 语义，处理器不会对存在数据依赖性的两个内存操作做重排序）。

下面的表格展示了常见处理器内存模型的细节特征：

| 内存模型名称 | 对应的处理器  | Store-Load 重排序 | Store-Store 重排序 | Load-Load 和 Load-Store 重排序 | 可以更早读取到其它处理器的写 | 可以更早读取到当前处理器的写 |
| ------------ | ------------- | ----------------- | ------------------ | ------------------------------ | ---------------------------- | ---------------------------- |
| TSO          | sparc-TSO X64 | Y                 |                    |                                |                              | Y                            |
| PSO          | sparc-PSO     | Y                 | Y                  |                                |                              | Y                            |
| RMO          | ia64          | Y                 | Y                  | Y                              |                              | Y                            |
| PowerPC      | PowerPC       | Y                 | Y                  | Y                              | Y                            | Y                            |

在这个表格中，我们可以看到所有处理器内存模型都允许写 - 读重排序，原因在第一章以说明过：它们都使用了写缓存区，写缓存区可能导致写 - 读操作重排序。同时，我们可以看到这些处理器内存模型都允许更早读到当前处理器的写，原因同样是因为写缓存区：由于写缓存区仅对当前处理器可见，这个特性导致当前处理器可以比其他处理器先看到临时保存在自己的写缓存区中的写。

上面表格中的各种处理器内存模型，从上到下，模型由强变弱。越是追求性能的处理器，内存模型设计的会越弱。因为这些处理器希望内存模型对它们的束缚越少越好，这样它们就可以做尽可能多的优化来提高性能。

由于常见的处理器内存模型比 JMM 要弱，java 编译器在生成字节码时，会在执行指令序列的适当位置插入内存屏障来限制处理器的重排序。同时，由于各种处理器内存模型的强弱并不相同，为了在不同的处理器平台向程序员展示一个一致的内存模型，JMM 在不同的处理器中需要插入的内存屏障的数量和种类也不相同。下图展示了 JMM 在不同处理器内存模型中需要插入的内存屏障的示意图：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153453221-750327606.png)

如上图所示，JMM 屏蔽了不同处理器内存模型的差异，它在不同的处理器平台之上为 java 程序员呈现了一个一致的内存模型。

#### [#](#jmm-处理器内存模型与顺序一致性内存模型之间的关系) JMM，处理器内存模型与顺序一致性内存模型之间的关系

JMM 是一个语言级的内存模型，处理器内存模型是硬件级的内存模型，顺序一致性内存模型是一个理论参考模型。下面是语言内存模型，处理器内存模型和顺序一致性内存模型的强弱对比示意图：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153506773-2141474464.png)

从上图我们可以看出：常见的 4 种处理器内存模型比常用的 3 中语言内存模型要弱，处理器内存模型和语言内存模型都比顺序一致性内存模型要弱。同处理器内存模型一样，越是追求执行性能的语言，内存模型设计的会越弱。

#### [#](#jmm-的设计) JMM 的设计

从 JMM 设计者的角度来说，在设计 JMM 时，需要考虑两个关键因素：

- 程序员对内存模型的使用。程序员希望内存模型易于理解，易于编程。程序员希望基于一个强内存模型来编写代码。
- 编译器和处理器对内存模型的实现。编译器和处理器希望内存模型对它们的束缚越少越好，这样它们就可以做尽可能多的优化来提高性能。编译器和处理器希望实现一个弱内存模型。

由于这两个因素互相矛盾，所以 JSR-133 专家组在设计 JMM 时的核心目标就是找到一个好的平衡点：一方面要为程序员提供足够强的内存可见性保证；另一方面，对编译器和处理器的限制要尽可能的放松。下面让我们看看 JSR-133 是如何实现这一目标的。

为了具体说明，请看前面提到过的计算圆面积的示例代码：

```java
double pi  = 3.14;    //A
double r   = 1.0;     //B
double area = pi * r * r; //C
```

上面计算圆的面积的示例代码存在三个 happens- before 关系：

- A happens- before B；
- B happens- before C；
- A happens- before C；

由于 A happens- before B，happens- before 的定义会要求：A 操作执行的结果要对 B 可见，且 A 操作的执行顺序排在 B 操作之前。 但是从程序语义的角度来说，对 A 和 B 做重排序即不会改变程序的执行结果，也还能提高程序的执行性能（允许这种重排序减少了对编译器和处理器优化的束缚）。也就是说，上面这 3 个 happens- before 关系中，虽然 2 和 3 是必需要的，但 1 是不必要的。因此，JMM 把 happens- before 要求禁止的重排序分为了下面两类：

- 会改变程序执行结果的重排序。
- 不会改变程序执行结果的重排序。

JMM 对这两种不同性质的重排序，采取了不同的策略：

- 对于会改变程序执行结果的重排序，JMM 要求编译器和处理器必须禁止这种重排序。
- 对于不会改变程序执行结果的重排序，JMM 对编译器和处理器不作要求（JMM 允许这种重排序）。

下面是 JMM 的设计示意图：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153520090-1228290304.png)

从上图可以看出两点：

- JMM 向程序员提供的 happens- before 规则能满足程序员的需求。JMM 的 happens- before 规则不但简单易懂，而且也向程序员提供了足够强的内存可见性保证（有些内存可见性保证其实并不一定真实存在，比如上面的 A happens- before B）。
- JMM 对编译器和处理器的束缚已经尽可能的少。从上面的分析我们可以看出，JMM 其实是在遵循一个基本原则：只要不改变程序的执行结果（指的是单线程程序和正确同步的多线程程序），编译器和处理器怎么优化都行。比如，如果编译器经过细致的分析后，认定一个锁只会被单个线程访问，那么这个锁可以被消除。再比如，如果编译器经过细致的分析后，认定一个 volatile 变量仅仅只会被单个线程访问，那么编译器可以把这个 volatile 变量当作一个普通变量来对待。这些优化既不会改变程序的执行结果，又能提高程序的执行效率。

#### [#](#jmm-的内存可见性保证) JMM 的内存可见性保证

Java 程序的内存可见性保证按程序类型可以分为下列三类：

- 单线程程序。单线程程序不会出现内存可见性问题。编译器，runtime 和处理器会共同确保单线程程序的执行结果与该程序在顺序一致性模型中的执行结果相同。
- 正确同步的多线程程序。正确同步的多线程程序的执行将具有顺序一致性（程序的执行结果与该程序在顺序一致性内存模型中的执行结果相同）。这是 JMM 关注的重点，JMM 通过限制编译器和处理器的重排序来为程序员提供内存可见性保证。
- 未同步 / 未正确同步的多线程程序。JMM 为它们提供了最小安全性保障：线程执行时读取到的值，要么是之前某个线程写入的值，要么是默认值（0，null，false）。

下图展示了这三类程序在 JMM 中与在顺序一致性内存模型中的执行结果的异同：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240122153533066-1000175401.png)

只要多线程程序是正确同步的，JMM 保证该程序在任意的处理器平台上的执行结果，与该程序在顺序一致性内存模型中的执行结果一致。

#### [#](#jsr-133-对旧内存模型的修补) JSR-133 对旧内存模型的修补

JSR-133 对 JDK5 之前的旧内存模型的修补主要有两个：

- 增强 volatile 的内存语义。旧内存模型允许 volatile 变量与普通变量重排序。JSR-133 严格限制 volatile 变量与普通变量的重排序，使 volatile 的写 - 读和锁的释放 - 获取具有相同的内存语义。
- 增强 final 的内存语义。在旧内存模型中，多次读取同一个 final 变量的值可能会不相同。为此，JSR-133 为 final 增加了两个重排序规则。现在，final 具有了初始化安全性。





# Java问题排查之Linux命令

> 声明：此章节内容整理自：[@pdai：调试排错 - Java 问题排查之Linux命令](https://www.pdai.tech/md/java/jvm/java-jvm-debug-tools-linux.html)



## 文本操作

### 文本查找 - grep

grep常用命令：

```bash
# 基本使用
grep yoursearchkeyword f.txt										# 文件查找
grep 'KeyWord otherKeyWord' f.txt cpf.txt							# 多文件查找, 含空格加引号
grep 'KeyWord' /home/admin -r -n									# 目录下查找所有符合关键字的文件
grep 'keyword' /home/admin -r -n -i									# -i 忽略大小写
grep 'KeyWord' /home/admin -r -n --include *.{vm,Java}				# 指定文件后缀
grep 'KeyWord' /home/admin -r -n --exclude *.{vm,Java}				# 反匹配

# cat + grep
cat f.txt | grep -i keyword											# 查找所有keyword且不分大小写  
cat f.txt | grep -c 'KeyWord'										# 统计Keyword次数

# seq + grep
seq 10 | grep 5 -A 3												# 上匹配
seq 10 | grep 5 -B 3												# 下匹配
seq 10 | grep 5 -C 3												# 上下匹配，平时用这个就妥了
```

Grep的参数：

```bash
--color=auto													# 显示颜色;
-i, --ignore-case												# 忽略字符大小写;
-o, --only-matching												# 只显示匹配到的部分;
-n, --line-number												# 显示行号;
-v, --invert-match												# 反向显示,显示未匹配到的行;
-E, --extended-regexp											# 支持使用扩展的正则表达式;
-q, --quiet, --silent											# 静默模式,即不输出任何信息;
-w, --word-regexp												# 整行匹配整个单词;
-c, --count														# 统计匹配到的行数; print a count of matching lines;

-B, --before-context=NUM：print NUM lines of leading context		# 后#行
-A, --after-context=NUM：print NUM lines of trailing context		# 前#行
-C, --context=NUM：print NUM lines of output context				# 前后各#行
```



### 文本分析 - awk

awk基本命令：

```bash
# 基本使用
awk '{print $4,$6}' f.txt
awk '{print NR,$0}' f.txt cpf.txt    
awk '{print FNR,$0}' f.txt cpf.txt
awk '{print FNR,FILENAME,$0}' f.txt cpf.txt
awk '{print FILENAME,"NR="NR,"FNR="FNR,"$"NF"="$NF}' f.txt cpf.txt
echo 1:2:3:4 | awk -F: '{print $1,$2,$3,$4}'

# 匹配
awk '/ldb/ {print}' f.txt						# 匹配ldb
awk '!/ldb/ {print}' f.txt						# 不匹配ldb
awk '/ldb/ && /LISTEN/ {print}' f.txt			# 匹配ldb和LISTEN
awk '$5 ~ /ldb/ {print}' f.txt					# 第五列匹配ldb
```

内建变量

```bash
`NR`: NR表示从awk开始执行后，按照记录分隔符读取的数据次数，默认的记录分隔符为换行符，因此默认的就是读取的数据行数，NR可以理解为Number of Record的缩写

`FNR`: 在awk处理多个输入文件的时候，在处理完第一个文件后，NR并不会从1开始，而是继续累加，因此就出现了FNR，每当处理一个新文件的时候，FNR就从1开始计数，FNR可以理解为File Number of Record

`NF`: NF表示目前的记录被分割的字段的数目，NF可以理解为Number of Field
```

更多请参考：[Linux awk 命令](https://www.runoob.com/linux/linux-comm-awk.html)

### [#](#文本处理-sed) 文本处理 - sed

sed常用：

```bash
# 文本打印
sed -n '3p' xxx.log							# 只打印第三行
sed -n '$p' xxx.log							# 只打印最后一行
sed -n '3,9p' xxx.log						# 只查看文件的第3行到第9行
sed -n -e '3,9p' -e '=' xxx.log				# 打印3-9行，并显示行号
sed -n '/root/p' xxx.log					# 显示包含root的行
sed -n '/hhh/,/omc/p' xxx.log				# 显示包含"hhh"的行到包含"omc"的行之间的行

# 文本替换
sed -i 's/root/world/g' xxx.log				# 用world 替换xxx.log文件中的root; s==search  查找并替换, g==global  全部替换, -i: implace

# 文本插入
sed '1,4i hahaha' xxx.log								# 在文件第一行和第四行的每行下面添加hahaha
sed -e '1i happy' -e '$a new year' xxx.log  			#【界面显示】在文件第一行添加happy,文件结尾添加new year
sed -i -e '1i happy' -e '$a new year' xxx.log			#【真实写入文件】在文件第一行添加happy,文件结尾添加new year

# 文本删除
sed  '3,9d' xxx.log									# 删除第3到第9行,只是不显示而已
sed '/hhh/,/omc/d' xxx.log							# 删除包含"hhh"的行到包含"omc"的行之间的行
sed '/omc/,10d' xxx.log								# 删除包含"omc"的行到第十行的内容

# 与find结合
find . -name  "*.txt" |xargs   sed -i 's/hhhh/\hHHh/g'
find . -name  "*.txt" |xargs   sed -i 's#hhhh#hHHh#g'
find . -name  "*.txt" -exec sed -i 's/hhhh/\hHHh/g' {} \;
find . -name  "*.txt" |xargs cat
```

更多请参考：[Linux sed 命令](https://www.runoob.com/linux/linux-comm-sed.html) 或者 [Linux sed命令详解](https://www.cnblogs.com/ftl1012/p/9250171.html)





## 文件操作

### 文件监听 - tail

最常用的`tail -f filename`

```bash
# 基本使用
tail -f xxx.log					# 循环监听文件
tail -300f xxx.log				# 倒数300行并追踪文件
tail +20 xxx.log				# 从第 20 行至文件末尾显示文件内容

# tailf使用
tailf xxx.log					# 等同于tail -f -n 10 打印最后10行，然后追踪文件
```

tail -f 与tail F 与tailf三者区别

```bash
`tail  -f `  等于--follow=descriptor，根据文件描述进行追踪，当文件改名或删除后，停止追踪

`tail -F` 等于 --follow=name ==retry，根据文件名字进行追踪，当文件改名或删除后，保持重试，当有新的文件和他同名时，继续追踪

`tailf` 等于tail -f -n 10（tail -f或-F默认也是打印最后10行，然后追踪文件），与tail -f不同的是，如果文件不增长，它不会去访问磁盘文件，所以tailf特别适合那些便携机上跟踪日志文件，因为它减少了磁盘访问，可以省电
```

tail的参数

```bash
-f				循环读取
-q				不显示处理信息
-v				显示详细的处理信息
-c<数目>		   显示的字节数
-n<行数>		   显示文件的尾部 n 行内容
--pid=PID		与-f合用,表示在进程ID,PID死掉之后结束
-q, --quiet, --silent			从不输出给出文件名的首部
-s, --sleep-interval=S			与-f合用,表示在每次反复的间隔休眠S秒
```





### 文件查找 - find

```bash
sudo -u admin find /home/admin /tmp /usr -name \*.log(多个目录去找)
find . -iname \*.txt(大小写都匹配)
find . -type d(当前目录下的所有子目录)
find /usr -type l(当前目录下所有的符号链接)
find /usr -type l -name "z*" -ls(符号链接的详细信息 eg:inode,目录)
find /home/admin -size +250000k(超过250000k的文件，当然+改成-就是小于了)
find /home/admin f -perm 777 -exec ls -l {} \; (按照权限查询文件)
find /home/admin -atime -1  1天内访问过的文件
find /home/admin -ctime -1  1天内状态改变过的文件    
find /home/admin -mtime -1  1天内修改过的文件
find /home/admin -amin -1  1分钟内访问过的文件
find /home/admin -cmin -1  1分钟内状态改变过的文件    
find /home/admin -mmin -1  1分钟内修改过的文件
```





### pgm

批量查询vm-shopbase满足条件的日志

```bash
pgm -A -f vm-shopbase 'cat /home/admin/shopbase/logs/shopbase.log.2017-01-17|grep 2069861630'
```





## 查看网络和进程

### 查看所有网络接口的属性

```bash
[root@pdai.tech ~]# ifconfig

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.31.165.194  netmask 255.255.240.0  broadcast 172.31.175.255
        ether 00:16:3e:08:c1:ea  txqueuelen 1000  (Ethernet)
        RX packets 21213152  bytes 2812084823 (2.6 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 25264438  bytes 46566724676 (43.3 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 502  bytes 86350 (84.3 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 502  bytes 86350 (84.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```



### 查看防火墙设置

```bash
[root@pdai.tech ~]# iptables -L

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```



### 查看路由表

```bash
[root@pdai.tech ~]# route -n

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.31.175.253  0.0.0.0         UG    0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
172.31.160.0    0.0.0.0         255.255.240.0   U     0      0        0 eth0
```



### netstat

查看所有监听端口

```bash
[root@pdai.tech ~]# netstat -lntp

Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name  
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      970/nginx: master p
tcp        0      0 0.0.0.0:9999            0.0.0.0:*               LISTEN      1249/Java         
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      970/nginx: master p
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1547/sshd         
tcp6       0      0 :::3306                 :::*                    LISTEN      1894/mysqld       
```

查看所有已经建立的连接

```bash
[root@pdai.tech ~]# netstat -antp

Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      970/nginx: master p
tcp        0      0 0.0.0.0:9999            0.0.0.0:*               LISTEN      1249/Java
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      970/nginx: master p
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1547/sshd
tcp        0      0 172.31.165.194:53874    100.100.30.25:80        ESTABLISHED 18041/AliYunDun
tcp        0     64 172.31.165.194:22       xxx.194.1.200:2649      ESTABLISHED 32516/sshd: root@pt
tcp6       0      0 :::3306                 :::*                    LISTEN      1894/m
```

查看当前连接

```bash
[root@pdai.tech ~]# netstat -nat|awk  '{print $6}'|sort|uniq -c|sort -rn

      5 LISTEN
      2 ESTABLISHED
      1 Foreign
      1 established)
```

查看网络统计信息进程

```bash
[root@pdai.tech ~]# netstat -s

Ip:
    21017132 total packets received
    0 forwarded
    0 incoming packets discarded
    21017131 incoming packets delivered
    25114367 requests sent out
    324 dropped because of missing route
Icmp:
    18088 ICMP messages received
    692 input ICMP message failed.
    ICMP input histogram:
        destination unreachable: 4241
        timeout in transit: 19
        echo requests: 13791
        echo replies: 4
        timestamp request: 33
    13825 ICMP messages sent
    0 ICMP messages failed
    ICMP output histogram:
        destination unreachable: 1
        echo replies: 13791
        timestamp replies: 33
IcmpMsg:
        InType0: 4
        InType3: 4241
        InType8: 13791
        InType11: 19
        InType13: 33
        OutType0: 13791
        OutType3: 1
        OutType14: 33
Tcp:
    12210 active connections openings
    208820 passive connection openings
    54198 failed connection attempts
    9805 connection resets received
...
```

netstat 请参考这篇文章: [Linux netstat命令详解](https://www.cnblogs.com/ftl1012/p/netstat.html)



### 查看所有进程

```bash
[root@pdai.tech ~]# ps -ef | grep Java

root      1249     1  0 Nov04 ?        00:58:05 Java -jar /opt/tech_doc/bin/tech_arch-0.0.1-RELEASE.jar --server.port=9999
root     32718 32518  0 08:36 pts/0    00:00:00 grep --color=auto Java
```



### top

top除了看一些基本信息之外，剩下的就是配合来查询vm的各种问题了

```bash
# top -H -p pid

top - 08:37:51 up 45 days, 18:45,  1 user,  load average: 0.01, 0.03, 0.05
Threads:  28 total,   0 running,  28 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.7 us,  0.7 sy,  0.0 ni, 98.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1882088 total,    74608 free,   202228 used,  1605252 buff/cache
KiB Swap:  2097148 total,  1835392 free,   261756 used.  1502036 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 1347 root      20   0 2553808 113752   1024 S  0.3  6.0  48:46.74 VM Periodic Tas
 1249 root      20   0 2553808 113752   1024 S  0.0  6.0   0:00.00 Java
 1289 root      20   0 2553808 113752   1024 S  0.0  6.0   0:03.74 Java
...
```



## 查看磁盘和内存相关

### 查看内存使用 - free -m

```bash
[root@pdai.tech ~]# free -m

              total        used        free      shared  buff/cache   available
Mem:           1837         196         824           0         816        1469
Swap:          2047         255        1792
```



### 查看各分区使用情况

```bash
[root@pdai.tech ~]# df -h

Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        909M     0  909M   0% /dev
tmpfs           919M     0  919M   0% /dev/shm
tmpfs           919M  452K  919M   1% /run
tmpfs           919M     0  919M   0% /sys/fs/cgroup
/dev/vda1        40G   15G   23G  40% /
tmpfs           184M     0  184M   0% /run/user/0
```



### 查看指定目录的大小

```bash
[root@pdai.tech ~]# du -sh

803M
```



### 查看内存总量

```bash
[root@pdai.tech ~]# grep MemTotal /proc/meminfo

MemTotal:        1882088 kB
```



### 查看空闲内存量

```bash
[root@pdai.tech ~]# grep MemFree /proc/meminfo

MemFree:           74120 kB
```



### 查看系统负载磁盘和分区

```bash
[root@pdai.tech ~]# grep MemFree /proc/meminfo

MemFree:           74120 kB
```



### 查看系统负载磁盘和分区

```bash
[root@pdai.tech ~]# cat /proc/loadavg

0.01 0.04 0.05 2/174 32751
```



### 查看挂接的分区状态

```bash
[root@pdai.tech ~]# mount | column -t

sysfs       on  /sys                             type  sysfs       (rw,nosuid,nodev,noexec,relatime)
proc        on  /proc                            type  proc        (rw,nosuid,nodev,noexec,relatime)
devtmpfs    on  /dev                             type  devtmpfs    (rw,nosuid,size=930732k,nr_inodes=232683,mode=755)
securityfs  on  /sys/kernel/security             type  securityfs  (rw,nosuid,nodev,noexec,relatime)
...
```



### 查看所有分区

```bash
[root@pdai.tech ~]# fdisk -l

Disk /dev/vda: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0008d73a

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    83884031    41940992   83  Linux
```



### 查看所有交换分区

```bash
[root@pdai.tech ~]# swapon -s

Filename                                Type            Size    Used    Priority
/etc/swap                               file    2097148 261756  -2
```



### 查看硬盘大小

```bash
[root@pdai.tech ~]# fdisk -l |grep Disk

Disk /dev/vda: 42.9 GB, 42949672960 bytes, 83886080 sectors
Disk label type: dos
Disk identifier: 0x0008d73a
```





## 查看用户和组相关

### 查看活动用户

```shell
[root@pdai.tech ~]# w

 08:47:20 up 45 days, 18:54,  1 user,  load average: 0.01, 0.03, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    xxx.194.1.200    08:32    0.00s  0.32s  0.32s -bash
```



### 查看指定用户信息

```bash
[root@pdai.tech ~]# id

uid=0(root) gid=0(root) groups=0(root)
```



### 查看用户登录日志

```bash
[root@pdai.tech ~]# last

root     pts/0        xxx.194.1.200    Fri Dec 20 08:32   still logged in
root     pts/0        xxx.73.164.60     Thu Dec 19 21:47 - 00:28  (02:41)
root     pts/0        xxx.106.236.255  Thu Dec 19 16:00 - 18:24  (02:23)
root     pts/1        xxx.194.3.173    Tue Dec 17 13:35 - 17:37  (04:01)
root     pts/0        xxx.194.3.173    Tue Dec 17 13:35 - 17:37  (04:02)
...
```



### 查看系统所有用户

```bash
[root@pdai.tech ~]# cut -d: -f1 /etc/passwd

root
bin
daemon
adm
...
```



### 查看系统所有组

```bash
cut -d: -f1 /etc/group
```



## 查看服务，模块和包相关

```bash
# 查看当前用户的计划任务服务
crontab -l 

# 列出所有系统服务
chkconfig –list 

# 列出所有启动的系统服务程序
chkconfig –list | grep on 

# 查看所有安装的软件包
rpm -qa 

# 列出加载的内核模块
lsmod 
```



## 查看系统，设备，环境信息

```bash
# 常用
env							# 查看环境变量资源
uptime						# 查看系统运行时间、用户数、负载
lsusb -tv					# 列出所有USB设备的linux系统信息命令
lspci -tv					# 列出所有PCI设备
head -n 1 /etc/issue		# 查看操作系统版本，是数字1不是字母L
uname -a					# 查看内核/操作系统/CPU信息的linux系统信息命令

# /proc/
cat /proc/cpuinfo			# 查看CPU相关参数的linux系统命令
cat /proc/partitions		# 查看linux硬盘和分区信息的系统信息命令
cat /proc/meminfo			# 查看linux系统内存信息的linux系统命令
cat /proc/version			# 查看版本，类似uname -r
cat /proc/ioports			# 查看设备io端口
cat /proc/interrupts		# 查看中断
cat /proc/pci				# 查看pci设备的信息
cat /proc/swaps				# 查看所有swap分区的信息
cat /proc/cpuinfo |grep "model name" && cat /proc/cpuinfo |grep "physical id"
```



## tsar

tsar是淘宝开源的的采集工具很好用, 将历史收集到的数据持久化在磁盘上，所以我们快速来查询历史的系统数据当然实时的应用情况也是可以查询的啦大部分机器上都有安装

```bash
tsar					## 可以查看最近一天的各项指标
tsar --live				## 可以查看实时指标，默认五秒一刷
tsar -d 20161218		## 指定查看某天的数据，貌似最多只能看四个月的数据
tsar --mem
tsar --load
tsar --cpu				## 当然这个也可以和-d参数配合来查询某天的单个指标的情况 
```

具体可以看这篇文章：[linux 淘宝开源监控工具tsa](https://www.jianshu.com/p/5562854ed901)





# [#](#Java问题排错) Java内存问题排查

> **内存泄漏**（memory leak）：在Java中如果不再使用一个对象，但是该对象依然在GC ROOT的引用链上，这个对象就不会被垃圾回收器回收，这种情况就称之为内存泄漏。 
>
> - PS：==内存泄漏绝大多数情况都是由堆内存泄漏引起的==。
>
> **内存溢出**：指的是内存的使用量超过了Java虚拟机可以分配的上限，最终产生了内存溢出OutOfMemory的错误。



**解决内存溢出的思路**：

1. **发现问题**：通过监控工具尽可能尽早地发现内存慢慢变大的现象。
2. **诊断原因**：通过分析内存快照或者在线分析方法调用过程，诊断问题产生的根源，定位到出现问题的源代码。
3. **修复问题**：尝试重现问题，如借助jmeter什么鬼之类的。之后修复，如源代码中的bug问题、技术方案不合理、业务设计不合理等等。
4. **验证测试**：在测试环境验证问题是否已经解决，最后发布上线。



**内存溢出产生的原因**：

1. **持续的内存泄漏**：内存泄漏持续发生，不可被回收同时不再使用的内存越来越多，就像滚雪球雪球越滚越大，最终内存被消耗完无法分配更多的内存取使用，导致内存溢出。

这种原因一般就是代码中的内存泄漏，所以一般在测试阶段就会被测试出来，如下示例：

- **不正确的`eauals()`和`hashcode()`**：定义新类时没有重写正确的equals()和hashCode()方法。在使用HashMap的场景下，如果使用这个类对象作为key，HashMap在判断key是否已经存在时会使用这些方法，如果重写方式不正确，会导致相同的数据被保存多份。

此种情况的解决方式：定义新实体类时记得重写这两个方法，且重写时使用“唯一标识”去区分不同对象，以及在使用HashMap时key使用实体的“唯一标识”。

- **非静态内部类和匿名内部类的错误使用**：非静态的内部类默认会持有外部类，尽管代码上不再使用外部类，所以如果有地方引用了这个非静态内部类，会导致外部类也被引用，垃圾回收时无法回收这个外部类。另外就是匿名内部类对象如果在非静态方法中被创建，会持有调用者对象，垃圾回收时无法回收调用者。

此种情况的解决方式：使用静态内部类和静态方法即可。

- **ThreadLocal的错误使用**：由于线程池中的线程不被回收导致的ThreadLocal内存泄漏。

如果仅仅使用手动创建的线程，就算没有调用ThreadLocal的remove方法清理数据，也不会产生内存泄漏。因为当线程被回收时，ThreadLocal也同样被回收。但是如果使用线程池就不一定了。

此种情况的解决方式：线程方法执行完，记得调用ThreadLocal中的remove方法清理对象。

- **静态变量的错误使用（很常见哦）**：大量的数据在静态变量中被引用，但不再使用，就成为了内存泄漏。

如果大量的数据在静态变量中被长期引用，数据就不会被释放，如果这些数据不再使用，就成为了内存泄漏。

此种情况的解决方式：

一是：尽量减少将对象长时间的保存在静态变量中，如果不再使用，必须将对象删除（比如在集合中）或者将静态变量设置为null。 

二是：使用单例模式时，尽量使用懒加载，而不是立即加载。

三是：Spring的Bean中不要长期存放大对象，如果是缓存用于提升性能，尽量设置过期时间定期失效。

- **资源没有正常关闭**：由于资源没有调用`close()`方法正常关闭，”可能“导致内存泄漏。

连接和流这些资源会占用内存，如果使用完之后没有关闭，**这部分内存"不一定"会出现内存泄漏**，但是会导致close方法不被执行。

不一定的原因：如下列代码

```java
public static void lead() throws SQLException {	// 此方法执行完
    Startement stmt = null;
    Connection conn = DriverManager.getConnection(DB_URL, USER, PASS);
    // 则Connection不在GC Roots引用链上，就会被回收，从而conn关联的Startement、RestultSet这些对象也会被回收，从而不会造成内存泄漏
    
    stmt = conn.createStatement();
    String sql = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
    RestultSet rs = stmt.executeQuery(sql);
    
    while(rs.next()){
        // ...............
    }
    
    // 最终没有关闭流管道
}
```

此种情况的解决方式：在finally块中关闭不再使用的资源。另外就是从 Java 7 开始，可以使用try-with-resources语法可以用于自动关闭资源。

![image-20240106215225651](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106215224113-1647299824.png)

2. **并发请求问题**：用户通过发送请求向Java应用获取数据，正常情况下Java应用将数据返回之后，这部分数据就可以在内存中被释放掉。但是由于用户的并发请求量有可能很大，同时处理数据的时间很长，导致大量的数据存在于内存中，最终超过了内存的上限，导致内存溢出。









## 发现问题

### Java 调试入门工具

> 声明：此节内容主要整理自[@pdai：调试排错 - Java 问题排查之工具单](https://www.pdai.tech/md/java/jvm/java-jvm-debug-tools-list.html)，在此基础上做了改动。

#### jps：查看当前进程

> jps是JDK提供的一个查看当前Java进程的小工具， 可以看做是Java Virtual Machine Process Status Tool的缩写

jps常用命令

```bash
jps								# 显示进程的ID 和 类的名称
jps –l							# 输出完全的包名，应用主类名，jar的完全路径名 
jps –v							# 输出JVM参数
jps –q							# 显示Java进程号
jps -m							# main 方法
jps -l xxx.xxx.xx.xx			# 远程查看


jps参数：

    -q							# 仅输出VM标识符，不包括Classname,jar name,arguments in main method 
    -m							# 输出main method的参数 
    -l							# 输出完全的包名，应用主类名，jar的完全路径名 
    -v							# 输出JVM参数 
    -V							# 输出通过flag文件传递到JVM中的参数(.hotspotrc文件或-XX:Flags=所指定的文件 
    -Joption					# 传递参数到vm,例如:-J-Xms512m
```



jps原理

> Java程序在启动以后，会在Java.io.tmpdir指定的目录下，就是临时文件夹里，生成一个类似于hsperfdata_User的文件夹，这个文件夹里（在Linux中为/tmp/hsperfdata_{userName}/），有几个文件，名字就是Java进程的pid，因此列出当前运行的Java进程，只是把这个目录里的文件名列一下而已。至于系统的参数什么，就可以解析这几个文件获得

更多请参考 [jps - Java Virtual Machine Process Status Tool](https://docs.oracle.com/Javase/1.5.0/docs/tooldocs/share/jps.html)





#### jstack：线程的栈信息

> jstack是JDK自带的线程堆栈分析工具，使用该命令可以查看或导出 Java 应用程序中线程堆栈信息

jstack常用命令:

```bash
# 基本
jstack 2815

# Java和native c/c++框架的所有栈信息
jstack -m 2815

# 额外的锁信息列表，查看是否死锁
jstack -l 2815




jstack参数：

    -l			# 长列表. 打印关于锁的附加信息,例如属于Java.util.concurrent 的 ownable synchronizers列表.

    -F			# 当’jstack [-l] pid’没有相应的时候强制打印栈信息

    -m			# 打印Java和native c/c++框架的所有栈信息.

    -h | -help	# 打印帮助信息
```



#### jinfo：查看参数信息

> jinfo 是 JDK 自带的命令，可以用来查看正在运行的 Java 应用程序的扩展参数，包括Java System属性和JVM命令行参数；也可以动态的修改正在运行的 JVM 一些参数。当系统崩溃时，jinfo可以从core文件里面知道崩溃的Java应用程序的配置信息

jinfo常用命令:

```bash
# 输出当前 JVM 进程的全部参数和系统属性
jinfo 2815

# 输出所有的参数
jinfo -flags 2815

# 查看指定的 JVM 参数的值
jinfo -flag PrintGC 2815

# 开启/关闭指定的JVM参数
jinfo -flag +PrintGC 2815

# 设置flag的参数
jinfo -flag name=value 2815

# 输出当前 JVM 进行的全部的系统属性
jinfo -sysprops 2815



jinfo参数：

    no option				# 输出全部的参数和系统属性
    -flag name				# 输出对应名称的参数
    -flag [+|-]name			# 开启或者关闭对应名称的参数
    -flag name=value		# 设定对应名称的参数
    -flags					# 输出全部的参数
    -sysprops				# 输出系统属性
```

更多请参考：[JVM 性能调优工具之 jinfo](https://www.jianshu.com/p/8d8aef212b25)





#### jmap：生成dump文件 和 查看堆情况

> JDK自带的jmap是一个多功能的命令。它可以生成 Java 程序的 dump 文件， 也可以查看堆内对象示例的统计信息、查看 ClassLoader 的信息以及 finalizer 队列。
>
> - PS：dump文件是什么去这里：https://www.cnblogs.com/toSeeMyDream/p/7151635.html

两个用途

```bash
# 查看堆的情况
jmap -heap 2815

# dump
jmap -dump:live,format=b,file=/tmp/heap2.bin 2815	# 只保留存活对象（GC Roots引用链上的对象）
jmap -dump:format=b,file=/tmp/heap3.bin 2815		# 全部对象（即在GC Roots引用链上和不在的都保存）
# 使用情况：有时显示内存很高，但使用上面一种生成存活对象的快照时内存变很小了
# 原因是生成dump时会进行Full GC，所以不利于分析，从而采用下面这种，让快照包含全部对象。


# 查看堆的占用
jmap -histo 2815 | head -10



jmap参数：

    no option					# 查看进程的内存映像信息,类似 Solaris pmap 命令
    heap						# 显示Java堆详细信息
    histo[:live]				# 显示堆中对象的统计信息
    clstats						# 打印类加载器信息
    finalizerinfo				# 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
    dump:<dump-options>			# 生成堆转储快照
    F							# 当-dump没有响应时，使用-dump或者-histo参数. 在这个模式下,live子参数无效.
    help						# 打印帮助信息
    J<flag>						# 指定传递给运行jmap的JVM的参数
```

更多请参考：

1. [JVM 性能调优工具之 jmap](https://www.jianshu.com/p/a4ad53179df3) 
2. [jmap - Memory Map](https://docs.oracle.com/Javase/1.5.0/docs/tooldocs/share/jmap.html)





#### [#](#jstat：总结垃圾回收统计) jstat：总结垃圾回收统计

> 缺点：无法精确到GC产生的时间，只能用于判断GC是否存在问题。

jstat参数众多，但是使用一个就够了。

```bash
# 命令格式：jstat -gcutil pid interval(间隔，单位ms)
jstat -gcutil 2815 1000
```

- C代表Capacity容量，U代表Used使用量

- S – 幸存者区，E – 伊甸园区，O – 老年代，M – 元空间
- YGC、YGT：年轻代GC次数和GC耗时（单位：秒）
- FGC、FGCT：Full GC次数和Full GC耗时
- GCT：GC总耗时



更多请参考：[Java的jstat命令使用详解](https://blog.csdn.net/heihaozi/article/details/123497656)





#### jdb：预发debug

jdb可以用来预发debug，假设你预发的Java_home是/opt/Java/，远程调试端口是8000，那么

```bash
jdb -attach 8000
```

出现以上代表jdb启动成功。后续可以进行设置断点进行调试。

具体参数可见oracle官方说明[jdb - The Java Debugger](http://docs.oracle.com/Javase/7/docs/technotes/tools/windows/jdb.html)







### Linux：Top命令

> top除了看一些基本信息之外，剩下的就是配合来查询vm的各种问题了。
>
> 缺点：只能查看最基础的进程信息，无法查看到每个部分的内存占用（堆、方法区、堆外）

top命令是Linux下用来查看系统信息的一个命令，它提供给我们去实时地去查看系统的资源，比如执行时的进程、线程和系统参数等信息。

> 关于下列两个概念的说明：
>
> 1. **常驻内存**：当前进程总的使用了多少内存。
>
> - PS：常驻内存包含了“共享内存”，所以当前进程真正使用的内存是：常驻内存 - 共享内存。
>
> 2. **共享内存**：当前进程第三方依赖需要的内存。只加载一次，其他地方就可以用了，故而称为“共享”。

```bash
# top -H -p pid

top - 08:37:51 up 45 days, 18:45,  1 user,  load average: 0.01, 0.03, 0.05
Threads:  28 total,   0 running,  28 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.7 us,  0.7 sy,  0.0 ni, 98.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1882088 total（堆的总内存）,    74608 free（空闲内存，若此值极小则说明本服务器的程序有问题）,   202228 used（已使用内存）,  1605252 buff/cache（缓存）		# 关注点
KiB Swap:  2097148 total,  1835392 free,   261756 used.  1502036 avail Mem


# %CPU		当前进程对CPU的使用率				 若此值长期保持很高，则需要关注程序请求量是否过大，或出现死循环之类的
# %MEM		当前进程占总内存的比率					若上面的 free值很小，而此值很高，则可以确定系统内存不足就是当前进程所造成的
# TIME+		当前进程自启动以来所消耗的CPU累计时间
# COMMAND	启动命令
  PID USER      PR  NI    VIRT（虚拟内存）    RES（常驻内存）    SHR（共享内存） S %CPU %MEM     TIME+ COMMAND
 1347 root      20   0 2553808 				113752   		 1024 			S  0.3  6.0  48:46.74 VM Periodic Tas
 1249 root      20   0 2553808 				113752   		 1024 			S  0.0  6.0   0:00.00 Java
 1289 root      20   0 2553808 				113752   		 1024 			S  0.0  6.0   0:03.74 Java
...
```







### JConsole：本地+远程监控

> Jconsole （Java Monitoring and Management Console），JDK自带的基于JMX的可视化监视、管理工具 官方文档可以参考[这里](https://docs.oracle.com/Javase/8/docs/technotes/guides/management/jconsole.html)
>
> 路径：JDK\bin\jconsole.exe



本地连接 或 远程连接：

> 注：远程连接在“测试环境”用就可以了，别在线上环境用。

![image-20240106195445531](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106195445077-496683861.png)



![image-20240106195600896](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106195559898-1219779873.png)







### VisualVM：本地+远程监控

> VisualVM 是一款免费的，集成了多个 JDK 命令行工具的可视化工具，整合了命令行 JDK 工具和轻量级分析功能，它能为您提供强大的分析能力，对 Java 应用程序做性能分析和调优这些功能包括生成和分析海量数据、跟踪内存泄漏、监控垃圾回收器、执行内存和 CPU 分析，同时它还支持在 MBeans 上进行浏览和操作。
>
> 注：这款软件在Oracle JDK 6~8 中发布（路径：JDK\bin\jvisualvm.exe），但是在 Oracle JDK 9 之后不在JDK安装目录下需要单独下载。下载地址：https://visualvm.github.io/
>
> 优点：支持Idea插件，开发过程中也可以使用。
>
> 缺点：对大量集群化部署的Java进程需要手动进行管理。



本地连接：JDK\bin\jvisualvm.exe的方式，这种是中文版

![image-20240106200430766](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106200429361-1686866202.png)

IDEA插件的方式：

![image-20240106200638281](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106200636862-1051496214.png)

以下两种方式均可启动VisualVM

![image-20240106200750834](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106200759167-968268559.png)

远程连接：

> 注：==只可用于“测试环境”，不可用于“生产环境”==。因为操作VisualVM中提供的功能时会停掉线程，从而影响用户。

1. 服务器中开启JMX远程连接

```bash
java -jar \
-Djava.rmi.server.hostname=xxxxxxxx \					# 配置主机名	就是服务器ip
-Dcom.sun.management.jmxremote \						# 开启JMX远程连接
-Dcom.sun.management.jmxremote.port=xxxx \				# 设置连接的端口号
-Dcom.sun.management.jmxremote.ssl=false \				# 关闭ssl连接
-Dcom.sun.management.jmxremote.authenticate=false \		# 关闭身份验证
xxxxx.jar												# 要启动的服务jar包
```

2. 使用VisualVM建立远程连接

![image-20240106202155366](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106202154183-1465862557.png)



### Arthas Tunnel

> 官网地址：https://arthas.aliyun.com/doc/tunnel.html
>
> 优点：
>
> 1. 功能强大，不止于监控基础的信息，还能监控单个方法的执行耗时等细节内容。
> 2.  支持应用的集群管理.



大概流程如下：

![image-20240106202700733](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106202659094-425000391.png)

大概操作流程如下：

1. 添加依赖（目前仅支持Spring Boot2.x版本），在配置文件中添加tunnel服务端的地址，便于tunnel去监控所有的程序。

```xml
<dependency>
    <groupId>com.taobao.arthas</groupId>
    <artifactId>arthas-spring-boot-starter</artifactId>
    <version>3.7.1</version>
</dependency>
```

YAML配置：

```yaml
arthas:
  # tunnel部署的地址
  tunnel-server: ws://localhost:7777/ws
  # tunnel显示的应用名称：注册到tunnel上的每个服务都要有个名称
  app-name: ${spring.application.name}
  # arthas http访问的端口 和 远程连接的端口		这两个端口不可重复
  http-port: 8888
  telnet-port: 9999
```

2. 将tunnel服务端程序部署在某台服务器上并启动

> 注：需要去官网[下载](https://github.com/alibaba/arthas/releases) tunnel的jar包丢在服务器目录中，

```bash
# 启动命令
nohup java -jar \										# nohup 即no hang up（不挂起），后台不挂断
-Darthas.detail.pages=true \							# 打开可视化页面	注：这个页面占用的端口是80
arthas-tunnel-server-下载的某版本-fatjar.jar $			# 别忘了有个 $	即：将这个任务放到后台执行
```

页面网址：部署tunnel的ip:8080/apps.html

3. 启动Java程序，然后再上一步的页面中就可以看到对应的应用名称了。打开tunnel的服务端页面，查看所有的进程列表，并选择进程（应用名称）就可进入arthas进行arthas的操作。

> 排错：在arthas-tunnel-server-下载的某版本-fatjar.jar所在的目录中有一个nohup.out文件，打开即可排错，如：有些服务没注册上来之类的。





### Eclipse Memory Analyzer (MAT)

> ==这玩意儿可以说在开发中都会接触到，所以需要好好了解一下==。
>
> MAT 是一种快速且功能丰富的 Java 堆分析器，可帮助你发现内存泄漏并减少内存消耗。MAT在的堆内存分析问题使用极为广泛，需要重点掌握。
>
> 可以在[这里](https://eclipse.dev/mat/download/previous//)下载， 官方文档可以看[这里](http://help.eclipse.org/latest/index.jsp?topic=/org.eclipse.mat.ui.help/welcome.html)
>
> **提示1**：启动时可能会提示某某版本的JDK不支持，需要某某版本或以上，那安装对应的JDK版本，然后将其bin目录的路径放到path配置中即可，但：建议将此版本配置移到最上面或比其他版本的JDK更靠上
>
> **提示2**：JDK8对应的MAT版本是1.11.0

先来了解三个东西：也是MAT的原理

1. **支配树（Dominator Tree）**：MAT提供了支配树的对象图。**支配树展示的是对象实例间的支配关系**。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106221447907-1599564403.png" alt="image-20240106221449391" />

如上图所示：

- 对于B来说：B引用了A，而B并没有再引用其他的（即：到B只有一条路，A ->B），所以就是说：A支配B（如右图所示）。
- 而C是同理，对于D和F来说（用D来举例）：D引用了B，而B引用了A；同时D引用了C，而C引用了A（所以是两条线），但归根到底就是A支配了D。
- 其他E、F也是和D是同理分析的。



2. **深堆（Retained Heap）和浅堆（Shallow Heap）**

> **浅堆(Shallow Heap）**：支配树中对象本身占用的空间。 
>
> **深堆（Retained Heap）**：支配树中对象的子树就是所有被该对象支配的内容，这些内容组成了对象的深堆（Retained Heap），也称之为保留集（ Retained Set ） 。**深堆的大小表示该对象如果可以被回收，能释放多大的内存空间**。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106222616825-1479814000.png" alt="image-20240106222618607" />

如上图所示：在这个支配树中，对于C这个对象来说

- 这个对象本身占用的空间就是"浅堆"。
- C这个对象 以及 它的子树 所组成的空间大小就是深堆，若C对象被回收，那能够回收的空间大小就是：C对象本身+其子树E对象 这二者的总空间大小。



> MAT内存泄漏检测的原理：MAT就是根据支配树，从叶子节点向根节点遍历，如果发现深堆的大小超过整个堆内存的一定比例阈值，就会将其标记成内存泄漏的“嫌疑对象”。





#### 使用MAT发现问题

1. 当堆内存溢出时，可以在堆内存溢出时将整个堆内存保存下来，生成内存快照(Heap Profile )文件。

> 使用内存快照的目的：找出是程序哪里引发的问题、定位到问题出现的地方。

生成内存快照的Java虚拟机参数：

```bash
-XX:+HeapDumpOnOutOfMemoryError				发生OutOfMemoryError错误时，自动生成hprof内存快照文件

-XX:HeapDumpPath=<path>						指定hprof文件的输出路径
```

2.  使用MAT打开hprof文件（file -> open head dump），并选择内存泄漏检测功能（Leak Suspects Report 即内存泄漏检测报告），MAT会自行根据内存快照中保存的数据分析内存泄漏的根源。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106231453019-531139479.png" alt="image-20240106231454300" />



> 服务器中导出运行中系统的内存快照的简单方式：场景为内存在持续增长，但未发生内存泄漏，所以上面的`-XX:+HeapDumpOnOutOfMemoryError`就不能用。
>

1. 通过JDK自带的jmap命令导出，格式为：

```bash
# 生成存活对象的内存快照（GC Roots引用链上的对象）
jmap -dump:live,format=b,file=文件路径和文件名 进程ID

# 进程ID获取方式
ps -ef|grep java
```

2. 通过arthas的heapdump命令导出，格式为：

```bash
heapdump --live 文件路径\文件名
```

> 导出的dump文件还可以直接使用在线工具 [HeapHero](https://heaphero.io/) 打开来分析。



> 生成的堆内存报告很大怎么办？

机器内存范围之内的快照文件，直接使用MAT打开分析即可。

但是经常会遇到生成的快照文件很大，要下载到本地来也要很久。此时就需要下载服务器操作系统对应的MAT。下载地址：https://eclipse.dev/mat/downloads.php

然后将下载的对应版本MAT丢在某服务器中，如：Linux中。

> 注意：服务器中放MAT的目录记得将读写权限打开。

之后通过MAT中的脚本生成分析报告：生成的报告就是像上面那种静态页面

> 注意：默认MAT分析时只使用了1G的堆内存，如果快照文件超过1G，需要修改MAT目录下的MemoryAnalyzer.ini配置文件调整最大堆内存（`-Xmx值`）。

```bash
# 生成之后，在快照文件路径中会有几个压缩包，对应：内存泄漏报告、系统总览图、组件，下载自己需要的压缩包到本地即可
./ParseHeapDump.sh 快照文件路径 org.eclipse.mat.api:suspects org.eclipse.mat.api:overview org.eclipse.mat.api:top_components
```

最后将分析报告下载到本地，打开即可（一般有一个index.html）。





> 涉及到SpringMVC时，怎么定位到是哪个接口导致的问题？

1. 生成内存快照，使用MAT打开
2. 打开支配树，使用深堆（Retained Heap）排序，找到当前执行线程，如下面的taskThread，随便打开一个即可。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106235928191-1330363652.png" alt="image-20240106235928972" />

3. 找到当前线程正在执行的方法是哪一个。即找handleMethod，右键选择list objects（当前对象关联的对象） -> outgoing references（当前对象引用了哪些对象）

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107000727455-679510651.png" alt="image-20240107000728753" />

4. 找到description，这里就可找到到底是哪个controller的哪个方法导致的问题。
5. 然后将本地代码的Java虚拟机参数弄成和服务器一样，之后重现问题（借助压测jmeter什么鬼之类的）。
6. 最后结合前面内存快照得到的原因，解决问题，验证测试即可。









### 在线定位问题：Arthas之stack命令 和 btrace工具

诊断和解决问题一般有两种方案：离线分析（即生成内存快照分析）、在线定位。

> 内存快照分析：

优点：有完整的内存快照，从而能更准确地判断出问题的原因。

缺点：

- 内存较大时，生成内存快照较慢，这个过程会影响用户的使用。
- 通过MAT分析内存快照，至少要准备 1.5 - 2倍大小的内存空间。



> 在线定位

优点：无需生成内存快照，整个过程对用户的影响“较小”。

缺点：

- 无法查看到详细的内存信息。
- 需要具备一定的经验。而且一般还需要借助另外的工具（本章节使用arthas的stack命令 和 btrace工具）。



> Arthas的stack在线定位大致思路

1. 将内存中存活对象以直方图的形式保存到文件中，这个过程会影响用户的时间，但是时间比较短暂。使用命令如下：

```bash
jmap -histo:live 进程ID > 文件路径/文件名		# 表示：将 > 符号左边的内容 输出到 右边这个路径中
```

2. 查看直方图，分析内存占用最多的对象（直方图是排好序的），一般这些对象就是造成内存泄漏的原因。
3. 使用arthas的 [stack](https://arthas.aliyun.com/doc/stack.html) 命令，追踪第2步中分析的对象创建的方法被调用的调用路径，找到对象创建的根源。

使用命令如下：假设2中分析出来的对象是UserEntity

> 注意：别忘了把Arthas的jar包上传到服务器目录中，不然下面的命令能用个毛线。

```bash
stack com.zixieqing.jvm.entity.UserEntity -n 1	# 意思：输出1次com.zixieqing.jvm.entity.UserEntity这个类的所有方法的调用路径
```

通过上面的方式就可以找到是：哪个类那个方法哪一行了，然后尝试重现问题，修复问题、验证测试即可。





#### btrace工具

> btrace是一个在Java 平台上执行的追踪工具，可以有效地用于线上运行系统的方法追踪，具有侵入性小、对性能的影响微乎其微等特点。是生产环境&预发的排查问题大杀器。项目中可以使用btrace工具，实现定制化，打印出方法被调用的栈信息等等。

使用方法：btrace 具体可以参考这里：https://github.com/btraceio/btrace

1. 下载btrace工具， 官方地址：https://github.com/btraceio/btrace/releases/latest
2. 编写btrace脚本，通常是一个Java文件。如下两个示例：

编写时为了有提示和提供对应注解方法且不报错，可以加入如下依赖：路径改为自己下载的本地btrace

```xml
<dependency>
    <groupId>org.openjdk.btrace</groupId>
    <artifactId>btrace-agent</artifactId>
    <version>${btrace.version}</version>
    <scope>system</scope>
    <systemPath>D:Install\btrace-v2.2.4-bin\libs\btrace-agent.jar</systemPath>
</dependency>
<dependency>
    <groupId>org.openjdk.btrace</groupId>
    <artifactId>btrace-boot</artifactId>
    <version>${btrace.version}</version>
    <scope>system</scope>
    <systemPath>D:Install\btrace-v2.2.4-bin\libs\btrace-boot.jar</systemPath>
</dependency>
<dependency>
    <groupId>org.openjdk.btrace</groupId>
    <artifactId>btrace-client</artifactId>
    <version>${btrace.version}</version>
    <scope>system</scope>
    <systemPath>D:Install\btrace-v2.2.4-bin\libs\btrace-client.jar</systemPath>
</dependency>
```

- 查看当前谁调用了ArrayList的add方法，同时只打印当前ArrayList的size大于500的线程调用栈。编写如下Java代码

```java
@BTrace
public class TracingAdd {
    
    @OnMethod(clazz = "Java.util.ArrayList", method="add", 
          location = @Location(value = Kind.CALL, clazz = "/./", method = "/./")
         )
    public static void m(@ProbeClassName String probeClass, @ProbeMethodName String probeMethod, 
                         @TargetInstance Object instance, @TargetMethodOrField String method) {

        if(getInt(field("Java.util.ArrayList", "size"), instance) > 479){
            println("check who ArrayList.add method:" + probeClass + "#" + probeMethod  + ", method:" + method + ", size:" + getInt(field("Java.util.ArrayList", "size"), instance));
            jstack();
            println();
            println("===========================");
            println();
        }
	}
}
```

- 监控当前服务方法被调用时返回的值以及请求的参数

```java
@BTrace
public class TaoBaoNav {
    @OnMethod(clazz = "com.taobao.sellerhome.transfer.biz.impl.C2CApplyerServiceImpl", method="nav", 
          location = @Location(value = Kind.RETURN)
         )
    public static void mt(long userId, int current, int relation, 
                          String check, String redirectUrl, @Return AnyType result) {

        println("parameter# userId:" + userId + ", current:" + current + ", relation:" + relation + ", check:" + check + ", redirectUrl:" + redirectUrl + ", result:" + result);
    }
}
```

3. 将btrace工具和脚本上传到服务器，在服务器上执行如下格式的命令：

> 注意：需要配置环境变量BTRACE_HOME，和配置JDK是一样的。

```bash
btrace 进程ID 脚本文件名
```

4. 观察执行结果。



> 上面示例看起来懵的话，直接去看这个示例：https://www.cnblogs.com/wei-zw/p/9502274.html





### IDEA本地调试和远程调试

> 直接去这里：[IDEA本地调试和远程调试](https://github.com/zixq-stack/Java-Note/blob/master/Java/IDEA%20%E6%9C%AC%E5%9C%B0%E8%B0%83%E8%AF%95%E5%92%8C%E8%BF%9C%E7%A8%8B%E8%B0%83%E8%AF%95.md)



# Java应用在线调试Arthas整理



### Arthas简介

> 在学习Arthas之前，推荐先看后面的 [@美团技术团队：Java动态调试技术原理](#Java动态调试技术原理)，这样你会对它最底层技术有个了解。可以看下文中最后有个对比图：Greys(Arthas也是基于它做的二次开发)和Java-debug-tool



### Arthas是什么

`Arthas` 是Alibaba开源的Java诊断工具，深受开发者喜爱



### Arthas能解决什么问题

当你遇到以下类似问题而束手无策时，`Arthas`可以帮助你解决：

- 这个类从哪个 jar 包加载的? 为什么会报各种类相关的 Exception?
- 我改的代码为什么没有执行到? 难道是我没 commit? 分支搞错了?
- 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗?
- 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
- 是否有一个全局视角来查看系统的运行状况?
- 有什么办法可以监控到JVM的实时运行状态?

`Arthas`支持JDK 6+，支持Linux/Mac/Windows，采用命令行交互模式，同时提供丰富的 `Tab` 自动补全功能，进一步方便进行问题的定位和诊断





### Arthas资源推荐

- [官方在线教程(推荐)](https://arthas.gitee.io/)
- [常见问题](https://github.com/alibaba/arthas/issues?utf8=✓&q=label%3Aquestion-answered+)





### Arthas基于哪些工具发展而来

- [greys-anatomy](https://github.com/oldmanpushcart/greys-anatomy): Arthas代码基于Greys二次开发而来
- [termd](https://github.com/termd/termd): Arthas的命令行实现基于termd开发，是一款优秀的命令行程序开发框架
- [crash](https://github.com/crashub/crash): Arthas的文本渲染功能基于crash中的文本渲染功能开发，可以从[这里](https://github.com/crashub/crash/tree/1.3.2/shell)看到源码
- [cli](https://github.com/eclipse-vertx/vert.x/tree/master/src/main/Java/io/vertx/core/cli): Arthas的命令行界面基于vert.x提供的cli库进行开发
- [compiler](https://github.com/skalogs/SkaETL/tree/master/compiler) Arthas里的内存编绎器代码来源
- [Apache Commons Net](https://commons.apache.org/proper/commons-net/) Arthas里的Telnet Client代码来源
- `JavaAgent`：运行在 main方法之前的拦截器，它内定的方法名叫 premain ，也就是说先执行 premain 方法然后再执行 main 方法
- `ASM`：一个通用的Java字节码操作和分析框架。它可以用于修改现有的类或直接以二进制形式动态生成类。ASM提供了一些常见的字节码转换和分析算法，可以从它们构建定制的复杂转换和代码分析工具。ASM提供了与其他Java字节码框架类似的功能，但是主要关注性能。因为它被设计和实现得尽可能小和快，所以非常适合在动态系统中使用(当然也可以以静态方式使用，例如在编译器中)





### 同类工具有哪些

- BTrace
- 美团 Java-debug-tool
- [去哪儿Bistoury: 一个集成了Arthas的项目](https://github.com/qunarcorp/bistoury)
- [一个使用MVEL脚本的fork](https://github.com/XhinLiang/arthas)





## Arthas入门

### Arthas 上手前

推荐先在线使用下arthas：[官方在线教程(推荐)](https://arthas.gitee.io/doc/arthas-tutorials.html?language=cn&id=arthas-basics)





### Arthas 安装

下载`arthas-boot.jar`，然后用`Java -jar`的方式启动：

```bash
curl -O https://alibaba.github.io/arthas/arthas-boot.jar	# 也可以选择去官网下载jar然后丢到服务器中

Java -jar arthas-boot.jar
```



### Arthas 案例展示

> 官方命令列表：https://arthas.aliyun.com/doc/dashboard.html



#### Dashboard：当前系统的实时数据

![dashboard](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107160932837-1171725582.png)





#### Thread：查看当前线程信息、线程的堆栈

一目了然的了解系统的状态，哪些线程比较占cpu? 他们到底在做什么?

```bash
$ thread -n 3

"as-command-execute-daemon" Id=29 cpuUsage=75% RUNNABLE
    at sun.management.ThreadImpl.dumpThreads0(Native Method)
    at sun.management.ThreadImpl.getThreadInfo(ThreadImpl.Java:440)
    at com.taobao.arthas.core.command.monitor200.ThreadCommand$1.action(ThreadCommand.Java:58)
    at com.taobao.arthas.core.command.handler.AbstractCommandHandler.execute(AbstractCommandHandler.Java:238)
    at com.taobao.arthas.core.command.handler.DefaultCommandHandler.handleCommand(DefaultCommandHandler.Java:67)
    at com.taobao.arthas.core.server.ArthasServer$4.run(ArthasServer.Java:276)
    at Java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.Java:1145)
    at Java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.Java:615)
    at Java.lang.Thread.run(Thread.Java:745)

    Number of locked synchronizers = 1
    - Java.util.concurrent.ThreadPoolExecutor$Worker@6cd0b6f8

"as-session-expire-daemon" Id=25 cpuUsage=24% TIMED_WAITING
    at Java.lang.Thread.sleep(Native Method)
    at com.taobao.arthas.core.server.DefaultSessionManager$2.run(DefaultSessionManager.Java:85)

"Reference Handler" Id=2 cpuUsage=0% WAITING on Java.lang.ref.Reference$Lock@69ba0f27
    at Java.lang.Object.wait(Native Method)
    -  waiting on Java.lang.ref.Reference$Lock@69ba0f27
    at Java.lang.Object.wait(Object.Java:503)
    at Java.lang.ref.Reference$ReferenceHandler.run(Reference.Java:133)
```





#### jad：反编译已加载类

对类进行反编译:

```java
$ jad Javax.servlet.Servlet

ClassLoader:
+-Java.net.URLClassLoader@6108b2d7
  +-sun.misc.Launcher$AppClassLoader@18b4aac2
    +-sun.misc.Launcher$ExtClassLoader@1ddf84b8

Location:
/Users/xxx/work/test/lib/servlet-api.jar

/*
 * Decompiled with CFR 0_122.
 */
package Javax.servlet;

import Java.io.IOException;
import Javax.servlet.ServletConfig;
import Javax.servlet.ServletException;
import Javax.servlet.ServletRequest;
import Javax.servlet.ServletResponse;

public interface Servlet {
    public void init(ServletConfig var1) throws ServletException;

    public ServletConfig getServletConfig();

    public void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    public String getServletInfo();

    public void destroy();
}
```





#### mc：Java文件生成Class文件

Memory Compiler/内存编译器，编译`.Java`文件生成`.Class`

```bash
mc /tmp/Test.Java
```





#### redefine：加载外部Class文件

加载外部的`.Class`文件，redefine JVM已加载的类。推荐使用 [retransform](https://arthas.aliyun.com/doc/retransform.html) 命令

```bash
redefine /tmp/Test.Class
redefine -c 327a647b /tmp/Test.Class /tmp/Test\$Inner.Class
```





#### sc：查看JVM已加载的类信息

查看JVM中已经加载的类信息

```bash
$ sc -d org.springframework.web.context.support.XmlWebApplicationContext

 Class-info        org.springframework.web.context.support.XmlWebApplicationContext
 code-source       /Users/xxx/work/test/WEB-INF/lib/spring-web-3.2.11.RELEASE.jar
 name              org.springframework.web.context.support.XmlWebApplicationContext
 isInterface       false
 isAnnotation      false
 isEnum            false
 isAnonymousClass  false
 isArray           false
 isLocalClass      false
 isMemberClass     false
 isPrimitive       false
 isSynthetic       false
 simple-name       XmlWebApplicationContext
 modifier          public
 annotation
 interfaces
 super-Class       +-org.springframework.web.context.support.AbstractRefreshableWebApplicationContext
                     +-org.springframework.context.support.AbstractRefreshableConfigApplicationContext
                       +-org.springframework.context.support.AbstractRefreshableApplicationContext
                         +-org.springframework.context.support.AbstractApplicationContext
                           +-org.springframework.core.io.DefaultResourceLoader
                             +-Java.lang.Object
 Class-loader      +-org.apache.catalina.loader.ParallelWebappClassLoader
                     +-Java.net.URLClassLoader@6108b2d7
                       +-sun.misc.Launcher$AppClassLoader@18b4aac2
                         +-sun.misc.Launcher$ExtClassLoader@1ddf84b8
 ClassLoaderHash   25131501
```





#### stack：输出当前方法被调用的调用路径

输出当前方法被调用的调用路径

```bash
$ stack test.arthas.TestStack doGet

Press Ctrl+C to abort.
Affect(Class-cnt:1 , method-cnt:1) cost in 286 ms.
ts=2018-09-18 10:11:45;thread_name=http-bio-8080-exec-10;id=d9;is_daemon=true;priority=5;TCCL=org.apache.catalina.loader.ParallelWebappClassLoader@25131501
    @test.arthas.TestStack.doGet()
        at Javax.servlet.http.HttpServlet.service(HttpServlet.Java:624)
        at Javax.servlet.http.HttpServlet.service(HttpServlet.Java:731)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.Java:303)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.Java:208)
        at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.Java:52)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.Java:241)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.Java:208)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.Java:241)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.Java:208)
        at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.Java:220)
        at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.Java:110)
        ...
        at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.Java:169)
        at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.Java:103)
        at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.Java:116)
        at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.Java:451)
        at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.Java:1121)
        at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.Java:637)
        at org.apache.tomcat.util.net.JIoEndpoint$SocketProcessor.run(JIoEndpoint.Java:316)
        at Java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.Java:1142)
        at Java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.Java:617)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.Java:61)
        at Java.lang.Thread.run(Thread.Java:745)
```





#### Trace：方法内部调用路径，并输出方法路径上的每个节点上耗时

观察方法执行的时候哪个子调用比较慢:

![trace](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107160932799-391739634.png)





#### Watch：方法执行数据观测

观察方法 `test.arthas.TestWatch#doGet` 执行的入参，仅当方法抛出异常时才输出

```bash
$ watch test.arthas.TestWatch doGet {params[0], throwExp} -e

Press Ctrl+C to abort.
Affect(Class-cnt:1 , method-cnt:1) cost in 65 ms.
ts=2018-09-18 10:26:28;result=@ArrayList[
    @RequestFacade[org.apache.catalina.connector.RequestFacade@79f922b2],
    @NullPointerException[Java.lang.NullPointerException],
]
```





#### Monitor：方法执行监控

监控某个特殊方法的调用统计数据，包括总调用次数，平均rt，成功率等信息，每隔5秒输出一次

```bash
$ monitor -c 5 org.apache.dubbo.demo.provider.DemoServiceImpl sayHello

Press Ctrl+C to abort.
Affect(Class-cnt:1 , method-cnt:1) cost in 109 ms.
 timestamp            Class                                           method    total  success  fail  avg-rt(ms)  fail-rate
----------------------------------------------------------------------------------------------------------------------------
 2018-09-20 09:45:32  org.apache.dubbo.demo.provider.DemoServiceImpl  sayHello  5      5        0     0.67        0.00%

 timestamp            Class                                           method    total  success  fail  avg-rt(ms)  fail-rate
----------------------------------------------------------------------------------------------------------------------------
 2018-09-20 09:45:37  org.apache.dubbo.demo.provider.DemoServiceImpl  sayHello  5      5        0     1.00        0.00%

 timestamp            Class                                           method    total  success  fail  avg-rt(ms)  fail-rate
----------------------------------------------------------------------------------------------------------------------------
 2018-09-20 09:45:42  org.apache.dubbo.demo.provider.DemoServiceImpl  sayHello  5      5        0     0.43        0.00%
```





#### tt（Time Tunnel）：记录方法调用信息

记录方法调用信息，支持事后查看方法调用的参数，返回值，抛出的异常等信息，仿佛穿越时空隧道回到调用现场一般

```bash
$ tt -t org.apache.dubbo.demo.provider.DemoServiceImpl sayHello

Press Ctrl+C to abort.
Affect(Class-cnt:1 , method-cnt:1) cost in 75 ms.
 INDEX   TIMESTAMP            COST(ms)  IS-RET  IS-EXP   OBJECT         Class                          METHOD
-------------------------------------------------------------------------------------------------------------------------------------
 1000    2018-09-20 09:54:10  1.971195  true    false    0x55965cca     DemoServiceImpl                sayHello
 1001    2018-09-20 09:54:11  0.215685  true    false    0x55965cca     DemoServiceImpl                sayHello
 1002    2018-09-20 09:54:12  0.236303  true    false    0x55965cca     DemoServiceImpl                sayHello
 1003    2018-09-20 09:54:13  0.159598  true    false    0x55965cca     DemoServiceImpl                sayHello
 1004    2018-09-20 09:54:14  0.201982  true    false    0x55965cca     DemoServiceImpl                sayHello
 1005    2018-09-20 09:54:15  0.214205  true    false    0x55965cca     DemoServiceImpl                sayHello
 1006    2018-09-20 09:54:16  0.241863  true    false    0x55965cca     DemoServiceImpl                sayHello
 1007    2018-09-20 09:54:17  0.305747  true    false    0x55965cca     DemoServiceImpl                sayHello
 1008    2018-09-20 09:54:18  0.18468   true    false    0x55965cca     DemoServiceImpl                sayHello
```





#### Classloader：类加载器

了解当前系统中有多少类加载器，以及每个加载器加载的类数量，帮助您判断是否有类加载器泄露

```bash
$ Classloader

 name                                                  numberOfInstances  loadedCountTotal
 BootstrapClassLoader                                  1                  3346
 com.taobao.arthas.agent.ArthasClassloader             1                  1262
 Java.net.URLClassLoader                               2                  1033
 org.apache.catalina.loader.ParallelWebappClassLoader  1                  628
 sun.reflect.DelegatingClassLoader                     166                166
 sun.misc.Launcher$AppClassLoader                      1                  31
 com.alibaba.fastjson.util.ASMClassLoader              6                  15
 sun.misc.Launcher$ExtClassLoader                      1                  7
 org.jvnet.hk2.internal.DelegatingClassLoader          2                  2
 sun.reflect.misc.MethodUtil                           1                  1
```





#### Web Console

> 官方地址：https://arthas.aliyun.com/doc/web-console.html

![web console](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107160932781-252621157.png)





### Arthas 命令集

#### 基础命令

- help——查看命令帮助信息
- [cat](https://arthas.aliyun.com/doc/cat.html)——打印文件内容，和linux里的cat命令类似
- [grep](https://arthas.aliyun.com/doc/grep.html)——匹配查找，和linux里的grep命令类似
- [pwd](https://arthas.aliyun.com/doc/pwd.html)——返回当前的工作目录，和linux命令类似
- cls——清空当前屏幕区域
- session——查看当前会话的信息
- [reset](https://arthas.aliyun.com/doc/reset.html)——重置增强类，将被 Arthas 增强过的类全部还原，Arthas 服务端关闭时会重置所有增强过的类
- version——输出当前目标 Java 进程所加载的 Arthas 版本号
- history——打印命令历史
- quit——退出当前 Arthas 客户端，其他 Arthas 客户端不受影响
- stop / shutdown——关闭 Arthas 服务端，所有 Arthas 客户端全部退出
- [keymap](https://arthas.aliyun.com/doc/keymap.html)——Arthas快捷键列表及自定义快捷键
- [profiler](https://arthas.aliyun.com/doc/profiler.html)——使用[async-profiler](https://github.com/jvm-profiling-tools/async-profiler)生成火焰图。

> 注意：profiler这个命令无法在Windows中运行，可以在Linux或MacOS中运行。



#### JVM相关

- [dashboard](https://arthas.aliyun.com/doc/dashboard.html)——当前系统的实时数据面板
- [thread](https://arthas.aliyun.com/doc/thread.html)——查看当前 JVM 的线程堆栈信息
- [JVM](https://arthas.aliyun.com/doc/jvm.html)——查看当前 JVM 的信息
- [sysprop](https://arthas.aliyun.com/doc/sysprop.html)——查看和修改JVM的系统属性
- [sysenv](https://arthas.aliyun.com/doc/sysenv.html)——查看JVM的环境变量
- [vmoption](https://arthas.aliyun.com/doc/vmoption.html)——查看和修改JVM里诊断相关的option
- [logger](https://arthas.aliyun.com/doc/logger.html)——查看和修改logger
- [getstatic](https://arthas.aliyun.com/doc/getstatic.html)——查看类的静态属性
- [ognl](https://arthas.aliyun.com/doc/ognl.html)——执行ognl表达式
- [mbean](https://arthas.aliyun.com/doc/mbean.html)——查看 Mbean 的信息
- [heapdump](https://arthas.aliyun.com/doc/heapdump.html)——dump Java heap, 类似jmap命令的heap dump功能





#### Class/Classloader相关

- [sc](https://arthas.aliyun.com/doc/sc.html)——查看JVM已加载的类信息
- [sm](https://arthas.aliyun.com/doc/sm.html)——查看已加载类的方法信息
- [jad](https://arthas.aliyun.com/doc/jad.html)——反编译指定已加载类的源码
- [mc](https://arthas.aliyun.com/doc/mc.html)——内存编绎器，内存编绎`.Java`文件为`.Class`文件
- [redefine](https://arthas.aliyun.com/doc/redefine.html)——加载外部的`.Class`文件，redefine到JVM里
- [dump](https://arthas.aliyun.com/doc/dump.html)——dump 已加载类的 byte code 到特定目录
- [Classloader](https://arthas.aliyun.com/doc/classloader.html)——查看Classloader的继承树，urls，类加载信息，使用Classloader去getResource





#### [#](#monitor/watch/trace相关) monitor/watch/trace相关

> 请注意：这些命令，都通过字节码增强技术来实现的，会在指定类的方法中插入一些切面来实现数据统计和观测，因此在线上、预发使用时，请尽量明确需要观测的类、方法以及条件，诊断结束要执行 `shutdown` 或 `stop` 亦或将增强过的类执行 `reset` 命令

- [monitor](https://arthas.aliyun.com/doc/monitor.html)——方法执行监控
- [watch](https://arthas.aliyun.com/doc/watch.html)——方法执行数据观测
- [trace](https://arthas.aliyun.com/doc/trace.html)——方法内部调用路径，并输出方法路径上的每个节点上耗时
- [stack](https://arthas.aliyun.com/doc/stack.html)——输出当前方法被调用的调用路径
- [tt](https://arthas.aliyun.com/doc/tt.html)——方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测





#### options

- [options](https://arthas.aliyun.com/doc/options.html)——查看或设置Arthas全局开关



#### 管道

Arthas支持使用管道对上述命令的结果进行进一步的处理，如`sm Java.lang.String * | grep 'index'`

- grep——搜索满足条件的结果
- plaintext——将命令的结果去除ANSI颜色
- wc——按行统计输出结果



#### 后台异步任务

当线上出现偶发的问题，比如需要watch某个条件，而这个条件一天可能才会出现一次时，异步后台任务就派上用场了，详情请参考[这里](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/async.md)

- 使用 > 将结果重写向到日志文件，使用 & 指定命令是后台运行，session断开不影响任务执行(生命周期默认为1天)
- jobs——列出所有job
- kill——强制终止任务
- fg——将暂停的任务拉到前台执行
- bg——将暂停的任务放到后台执行







## [#](Arthas场景实战) Arthas场景实战

### 查看最繁忙的线程，以及是否有阻塞情况发生?

> 场景：我想看下最繁忙的线程，以及是否有阻塞情况发生? 常规查看线程，一般我们可以通过 top 等系统命令进行查看，但是那毕竟要很多个步骤，很麻烦

```bash
thread -n 3			# 查看最繁忙的三个线程栈信息
thread				# 以直观的方式展现所有的线程情况
thread -b			# 找出当前阻塞其他线程的线程
```



### 确认某个类是否已被系统加载?

> 场景：我新写了一个类或者一个方法，我想知道新写的代码是否被部署了?

```bash
# 即可以找到需要的类全路径，如果存在的话
sc *MyServlet

# 查看这个类所有的方法
sm pdai.tech.servlet.TestMyServlet *

# 查看某个方法的信息，如果存在的话
sm pdai.tech.servlet.TestMyServlet testMethod  
```



### 如何查看一个Class类的源码信息?

> 场景：我新修改的内容在方法内部，而上一个步骤只能看到方法，这时候可以反编译看下源码

```bash
# 直接反编译出Java 源代码，包含一此额外信息的
jad pdai.tech.servlet.TestMyServlet
```



### 重要：如何跟踪某个方法的返回值、入参.... ?

> 场景：我想看下我新加的方法在线运行的参数和返回值？或者是 在使用trace定位到性能较低的方法之后，使用watch命令监控该方法，获得更为详细的方法信息。

```bash
# 同时监控入参，返回值，及异常
watch pdai.tech.servlet.TestMyServlet testMethod "{params, returnObj, throwExp}" -e -x 2


# 参数
"{params, returnObj, throwExp}"			单引号、双引号都行。打印参数、返回值、抛出异常	可以任意选择，一般都只看params，从而模拟现象
‘#cost>毫秒值'						  	  只打印耗时超过该毫秒值的调用。
-e										在 函数异常之后 观察
-x 2									打印的结果中如果有嵌套（比如对象里有属性），最多只展开2层。允许设置的最大值为4。
```

> 对该命令注意前面说的 [monitor/watch/trace相关](#monitor/watch/trace相关) 注意事项，监控完了需要使用命令结束。

具体看 [watch](https://arthas.aliyun.com/doc/watch.html) 命令



### 如何看方法调用栈的信息?

> 场景：我想看下某个方法的调用栈的信息?

```bash
# stack 类名 方法名
stack pdai.tech.servlet.TestMyServlet testMethod
```

运行此命令之后需要即时触发方法才会有响应的信息打印在控制台上。

具体请看 [stack](https://arthas.aliyun.com/doc/stack.html) 命令



### [#](#重要：找到最耗时的方法调用?) 重要：找到最耗时的方法调用?

> 场景：testMethod这个方法入口响应很慢，如何找到最耗时的子调用？即方法嵌套，找出具体是哪个方法耗时。

```bash
# 执行的时候每个子调用的运行时长，可以找到最耗时的子调用
# trace 类名 方法名
trace pdai.tech.servlet.TestMyServlet testMethod


# 此命令此场景常用参数
--skipJDKMethod false		可以输出JDK核心包中的方法及耗时。
‘#cost>毫秒值’			  	  只打印耗时超过该毫秒值的调用。
–n 数值					   最多显示该数值条数的数据。
```

> 对该命令注意前面说的 [monitor/watch/trace相关](#monitor/watch/trace相关) 注意事项，监控完了需要使用命令结束。

运行此命令之后需要即时触发方法才会有响应的信息打印在控制台上，然后一层一层看子调用。

更多请看 [trace](https://arthas.aliyun.com/doc/trace.html) 命令。



### 重要：如何临时更改代码运行?

> 场景：我找到了问题所在，能否线上直接修改测试，而不需要在本地改了代码后，重新打包部署，然后重启观察效果?

```bash
# 先反编译出Class源码
jad --source-only com.example.demo.arthas.user.UserController > /tmp/UserController.Java  

# 然后使用外部工具编辑内容
mc /tmp/UserController.Java -d /tmp  # 再编译成Class

# 最后，重新载入定义的类，就可以实时验证你的猜测了
redefine /tmp/com/example/demo/arthas/user/UserController.Class
```

如上，是直接更改线上代码的方式，但是一般好像是编译不成功的，所以，最好是本地ide编译成 Class文件后，再上传替换为好！

总之，已经完全不用重启和发布了！这个功能真的很方便，比起重启带来的代价，真的是不可比的，比如，重启时可能导致负载重分配，选主等等问题，就不是你能控制的了



### 我如何测试某个方法的性能问题?

> 场景：我想看下某个方法的性能

```bash
monitor -c 5 demo.MathGame primeFactors
```





### 如何定位偏底层的性能问题？

> 场景：通过前面的排查之后，发现是偏底层的API耗时长，我怎么确定是底层什么原因导致的？如：for循环中向ArrayList添加数据。
>
> 可以采用trace+watch，但稍微麻烦，下面说明 profiler 命令生成性能监控火焰图来直观的查看。

```bash
# 步骤一
profiler start							开始监控方法执行性能。运行此命令之后需要即时触发方法才会进行监控

				注意：该命令不可在Windows中执行。

# 步骤二
profiler stop --format html				以HTML的方式生成火焰图
```

> 火焰图中一般找绿色部分Java中栈顶上比较平（比较宽）的部分，很可能就是性能的瓶颈。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240113173648691-627056610.png" alt="image-20240113173653011" />



更多请看这里：[profiler](https://arthas.aliyun.com/doc/profiler.html)



### 更多

请参考: [官方Issue墙](https://github.com/alibaba/arthas/issues?q=label%3Auser-case)



## Arthas源码

首先我们先放出一张整体宏观的模块调用图：

![trace](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107160932807-302497775.jpg)

源码理解可以这篇文章：https://yq.aliyun.com/articles/704228





## 参考资料

- https://www.cnblogs.com/muxuanchan/p/10097639.html
- https://www.cnblogs.com/yougewe/p/10770690.html
- https://help.aliyun.com/document_detail/112975.html





# [#](#Java动态调试技术原理) Java动态调试技术原理

> 本文转载自 美团技术团队胡健的[Java 动态调试技术原理及实践](https://tech.meituan.com/2019/11/07/java-dynamic-debugging-technology.html), 通过学习Java agent方式进行动态调试了解目前很多大厂开源的一些基于此的调试工具





##  简介

断点调试是我们最常使用的调试手段，它可以获取到方法执行过程中的变量信息，并可以观察到方法的执行路径但断点调试会在断点位置停顿，使得整个应用停止响应在线上停顿应用是致命的，动态调试技术给了我们创造新的调试模式的想象空间。本文将研究Java语言中的动态调试技术，首先概括Java动态调试所涉及的技术基础，接着介绍我们在Java动态调试领域的思考及实践，通过结合实际业务场景，设计并实现了一种具备动态性的断点调试工具Java-debug-tool，显著提高了故障排查效率

JVMTI (JVM Tool Interface)是Java虚拟机对外提供的Native编程接口，通过JVMTI，外部进程可以获取到运行时JVM的诸多信息，比如线程、GC等。Agent是一个运行在目标JVM的特定程序，它的职责是负责从目标JVM中获取数据，然后将数据传递给外部进程。加载Agent的时机可以是目标JVM启动之时，也可以是在目标JVM运行时进行加载，而在目标JVM运行时进行Agent加载具备动态性，对于时机未知的Debug场景来说非常实用。下面将详细分析Java Agent技术的实现细节



## Agent的实现模式

JVMTI是一套Native接口，在Java SE 5之前，要实现一个Agent只能通过编写Native代码来实现。从Java SE 5开始，可以使用Java的Instrumentation接口(Java.lang.instrument)来编写Agent。无论是通过Native的方式还是通过Java Instrumentation接口的方式来编写Agent，它们的工作都是借助JVMTI来进行完成，下面介绍通过Java Instrumentation接口编写Agent的方法



### 通过Java Instrumentation API

- 【[可选]：在maven中可以添加maven-assembly-plugin插件，此插件可以打包出java agent的jar包。】

```xml
<build>
	<plugins>
    	<plugin>
        	<groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plgin</artifactId>
            <configuration>
            	<!-- 将所有依赖都打入一个jar包中 -->
                <descriptorRefs>
                	<descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
                <!-- 指定java agent相关配置文件 -->
                <archive>
                	<manifestFile>src/main/resources/MANIFEST.MF</manifestFile>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
```

- 实现Agent启动方法：新建类，根据需要使用如下API即可，如AgentMain类

Java Agent支持目标JVM启动时加载（即静态加载模式），也支持在目标JVM运行时加载（即动态加载模式），这两种不同的加载模式会使用不同的入口函数，如果需要在目标JVM启动的同时加载Agent，那么可以选择实现下面的方法：

```java
// 静态加载模式
[1] public static void premain(String agentArgs, Instrumentation inst);
[2] public static void premain(String agentArgs);
```

JVM将首先寻找[1]，如果没有发现[1]，再寻找[2]。

如果希望在目标JVM运行时加载Agent，则需要实现下面的方法：

```java
// 动态加载模式
[1] public static void agentmain(String agentArgs, Instrumentation inst);
[2] public static void agentmain(String agentArgs);
```

这两组方法的第一个参数AgentArgs是随同 “-Javaagent”一起传入的程序参数，如果这个字符串代表了多个参数，就需要自己解析这些参数，inst是Instrumentation类型的对象，是JVM自动传入的，我们可以拿这个参数进行类增强等操作

- 指定Main-Class

Agent需要打包成一个jar包，在resources中新建MANIFEST.MF文件，此文件主要用于描述java agent的配置属性，比如使用哪一个类的agentmain方法。

> 注意：使用时去掉下面的注释

```ini
Mainfest-Version: 1.0
# 静态加载模式：指定 public static void premain() 方法所在的类路径
Premain-Class: xxxx.xxxx.xxxx.xxx.xxx.PreAgentMain
# 动态加载模式：指定 public static void agentmain(）方法所在的类路径
Agent-Class: xxxx.xxxx.xxxx.xxx.xxx.AgentMain
# 在Java Agent中能否重新定义一个类
Can-Redefine-Classes: true
# 能否将老的类转换成新的类
Can-Retransform-Classes: true
# 能否在Java Agent中生成native本地方法，有需要可以使用C/C++创建native方法
Can-Set-Native-Method-Prefix: true
# 注意：这里最好空一行，不然可能会报错

```

- 挂载到目标JVM：可以选择新建类，如AttachMain类

将编写的Agent打成jar包后，就可以挂载到目标JVM上去了如果选择在目标JVM启动时加载Agent，则可以使用 “-Javaagent:[=]“，具体的使用方法可以使用“Java -Help”来查看。如果想要在运行时挂载Agent到目标JVM，就需要做一些额外的开发了

com.sun.tools.attach.VirtualMachine 这个类代表一个JVM抽象，可以通过这个类找到目标JVM，并且将Agent挂载到目标JVM上。下面是使用com.sun.tools.attach.VirtualMachine进行动态挂载Agent的一般实现：

```java
    private void attachAgentToTargetJVM() throws Exception {
        List<VirtualMachineDescriptor> virtualMachineDescriptors = VirtualMachine.list();
        VirtualMachineDescriptor targetVM = null;
        for (VirtualMachineDescriptor descriptor : virtualMachineDescriptors) {
            // 通过指定的进程ID找到目标JVM
            if (descriptor.id().equals(configure.getPid())) {
                targetVM = descriptor;
                break;
            }
        }
        if (targetVM == null) {
            throw new IllegalArgumentException("could not find the target JVM by process id:" + configure.getPid());
        }
        VirtualMachine virtualMachine = null;
        try {
            // 通过Attach挂载到目标JVM上	最核心的就是这两步
            virtualMachine = VirtualMachine.attach(targetVM);
            virtualMachine.loadAgent("{agent}", "{params}");
        } catch (Exception e) {
            if (virtualMachine != null) {
                // 将Agent从目标JVM卸载
                virtualMachine.detach();
            }
        }
    }
```

首先通过指定的进程ID找到目标JVM，然后通过Attach挂载到目标JVM上，执行加载Agent操作。VirtualMachine的Attach方法就是用来将Agent挂载到目标JVM上去的，而Detach则是将Agent从目标JVM卸载。关于Agent是如何挂载到目标JVM上的具体技术细节，将在下文中进行分析



## 启动时加载Agent

### 参数解析

创建JVM时，JVM会进行参数解析，即解析那些用来配置JVM启动的参数，比如堆大小、GC等；本文主要关注解析的参数为-agentlib、 -agentpath、 -Javaagent，这几个参数用来指定Agent，JVM会根据这几个参数加载Agent。下面来分析一下JVM是如何解析这几个参数的

```java
  // -agentlib and -agentpath
  if (match_option(option, "-agentlib:", &tail) ||
          (is_absolute_path = match_option(option, "-agentpath:", &tail))) {
      if(tail != NULL) {
        const char* pos = strchr(tail, '=');
        size_t len = (pos == NULL) ? strlen(tail) : pos - tail;
        char* name = strncpy(NEW_C_HEAP_ARRAY(char, len + 1, mtArguments), tail, len);
        name[len] = '\0';
        char *options = NULL;
        if(pos != NULL) {
          options = os::strdup_check_oom(pos + 1, mtArguments);
        }
#if !INCLUDE_JVMTI
        if (valid_jdwp_agent(name, is_absolute_path)) {
          jio_fprintf(defaultStream::error_stream(),
            "Debugging agents are not supported in this VM\n");
          return JNI_ERR;
        }
#endif // !INCLUDE_JVMTI
        add_init_agent(name, options, is_absolute_path);
      }
    // -Javaagent
    } else if (match_option(option, "-Javaagent:", &tail)) {
#if !INCLUDE_JVMTI
      jio_fprintf(defaultStream::error_stream(),
        "Instrumentation agents are not supported in this VM\n");
      return JNI_ERR;
#else
      if (tail != NULL) {
        size_t length = strlen(tail) + 1;
        char *options = NEW_C_HEAP_ARRAY(char, length, mtArguments);
        jio_snprintf(options, length, "%s", tail);
        add_init_agent("instrument", options, false);
        // Java agents need module Java.instrument
        if (!create_numbered_property("JDK.module.addmods", "Java.instrument", addmods_count++)) {
          return JNI_ENOMEM;
        }
      }
#endif // !INCLUDE_JVMTI
    }
```

上面的代码片段截取自hotspot/src/share/vm/runtime/arguments.cpp中的 Arguments::parse_each_vm_init_arg(const JavaVMInitArgs* args, bool* patch_mod_Javabase, Flag::Flags origin) 函数，该函数用来解析一个具体的JVM参数。这段代码的主要功能是解析出需要加载的Agent路径，然后调用add_init_agent函数进行解析结果的存储。下面先看一下add_init_agent函数的具体实现：

```java
  // -agentlib and -agentpath arguments
  static AgentLibraryList _agentList;
  static void add_init_agent(const char* name, char* options, bool absolute_path)
    { _agentList.add(new AgentLibrary(name, options, absolute_path, NULL)); }
```

AgentLibraryList是一个简单的链表结构，add_init_agent函数将解析好的、需要加载的Agent添加到这个链表中，等待后续的处理

这里需要注意，解析-Javaagent参数有一些特别之处，这个参数用来指定一个我们通过Java Instrumentation API来编写的Agent，Java Instrumentation API底层依赖的是JVMTI，对-JavaAgent的处理也说明了这一点，在调用add_init_agent函数时第一个参数是“instrument”，关于加载Agent这个问题在下一小节进行展开。到此，我们知道在启动JVM时指定的Agent已经被JVM解析完存放在了一个链表结构中。下面来分析一下JVM是如何加载这些Agent的



### 执行加载操作

在创建JVM进程的函数中，解析完JVM参数之后，下面的这段代码和加载Agent相关：

```java
  // Launch -agentlib/-agentpath and converted -Xrun agents
  if (Arguments::init_agents_at_startup()) {
    create_vm_init_agents();
  }
  static bool init_agents_at_startup() {
    return !_agentList.is_empty(); 
  }
```

当JVM判断出上一小节中解析出来的Agent不为空的时候，就要去调用函数create_vm_init_agents来加载Agent，下面来分析一下create_vm_init_agents函数是如何加载Agent的

```java
void Threads::create_vm_init_agents() {
  AgentLibrary* agent;
  for (agent = Arguments::agents(); agent != NULL; agent = agent->next()) {
    OnLoadEntry_t  on_load_entry = lookup_agent_on_load(agent);
    if (on_load_entry != NULL) {
      // Invoke the Agent_OnLoad function
      jint err = (*on_load_entry)(&main_vm, agent->options(), NULL);
    }
  }
}
```

create_vm_init_agents这个函数通过遍历Agent链表来逐个加载Agent。通过这段代码可以看出，首先通过lookup_agent_on_load来加载Agent并且找到Agent_OnLoad函数，这个函数是Agent的入口函数。如果没找到这个函数，则认为是加载了一个不合法的Agent，则什么也不做，否则调用这个函数，这样Agent的代码就开始执行起来了。对于使用Java Instrumentation API来编写Agent的方式来说，在解析阶段观察到在add_init_agent函数里面传递进去的是一个叫做”instrument”的字符串，其实这是一个动态链接库。在Linux里面，这个库叫做libinstrument.so，在BSD系统中叫做libinstrument.dylib，该动态链接库在{Java_HOME}/jre/lib/目录下



### instrument动态链接库

libinstrument用来支持使用Java Instrumentation API来编写Agent，在libinstrument中有一个非常重要的类称为：JPLISAgent(Java Programming Language Instrumentation Services Agent)，它的作用是初始化所有通过Java Instrumentation API编写的Agent，并且也承担着通过JVMTI实现Java Instrumentation中暴露API的责任

我们已经知道，在JVM启动的时候，JVM会通过-Javaagent参数加载Agent。最开始加载的是libinstrument动态链接库，然后在动态链接库里面找到JVMTI的入口方法：Agent_OnLoad。下面就来分析一下在libinstrument动态链接库中，Agent_OnLoad函数是怎么实现的

```java
JNIEXPORT jint JNICALL
DEF_Agent_OnLoad(JavaVM *vm, char *tail, void * reserved) {
    initerror = createNewJPLISAgent(vm, &agent);
    if ( initerror == JPLIS_INIT_ERROR_NONE ) {
        if (parseArgumentTail(tail, &jarfile, &options) != 0) {
            fprintf(stderr, "-Javaagent: memory allocation failure.\n");
            return JNI_ERR;
        }
        attributes = readAttributes(jarfile);
        premainClass = getAttribute(attributes, "Premain-Class");
        /* Save the jarfile name */
        agent->mJarfile = jarfile;
        /*
         * Convert JAR attributes into agent capabilities
         */
        convertCapabilityAttributes(attributes, agent);
        /*
         * Track (record) the agent Class name and options data
         */
        initerror = recordCommandLineData(agent, premainClass, options);
    }
    return result;
}
```

上述代码片段是经过精简的libinstrument中Agent_OnLoad实现，大概的流程就是：先创建一个JPLISAgent，然后将ManiFest中设定的一些参数解析出来， 比如(Premain-Class)等。创建了JPLISAgent之后，调用initializeJPLISAgent对这个Agent进行初始化操作。跟进initializeJPLISAgent看一下是如何初始化的：

```java
JPLISInitializationError initializeJPLISAgent(JPLISAgent *agent, JavaVM *vm, JVMtiEnv *JVMtienv) {
    /* check what capabilities are available */
    checkCapabilities(agent);
    /* check phase - if live phase then we don't need the VMInit event */
    JVMtierror = (*JVMtienv)->GetPhase(JVMtienv, &phase);
    /* now turn on the VMInit event */
    if ( JVMtierror == JVMTI_ERROR_NONE ) {
        JVMtiEventCallbacks callbacks;
        memset(&callbacks, 0, sizeof(callbacks));
        callbacks.VMInit = &eventHandlerVMInit;
        JVMtierror = (*JVMtienv)->SetEventCallbacks(JVMtienv,&callbacks,sizeof(callbacks));
    }
    if ( JVMtierror == JVMTI_ERROR_NONE ) {
        JVMtierror = (*JVMtienv)->SetEventNotificationMode(JVMtienv,JVMTI_ENABLE,JVMTI_EVENT_VM_INIT,NULL);
    }
    return (JVMtierror == JVMTI_ERROR_NONE)? JPLIS_INIT_ERROR_NONE : JPLIS_INIT_ERROR_FAILURE;
}
```

这里，我们关注callbacks.VMInit = &eventHandlerVMInit;这行代码，这里设置了一个VMInit事件的回调函数，表示在JVM初始化的时候会回调eventHandlerVMInit函数。下面来看一下这个函数的实现细节，猜测就是在这里调用了Premain方法：

```java
void JNICALL  eventHandlerVMInit( JVMtiEnv *JVMtienv,JNIEnv *jnienv,jthread thread) {
   // ...
   success = processJavaStart( environment->mAgent, jnienv);
  // ...
}
jboolean  processJavaStart(JPLISAgent *agent,JNIEnv *jnienv) {
    result = createInstrumentationImpl(jnienv, agent);
    /*
     *  Load the Java agent, and call the premain.
     */
    if ( result ) {
        result = startJavaAgent(agent, jnienv, agent->mAgentClassName, agent->mOptionsString, agent->mPremainCaller);
    }
    return result;
}
jboolean startJavaAgent( JPLISAgent *agent,JNIEnv *jnienv,const char *Classname,const char *optionsString,jmethodID agentMainMethod) {
  // ...  
  invokeJavaAgentMainMethod(jnienv,agent->mInstrumentationImpl,agentMainMethod, ClassNameObject,optionsStringObject);
  // ...
}
```

看到这里，Instrument已经实例化，invokeJavaAgentMainMethod这个方法将我们的premain方法执行起来了。接着，我们就可以根据Instrument实例来做我们想要做的事情了





## 运行时加载Agent

比起JVM启动时加载Agent，运行时加载Agent就比较有诱惑力了，因为运行时加载Agent的能力给我们提供了很强的动态性，我们可以在需要的时候加载Agent来进行一些工作。因为是动态的，我们可以按照需求来加载所需要的Agent，下面来分析一下动态加载Agent的相关技术细节



### AttachListener

Attach机制通过Attach Listener线程来进行相关事务的处理，下面来看一下Attach Listener线程是如何初始化的

```java
// Starts the Attach Listener thread
void AttachListener::init() {
  // 创建线程相关部分代码被去掉了
  const char thread_name[] = "Attach Listener";
  Handle string = Java_lang_String::create_from_str(thread_name, THREAD);
  { MutexLocker mu(Threads_lock);
    JavaThread* listener_thread = new JavaThread(&attach_listener_thread_entry);
    // ...
  }
}
```

我们知道，一个线程启动之后都需要指定一个入口来执行代码，Attach Listener线程的入口是attach_listener_thread_entry，下面看一下这个函数的具体实现：

```java
static void attach_listener_thread_entry(JavaThread* thread, TRAPS) {
  AttachListener::set_initialized();
  for (;;) {
      AttachOperation* op = AttachListener::dequeue();
      // find the function to dispatch too
      AttachOperationFunctionInfo* info = NULL;
      for (int i=0; funcs[i].name != NULL; i++) {
        const char* name = funcs[i].name;
        if (strcmp(op->name(), name) == 0) {
          info = &(funcs[i]); break;
        }}
       // dispatch to the function that implements this operation
        res = (info->func)(op, &st);
      //...
    }
}
```

整个函数执行逻辑，大概是这样的：

- 拉取一个需要执行的任务：AttachListener::dequeue
- 查询匹配的命令处理函数
- 执行匹配到的命令执行函数

其中第二步里面存在一个命令函数表，整个表如下：

```java
static AttachOperationFunctionInfo funcs[] = {
  { "agentProperties",  get_agent_properties },
  { "datadump",         data_dump },
  { "dumpheap",         dump_heap },
  { "load",             load_agent },
  { "properties",       get_system_properties },
  { "threaddump",       thread_dump },
  { "inspectheap",      heap_inspection },
  { "setflag",          set_flag },
  { "printflag",        print_flag },
  { "jcmd",             jcmd },
  { NULL,               NULL }
};
```

对于加载Agent来说，命令就是“load”现在，我们知道了Attach Listener大概的工作模式，但是还是不太清楚任务从哪来，这个秘密就藏在AttachListener::dequeue这行代码里面，接下来我们来分析一下dequeue这个函数：

```java
LinuxAttachOperation* LinuxAttachListener::dequeue() {
  for (;;) {
    // wait for client to connect
    struct sockaddr addr;
    socklen_t len = sizeof(addr);
    RESTARTABLE(::accept(listener(), &addr, &len), s);
    // get the credentials of the peer and check the effective uid/guid
    // - check with jeff on this.
    struct ucred cred_info;
    socklen_t optlen = sizeof(cred_info);
    if (::getsockopt(s, SOL_SOCKET, SO_PEERCRED, (void*)&cred_info, &optlen) == -1) {
      ::close(s);
      continue;
    }
    // peer credential look okay so we read the request
    LinuxAttachOperation* op = read_request(s);
    return op;
  }
}
```

这是Linux上的实现，不同的操作系统实现方式不太一样。上面的代码表面，Attach Listener在某个端口监听着，通过accept来接收一个连接，然后从这个连接里面将请求读取出来，然后将请求包装成一个AttachOperation类型的对象，之后就会从表里查询对应的处理函数，然后进行处理

Attach Listener使用一种被称为“懒加载”的策略进行初始化，也就是说，JVM启动的时候Attach Listener并不一定会启动起来。下面我们来分析一下这种“懒加载”策略的具体实现方案

```java
  // Start Attach Listener if +StartAttachListener or it can't be started lazily
  if (!DisableAttachMechanism) {
    AttachListener::vm_start();
    if (StartAttachListener || AttachListener::init_at_startup()) {
      AttachListener::init();
    }
  }
// Attach Listener is started lazily except in the case when
// +ReduseSignalUsage is used
bool AttachListener::init_at_startup() {
  if (ReduceSignalUsage) {
    return true;
  } else {
    return false;
  }
}
```

上面的代码截取自create_vm函数，DisableAttachMechanism、StartAttachListener和ReduceSignalUsage这三个变量默认都是false，所以AttachListener::init();这行代码不会在create_vm的时候执行，而vm_start会执行下面来看一下这个函数的实现细节：

```java
void AttachListener::vm_start() {
  char fn[UNIX_PATH_MAX];
  struct stat64 st;
  int ret;
  int n = snprintf(fn, UNIX_PATH_MAX, "%s/.Java_pid%d",
           os::get_temp_directory(), os::current_process_id());
  assert(n < (int)UNIX_PATH_MAX, "Java_pid file name buffer overflow");
  RESTARTABLE(::stat64(fn, &st), ret);
  if (ret == 0) {
    ret = ::unlink(fn);
    if (ret == -1) {
      log_debug(attach)("Failed to remove stale attach pid file at %s", fn);
    }
  }
}
```

这是在Linux上的实现，是将/tmp/目录下的.Java_pid{pid}文件删除，后面在创建Attach Listener线程的时候会创建出来这个文件。上面说到，AttachListener::init()这行代码不会在create_vm的时候执行，这行代码的实现已经在上文中分析了，就是创建Attach Listener线程，并监听其他JVM的命令请求。现在来分析一下这行代码是什么时候被调用的，也就是“懒加载”到底是怎么加载起来的

```java
  // Signal Dispatcher needs to be started before VMInit event is posted
  os::signal_init();
```

这是create_vm中的一段代码，看起来跟信号相关，其实Attach机制就是使用信号来实现“懒加载“的。下面我们来仔细地分析一下这个过程

```java
void os::signal_init() {
  if (!ReduceSignalUsage) {
    // Setup JavaThread for processing signals
    EXCEPTION_MARK;
    Klass* k = SystemDictionary::resolve_or_fail(vmSymbols::Java_lang_Thread(), true, CHECK);
    instanceKlassHandle klass (THREAD, k);
    instanceHandle thread_oop = klass->allocate_instance_handle(CHECK);
    const char thread_name[] = "Signal Dispatcher";
    Handle string = Java_lang_String::create_from_str(thread_name, CHECK);
    // Initialize thread_oop to put it into the system threadGroup
    Handle thread_group (THREAD, Universe::system_thread_group());
    JavaValue result(T_VOID);
    JavaCalls::call_special(&result, thread_oop,klass,vmSymbols::object_initializer_name(),vmSymbols::threadgroup_string_void_signature(),
                           thread_group,string,CHECK);
    KlassHandle group(THREAD, SystemDictionary::ThreadGroup_klass());
    JavaCalls::call_special(&result,thread_group,group,vmSymbols::add_method_name(),vmSymbols::thread_void_signature(),thread_oop,CHECK);
    os::signal_init_pd();
    { MutexLocker mu(Threads_lock);
      JavaThread* signal_thread = new JavaThread(&signal_thread_entry);
     // ...
    }
    // Handle ^BREAK
    os::signal(SIGBREAK, os::user_handler());
  }
}
```

JVM创建了一个新的进程来实现信号处理，这个线程叫“Signal Dispatcher”，一个线程创建之后需要有一个入口，“Signal Dispatcher”的入口是signal_thread_entry：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155511352-1064313242.png)

这段代码截取自signal_thread_entry函数，截取中的内容是和Attach机制信号处理相关的代码。这段代码的意思是，当接收到“SIGBREAK”信号，就执行接下来的代码，这个信号是需要Attach到JVM上的信号发出来，这个后面会再分析。我们先来看一句关键的代码：AttachListener::is_init_trigger()：

```java
bool AttachListener::is_init_trigger() {
  if (init_at_startup() || is_initialized()) {
    return false;               // initialized at startup or already initialized
  }
  char fn[PATH_MAX+1];
  sprintf(fn, ".attach_pid%d", os::current_process_id());
  int ret;
  struct stat64 st;
  RESTARTABLE(::stat64(fn, &st), ret);
  if (ret == -1) {
    log_trace(attach)("Failed to find attach file: %s, trying alternate", fn);
    snprintf(fn, sizeof(fn), "%s/.attach_pid%d", os::get_temp_directory(), os::current_process_id());
    RESTARTABLE(::stat64(fn, &st), ret);
  }
  if (ret == 0) {
    // simple check to avoid starting the attach mechanism when
    // a bogus user creates the file
    if (st.st_uid == geteuid()) {
      init();
      return true;
    }
  }
  return false;
}
```

首先检查了一下是否在JVM启动时启动了Attach Listener，或者是否已经启动过。如果没有，才继续执行，在/tmp目录下创建一个叫做.attach_pid%d的文件，然后执行AttachListener的init函数，这个函数就是用来创建Attach Listener线程的函数，上面已经提到多次并进行了分析。到此，我们知道Attach机制的奥秘所在，也就是Attach Listener线程的创建依靠Signal Dispatcher线程，Signal Dispatcher是用来处理信号的线程，当Signal Dispatcher线程接收到“SIGBREAK”信号之后，就会执行初始化Attach Listener的工作



### 运行时加载Agent的实现

我们继续分析，到底是如何将一个Agent挂载到运行着的目标JVM上，在上文中提到了一段代码，用来进行运行时挂载Agent，可以参考上文中展示的关于“attachAgentToTargetJVM”方法的代码。这个方法里面的关键是调用VirtualMachine的attach方法进行Agent挂载的功能。下面我们就来分析一下VirtualMachine的attach方法具体是怎么实现的

```java
public static VirtualMachine attach(String var0) throws AttachNotSupportedException, IOException {
    if (var0 == null) {
        throw new NullPointerException("id cannot be null");
    } else {
        List var1 = AttachProvider.providers();
        if (var1.size() == 0) {
            throw new AttachNotSupportedException("no providers installed");
        } else {
            AttachNotSupportedException var2 = null;
            Iterator var3 = var1.iterator();
            while(var3.hasNext()) {
                AttachProvider var4 = (AttachProvider)var3.next();
                try {
                    return var4.attachVirtualMachine(var0);
                } catch (AttachNotSupportedException var6) {
                    var2 = var6;
                }
            }
            throw var2;
        }
    }
}
```

这个方法通过attachVirtualMachine方法进行attach操作，在MacOS系统中，AttachProvider的实现类是BsdAttachProvider。我们来看一下BsdAttachProvider的attachVirtualMachine方法是如何实现的：

```java
public VirtualMachine attachVirtualMachine(String var1) throws AttachNotSupportedException, IOException {
    this.checkAttachPermission();
    this.testAttachable(var1);
    return new BsdVirtualMachine(this, var1);
}
BsdVirtualMachine(AttachProvider var1, String var2) throws AttachNotSupportedException, IOException {
    int var3 = Integer.parseInt(var2);
    this.path = this.findSocketFile(var3);
    if (this.path == null) {
        File var4 = new File(tmpdir, ".attach_pid" + var3);
        createAttachFile(var4.getPath());
        try {
            sendQuitTo(var3);
            int var5 = 0;
            long var6 = 200L;
            int var8 = (int)(this.attachTimeout() / var6);
            do {
                try {
                    Thread.sleep(var6);
                } catch (InterruptedException var21) {
                    ;
                }
                this.path = this.findSocketFile(var3);
                ++var5;
            } while(var5 <= var8 && this.path == null);
        } finally {
            var4.delete();
        }
    }
    int var24 = socket();
    connect(var24, this.path);
}
private String findSocketFile(int var1) {
    String var2 = ".Java_pid" + var1;
    File var3 = new File(tmpdir, var2);
    return var3.exists() ? var3.getPath() : null;
}
```

findSocketFile方法用来查询目标JVM上是否已经启动了Attach Listener，它通过检查”tmp/“目录下是否存在Java_pid{pid}来进行实现。如果已经存在了，则说明Attach机制已经准备就绪，可以接受客户端的命令了，这个时候客户端就可以通过connect连接到目标JVM进行命令的发送，比如可以发送“load”命令来加载Agent；如果Java_pid{pid}文件还不存在，则需要通过sendQuitTo方法向目标JVM发送一个“SIGBREAK”信号，让它初始化Attach Listener线程并准备接受客户端连接。可以看到，发送了信号之后客户端会循环等待Java_pid{pid}这个文件，之后再通过connect连接到目标JVM上



### load命令的实现

下面来分析一下，“load”命令在JVM层面的实现：

```java
static jint load_agent(AttachOperation* op, outputStream* out) {
  // get agent name and options
  const char* agent = op->arg(0);
  const char* absParam = op->arg(1);
  const char* options = op->arg(2);
  // If loading a Java agent then need to ensure that the Java.instrument module is loaded
  if (strcmp(agent, "instrument") == 0) {
    Thread* THREAD = Thread::current();
    ResourceMark rm(THREAD);
    HandleMark hm(THREAD);
    JavaValue result(T_OBJECT);
    Handle h_module_name = Java_lang_String::create_from_str("Java.instrument", THREAD);
    JavaCalls::call_static(&result,SystemDictionary::module_Modules_klass(),vmSymbols::loadModule_name(),
                           vmSymbols::loadModule_signature(),h_module_name,THREAD);
  }
  return JVMtiExport::load_agent_library(agent, absParam, options, out);
}
```

这个函数先确保加载了Java.instrument模块，之后真正执行Agent加载的函数是 load_agent_library ,这个函数的套路就是加载Agent动态链接库，如果是通过Java instrument API实现的Agent，则加载的是libinstrument动态链接库，然后通过libinstrument里面的代码实现运行agentmain方法的逻辑，这一部分内容和libinstrument实现premain方法运行的逻辑其实差不多，这里不再做分析至此，我们对Java Agent技术已经有了一个全面而细致的了解



## 动态字节码修改的限制

上文中已经详细分析了Agent技术的实现，我们使用Java Instrumentation API来完成动态类修改的功能，在Instrumentation接口中，通过addTransformer方法来增加一个类转换器，类转换器由类ClassFileTransformer接口实现ClassFileTransformer接口中唯一的方法transform用于实现类转换，当类被加载的时候，就会调用transform方法，进行类转换在运行时，我们可以通过Instrumentation的redefineClasses方法进行类重定义，在方法上有一段注释需要特别注意：

```java
     * The redefinition may change method bodies, the constant pool and attributes.
     * The redefinition must not add, remove or rename fields or methods, change the
     * signatures of methods, or change inheritance.  These restrictions maybe be
     * lifted in future versions.  The Class file bytes are not checked, verified and installed
     * until after the transformations have been applied, if the resultant bytes are in
     * error this method will throw an exception.
```

这里面提到，我们不可以增加、删除或者重命名字段和方法，改变方法的签名或者类的继承关系。认识到这一点很重要，当我们通过ASM获取到增强的字节码之后，如果增强后的字节码没有遵守这些规则，那么调用redefineClasses方法来进行类的重定义就会失败。那redefineClasses方法具体是怎么实现类的重定义的呢? 它对运行时的JVM会造成什么样的影响呢? 下面来分析redefineClasses的实现细节





## 重定义类字节码的实现细节

上文中我们提到，libinstrument动态链接库中，JPLISAgent不仅实现了Agent入口代码执行的路由，而且还是Java代码与JVMTI之间的一道桥梁。我们在Java代码中调用Java Instrumentation API的redefineClasses，其实会调用libinstrument中的相关代码，我们来分析一下这条路径

```java
public void redefineClasses(ClassDefinition... var1) throws ClassNotFoundException {
    if (!this.isRedefineClassesSupported()) {
        throw new UnsupportedOperationException("redefineClasses is not supported in this environment");
    } else if (var1 == null) {
        throw new NullPointerException("null passed as 'definitions' in redefineClasses");
    } else {
        for(int var2 = 0; var2 < var1.length; ++var2) {
            if (var1[var2] == null) {
                throw new NullPointerException("element of 'definitions' is null in redefineClasses");
            }
        }
        if (var1.length != 0) {
            this.redefineClasses0(this.mNativeAgent, var1);
        }
    }
}
private native void redefineClasses0(long var1, ClassDefinition[] var3) throws ClassNotFoundException;
```

这是InstrumentationImpl中的redefineClasses实现，该方法的具体实现依赖一个Native方法redefineClasses()，我们可以在libinstrument中找到这个Native方法的实现：

```java
JNIEXPORT void JNICALL Java_sun_instrument_InstrumentationImpl_redefineClasses0
  (JNIEnv * jnienv, jobject implThis, jlong agent, jobjectArray ClassDefinitions) {
    redefineClasses(jnienv, (JPLISAgent*)(intptr_t)agent, ClassDefinitions);
}
```

redefineClasses这个函数的实现比较复杂，代码很长下面是一段关键的代码片段：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155511391-960675052.png)

可以看到，其实是调用了JVMTI的RetransformClasses函数来完成类的重定义细节

```java
// Class_count - pre-checked to be greater than or equal to 0
// Class_definitions - pre-checked for NULL
JVMtiError JVMtiEnv::RedefineClasses(jint Class_count, const JVMtiClassDefinition* Class_definitions) {
//TODO: add locking
  VM_RedefineClasses op(Class_count, Class_definitions, JVMti_Class_load_kind_redefine);
  VMThread::execute(&op);
  return (op.check_error());
} /* end RedefineClasses */
```

重定义类的请求会被JVM包装成一个VM_RedefineClasses类型的VM_Operation，VM_Operation是JVM内部的一些操作的基类，包括GC操作等。VM_Operation由VMThread来执行，新的VM_Operation操作会被添加到VMThread的运行队列中去，VMThread会不断从队列里面拉取VM_Operation并调用其doit等函数执行具体的操作。VM_RedefineClasses函数的流程较为复杂，下面是VM_RedefineClasses的大致流程：

- 加载新的字节码，合并常量池，并且对新的字节码进行校验工作

```java
  // Load the caller's new Class definition(s) into _scratch_Classes.
  // Constant pool merging work is done here as needed. Also calls
  // compare_and_normalize_Class_versions() to verify the Class
  // definition(s).
  JVMtiError load_new_Class_versions(TRAPS);
```

- 清除方法上的断点

```java
  // Remove all breakpoints in methods of this Class
  JVMtiBreakpoints& JVMti_breakpoints = JVMtiCurrentBreakpoints::get_JVMti_breakpoints();
  JVMti_breakpoints.clearall_in_Class_at_safepoint(the_Class());
```

- JIT逆优化

```java
  // Deoptimize all compiled code that depends on this Class
  flush_dependent_code(the_Class, THREAD);
```

- 进行字节码替换工作，需要进行更新类itable/vtable等操作
- 进行类重定义通知

```java
  SystemDictionary::notice_modification();
```

VM_RedefineClasses实现比较复杂的，详细实现可以参考 [RedefineClasses](https://github.com/pandening/openJDK/blob/0301fc792ffd3c7b506ef78887af250e0e3ae09e/src/hotspot/share/prims/JVMtiEnv.cpp#L456)的实现





## Java-debug-tool

Java-debug-tool是一个使用Java Instrument API来实现的动态调试工具，它通过在目标JVM上启动一个TcpServer来和调试客户端通信。调试客户端通过命令行来发送调试命令给TcpServer，TcpServer中有专门用来处理命令的handler，handler处理完命令之后会将结果发送回客户端，客户端通过处理将调试结果展示出来。下面将详细介绍Java-debug-tool的整体设计和实现



### Java-debug-tool整体架构

Java-debug-tool包括一个Java Agent和一个用于处理调试命令的核心API，核心API通过一个自定义的类加载器加载进来，以保证目标JVM的类不会被污染。整体上Java-debug-tool的设计是一个Client-Server的架构，命令客户端需要完整的完成一个命令之后才能继续执行下一个调试命令。Java-debug-tool支持多人同时进行调试，下面是整体架构图：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155511371-473664371.png)

下面对每一层做简单介绍：

- 交互层：负责将程序员的输入转换成调试交互协议，并且将调试信息呈现出来
- 连接管理层：负责管理客户端连接，从连接中读调试协议数据并解码，对调试结果编码并将其写到连接中去；同时将那些超时未活动的连接关闭
- 业务逻辑层：实现调试命令处理，包括命令分发、数据收集、数据处理等过程
- 基础实现层：Java-debug-tool实现的底层依赖，通过Java Instrumentation提供的API进行类查找、类重定义等能力，Java Instrumentation底层依赖JVMTI来完成具体的功能

在Agent被挂载到目标JVM上之后，Java-debug-tool会安排一个Spy在目标JVM内活动，这个Spy负责将目标JVM内部的相关调试数据转移到命令处理模块，命令处理模块会处理这些数据，然后给客户端返回调试结果。命令处理模块会增强目标类的字节码来达到数据获取的目的，多个客户端可以共享一份增强过的字节码，无需重复增强下面从Java-debug-tool的字节码增强方案、命令设计与实现等角度详细说明





### Java-debug-tool的字节码增强方案

Java-debug-tool使用字节码增强来获取到方法运行时的信息，比如方法入参、出参等，可以在不同的字节码位置进行增强，这种行为可以称为“插桩”，每个“桩”用于获取数据并将他转储出去。Java-debug-tool具备强大的插桩能力，不同的桩负责获取不同类别的数据，下面是Java-debug-tool目前所支持的“桩”：

- 方法进入点：用于获取方法入参信息
- Fields获取点1：在方法执行前获取到对象的字段信息
- 变量存储点：获取局部变量信息
- Fields获取点2：在方法退出前获取到对象的字段信息
- 方法退出点：用于获取方法返回值
- 抛出异常点：用于获取方法抛出的异常信息
- 通过上面这些代码桩，Java-debug-tool可以收集到丰富的方法执行信息，经过处理可以返回更加可视化的调试结果



#### 字节码增强

Java-debug-tool在实现上使用了ASM工具来进行字节码增强，并且每个插桩点都可以进行配置，如果不想要什么信息，则没必要进行对应的插桩操作这种可配置的设计是非常有必要的，因为有时候我们仅仅是想要知道方法的入参和出参，但Java-debug-tool却给我们返回了所有的调试信息，这样我们就得在众多的输出中找到我们所关注的内容。如果可以进行配置，则除了入参点和出参点外其他的桩都不插，那么就可以快速看到我们想要的调试数据，这种设计的本质是为了让调试者更加专注。下面是Java-debug-tool的字节码增强工作方式：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155511352-304047437.png)

如图所示，当调试者发出调试命令之后，Java-debug-tool会识别命令并判断是否需要进行字节码增强，如果命令需要增强字节码，则判断当前类+当前方法是否已经被增强过。上文已经提到，字节码替换是有一定损耗的，这种具有损耗的操作发生的次数越少越好，所以字节码替换操作会被记录起来，后续命令直接使用即可，不需要重复进行字节码增强，字节码增强还涉及多个调试客户端的协同工作问题，当一个客户端增强了一个类的字节码之后，这个客户端就锁定了该字节码，其他客户端变成只读，无法对该类进行字节码增强，只有当持有锁的客户端主动释放锁或者断开连接之后，其他客户端才能继续增强该类的字节码

字节码增强模块收到字节码增强请求之后，会判断每个增强点是否需要插桩，这个判断的根据就是上文提到的插桩配置，之后字节码增强模块会生成新的字节码，Java-debug-tool将执行字节码替换操作，之后就可以进行调试数据收集了

经过字节码增强之后，原来的方法中会插入收集运行时数据的代码，这些代码在方法被调用的时候执行，获取到诸如方法入参、局部变量等信息，这些信息将传递给数据收集装置进行处理。数据收集的工作通过Advice完成，每个客户端同一时间只能注册一个Advice到Java-debug-tool调试模块上，多个客户端可以同时注册自己的Advice到调试模块上。Advice负责收集数据并进行判断，如果当前数据符合调试命令的要求，Java-debug-tool就会卸载这个Advice，Advice的数据就会被转移到Java-debug-tool的命令结果处理模块进行处理，并将结果发送到客户端





#### Advice的工作方式

Advice是调试数据收集器，不同的调试策略会对应不同的Advice。Advice是工作在目标JVM的线程内部的，它需要轻量级和高效，意味着Advice不能做太过于复杂的事情，它的核心接口“match”用来判断本次收集到的调试数据是否满足调试需求。如果满足，那么Java-debug-tool就会将其卸载，否则会继续让他收集调试数据，这种“加载Advice” -> “卸载Advice”的工作模式具备很好的灵活性

关于Advice，需要说明的另外一点就是线程安全，因为它加载之后会运行在目标JVM的线程中，目标JVM的方法极有可能是多线程访问的，这也就是说，Advice需要有能力处理多个线程同时访问方法的能力，如果Advice处理不当，则可能会收集到杂乱无章的调试数据。下面的图片展示了Advice和Java-debug-tool调试分析模块、目标方法执行以及调试客户端等模块的关系

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155511371-1082875809.png)

Advice的首次挂载由Java-debug-tool的命令处理器完成，当一次调试数据收集完成之后，调试数据处理模块会自动卸载Advice，然后进行判断，如果调试数据符合Advice的策略，则直接将数据交由数据处理模块进行处理，否则会清空调试数据，并再次将Advice挂载到目标方法上去，等待下一次调试数据。非首次挂载由调试数据处理模块进行，它借助Advice按需取数据，如果不符合需求，则继续挂载Advice来获取数据，否则对调试数据进行处理并返回给客户端





### Java-debug-tool的命令设计与实现

#### 命令执行

上文已经完整的描述了Java-debug-tool的设计以及核心技术方案，本小节将详细介绍Java-debug-tool的命令设计与实现。首先需要将一个调试命令的执行流程描述清楚，下面是一张用来表示命令请求处理流程的图片：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155511335-1530405067.png)

上图简单的描述了Java-debug-tool的命令处理方式，客户端连接到服务端之后，会进行一些协议解析、协议认证、协议填充等工作，之后将进行命令分发。服务端如果发现客户端的命令不合法，则会立即返回错误信息，否则再进行命令处理命令。处理属于典型的三段式处理，前置命令处理、命令处理以及后置命令处理，同时会对命令处理过程中的异常信息进行捕获处理，三段式处理的好处是命令处理被拆成了多个阶段，多个阶段负责不同的职责。前置命令处理用来做一些命令权限控制的工作，并填充一些类似命令处理开始时间戳等信息，命令处理就是通过字节码增强，挂载Advice进行数据收集，再经过数据处理来产生命令结果的过程，后置处理则用来处理一些连接关闭、字节码解锁等事项

Java-debug-tool允许客户端设置一个命令执行超时时间，超过这个时间则认为命令没有结果，如果客户端没有设置自己的超时时间，就使用默认的超时时间进行超时控制。Java-debug-tool通过设计了两阶段的超时检测机制来实现命令执行超时功能：首先，第一阶段超时触发，则Java-debug-tool会友好的警告命令处理模块处理时间已经超时，需要立即停止命令执行，这允许命令自己做一些现场清理工作，当然需要命令执行线程自己感知到这种超时警告；当第二阶段超时触发，则Java-debug-tool认为命令必须结束执行，会强行打断命令执行线程。超时机制的目的是为了不让命令执行太长时间，命令如果长时间没有收集到调试数据，则应该停止执行，并思考是否调试了一个错误的方法当然，超时机制还可以定期清理那些因为未知原因断开连接的客户端持有的调试资源，比如字节码锁





#### 获取方法执行视图

Java-debug-tool通过下面的信息来向调试者呈现出一次方法执行的视图：

- 正在调试的方法信息
- 方法调用堆栈
- 调试耗时，包括对目标JVM造成的STW时间
- 方法入参，包括入参的类型及参数值
- 方法的执行路径
- 代码执行耗时
- 局部变量信息
- 方法返回结果
- 方法抛出的异常
- 对象字段值快照

下图展示了Java-debug-tool获取到正在运行的方法的执行视图的信息

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155511372-1764753977.png)





### Java-debug-tool与同类产品对比分析

Java-debug-tool的同类产品主要是greys，其他类似的工具大部分都是基于greys进行的二次开发，所以直接选择greys来和Java-debug-tool进行对比

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155512151-635229710.jpg)

本文详细剖析了Java动态调试关键技术的实现细节，并介绍了我们基于Java动态调试技术结合实际故障排查场景进行的一点探索实践；动态调试技术为研发人员进行线上问题排查提供了一种新的思路，我们基于动态调试技术解决了传统断点调试存在的问题，使得可以将断点调试这种技术应用在线上，以线下调试的思维来进行线上调试，提高问题排查效率







# GC调优

> GC调优：指的是对垃圾回收（Garbage Collection）进行调优。GC调优的主要目标是避免由垃圾回收引起程序性能下降。
>
> GC调优无唯一的标准答案，如何调优与硬件、程序本身、使用情况均有关系。本章节主要说明的是需要的工具和基本方法。



**GC调优的核心分成三部分**：

1. 通用JVM参数的设置。
2. 特定垃圾回收器的JVM参数的设置。
3. 解决由频繁的Full GC引起的程序性能问题。



**GC调优的核心指标**：

1. **吞吐量（Throughput）**：吞吐量分为业务吞吐量和垃圾回收吞吐量。

- **业务吞吐量**：指的在一段时间内，程序需要完成的业务数量。保证高吞吐量的常规手段有两条：

一是：优化业务执行性能，减少单次业务的执行时间。

二是：优化垃圾回收吞吐量。

- **垃圾回收吞吐量**：指的是 CPU 用于执行用户代码的时间与 CPU 总执行时间的比值，即吞吐量 = 执行用户代码时间 /（执行用户代码时间 + GC时间）。吞吐量数值越高，垃圾回收的效率就越高，允许更多的CPU时间去处理用户的业务，相应的业务吞吐量也就越高。

2. **延迟**：指的是从用户发起一个请求到收到响应这其中经历的时间。延迟 = GC延迟 + 业务执行时间。

3. **内存使用量**：指的是Java应用占用系统内存的最大值。一般通过JVM参数调整，在满足上述两个指标的前提下，这个值越小越好。

> 企业中所要追求的便是：高吞吐量、低延迟、低内存使用量。



**GC调优的一般步骤**：和前面玩的排错思路是一套的

1. **发现问题**：通过监控工具尽可能早地发现GC时间过长、频率过高的现象。

相关工具：

- JDK自带的jstat（垃圾回收统计）。此工具在前面 [jstat：总结垃圾回收统计](#jstat：总结垃圾回收统计) 已见过。主要命令如下：

> 缺点：无法精确到GC产生的时间，只能用于判断GC是否存在问题。

```bash
# jstat -gcutil pid interval(间隔，单位ms) [统计次数]
jstat -gcutil 2815 1000



C	代表Capacity容量，U代表Used使用量

S – 幸存者区	E – 伊甸园区	O – 老年代		M – 元空间

YGC、YGT：年轻代GC次数和GC耗时（单位：秒）

FGC、FGCT：Full GC次数和Full GC耗时

GCT：GC总耗时
```

- visualvm的插件：visualGC。可以通过visualvm -> tools -> plugin安装，也可以在IDEA中搜索该插件安装。

> 适合开发时使用，缺点：对程序运行性能有一定影响，且生产环境我们一般没有相应权限进行操作。

- 生成GC日志：可以更好的看到垃圾回收细节上的数据，同时也可以根据每款垃圾回收器的不同特点更好地发现存在的问题。

```txt
使用方法（JDK 8及以下）：-XX:+PrintGCDetails -Xloggc:文件名


使用方法（JDK 9+）：-Xlog:gc*:file=文件名
```

- [GC Viewer](https://github.com/chewiebug/GCViewer) ：是一个将GC日志转换成可视化图表的小工具。使用方式如下：

```bash
java -jar gcviewer_1.3.4.jar 日志文件.log
```

- [GCeasy](https://gceasy.io/)：业界首款使用AI机器学习技术在线进行GC分析和诊断的工具。可定位内存泄漏、GC延迟高的问题，提供JVM参数优化建议，支持在线的可视化工具图表展示。此工具进入网址选择日志文件（支持tar、zip），之后点击分析，最后就会生成分析报告。

- Prometheus + Grafana：这个玩意儿可以作为GC调优的工具，也可以用来做内存分析，但是，要整知识内容的话又很麻烦和费笔墨，因此，自行去网上找相关资料学习（可以直接去这里：https://www.cnblogs.com/chanshuyi/category/1862951.html）。PS：大厂中这个搭配是肯定会见到的，搭建一般是运维人员弄好，开发人员要学会怎么去看这个玩意儿。

2. **诊断问题**：通过分析工具，诊断问题产生的原因。
3. **修复问题**：调整JVM参数 或 修复源代码中的问题。
4. **测试验证**：在测试环境运行之后获得GC数据，验证问题是否解决。





**常见的GC模式**：可通过该情况大致判定自己的GC情况是属于哪一种，从而更快知道是什么类型原因导致的。

1. **正常情况**

特点：呈现锯齿状，对象创建之后内存上升，一旦发生垃圾回收之后下降到底部，并且每次下降之后的内存大小接近，存留的对象较少

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240111221756086-1997805769.png" alt="image-20240111221800236" />



2. **缓存对象过多**

特点：呈现锯齿状，对象创建之后内存上升，一旦发生垃圾回收之后下降到底部，并且每次下降之后的内存大小接近，处于比较高的位置。

问题产生原因： 程序中保存了大量的缓存对象，导致GC之后无法释放，可以使用MAT或者HeapHero等工具进行分析内存占用的原因。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240111221856601-2001887185.png" alt="image-20240111221901649" />

3. **内存泄漏**

特点：呈现锯齿状，每次垃圾回收之后下降到的内存位置越来越高，最后由于垃圾回收无法释放空间导致对象无法分配产生OutOfMemory的错误。

问题产生原因： 程序中保存了大量的内存泄漏对象，导致GC之后无法释放，可以使用MAT或者HeapHero等工具进行分析是哪些对象产生了内存泄漏。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240111221951275-900813531.png" alt="image-20240111221956476" />

4. **持续的Full GC**

特点：在某个时间点产生多次Full GC，CPU使用率同时飙高，用户请求基本无法处理。一段时间之后恢复正常。

问题产生原因： 在该时间范围请求量激增，程序开始生成更多对象，同时垃圾收集无法跟上对象创建速率，导致持续地在进行FULL GC。GC分析报告。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240111222111709-648808512.png" alt="image-20240111222116827" />



5. **元空间不足导致的Full GC**

特点：堆内存的大小并不是特别大，但是持续发生Full GC。

问题产生原因： 元空间大小不足（对象创建的速度比垃圾回收的速度快），导致持续FULLGC回收元空间的数据。GC分析报告。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240111222317902-1251720853.png" alt="image-20240111222323190" />





## 解决GC问题的手段

下面前三种是比较推荐的手段，第四种仅在前三种无法解决时选用：

1. **优化基础JVM参数**：基础JVM参数的设置不当，会导致频繁 FULLGC的产生。
2. **减少对象产生**：大多数场景下的FULL GC是由于对象产生速度过快导致的， 减少对象产生可以有效的缓解FULL GC的发生。这个解决方式去看前面：[Java内存问题排查](#Java内存问题排查)。
3. **更换垃圾回收器**：选择适合当前业务场景的垃圾回收器，减少延迟、提高吞吐量。这个解决方式去看前面的：[垃圾回收器](#垃圾回收器)。
4. **优化垃圾回收器参数**：优化垃圾回收器的参数， 能在一定程度上提升GC 效率。





### 优化基础JVM参数

#### JVM参数

> 怎么找JDK对应版本所支持的所有JVM参数？
>
> - 进入网址选择自己要的JDK版本：https://docs.oracle.com/en/java/javase/index.html
> - 找到JavaSE Tools Reference：版本不同名字有区别，基本都是这个名字 或者 Tools -> [JDK] Tools Reference / Specifications。
> - 找到 java选项 点进去即可。
>
> 下列是JDK8的常见JVM参数说。

- **-Xmx 和 –Xms**

-Xmx参数：设置的是最大堆内存，但是由于程序是运行在服务器或者容器上，计算可用内存时，要将元空间、操作系统、其它软件占用的内存排除掉。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240111223146580-927882503.png" alt="image-20240111223151358" style="zoom: 67%;" />

如： 服务器内存4G，操作系统+元空间最大值+其它软件占用1.5G，所以4G - 1.5G = 2.5G，因此该值设置为 -Xmx2g 即可。

> 注：最合理的设置方式应该是根据最大并发量估算服务器的配置，然后再根据服务器配置计算最大堆内存的值。

-Xms参数：用来设置初始堆大小，建议将-Xms设置的和-Xmx一样大，有以下几点好处：

1. 运行时性能更好：堆的扩容是需要向操作系统申请内存的，这样会导致程序性能短期下降。
2. 可用性问题：如果在扩容时其他程序正在使用大量内存，很容易因为操作系统内存不足分配失败。
3. 启动速度更快：[Oracle官方文档](https://docs.oracle.com/cd/E15289_01/JRPTG/tune_startup.htm) 的原话：如果初始堆太小，Java 应用程序启动会变得很慢，因为 JVM 被迫频繁执行垃圾收集，直到堆增长到更合理的大小。为了获得最佳启动性能，请将初始堆大小设置为与最大堆大小相同。



- **-Xss**：虚拟机栈大小

```bash
# 示例
-Xss256k
```

如果我们不指定栈的大小，JVM 将创建一个具有默认大小的栈。大小取决于操作系统和计算机的体系结构。比如Linux x86 64位：1MB，如果不需要用到这么大的栈内存，完全可以将此值调小节省内存空间，合理值为 256k – 1m之间。



> 由于JVM底层设计极为复杂，一个参数的调整也许让某个接口得益，但同样有可能影响其他更多接口。因此接下来的参数不建议手动调整。

- **-Xmn**：年轻代的大小，默认值为整个堆的1/3

调整此参数的目的：可以根据峰值流量计算最大的年轻代大小，尽量让对象只存放在年轻代，不进入老年代。

但是实际的场景中，接口的响应时间、创建对象的大小、程序内部还会有一些定时任务等不确定因素都会导致这个值的大小。并不能仅凭计算得出，如果设置该值则要进行大量的测试。G1垃圾回收器尽量不要设置该值，G1会动态调整年轻代的大小。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240111230946661-693250872.png" alt="image-20240111230950850" />



- **-XX:NewRatio**：设置新生代与老年代比值。默认值为2

-XX:NewRatio=4 表示新生代与老年代所占比例为1:4 ，新生代占比整个堆的五分之一。如果设置了-Xmn的情况下，该参数是不需要在设置的。



- **-XX:MaxMetaspaceSize 和 –XX:MetaspaceSize**

-XX:MaxMetaspaceSize=值 参数：指的是最大元空间大小，默认值比较大，如果出现元空间内存泄漏会让操作系统可用内存不可控，建议根据测试情况设置最大值，一般设置为256m。

-XX:MetaspaceSize=值 参数：指的是到达这个值之后会触发Full GC（不是元空间初始大小）， 后续什么时候再触发JVM会自行计算。==如果设置为和MaxMetaspaceSize一样大，就不会Full GC，但是元空间中能回收的对象也无法回收==。



- **-XX:MaxTenuringThreshold**：最大晋升阈值。年龄大于此值之后，会进入老年代。另外JVM有动态年龄判断机制：将年龄从小到大的对象占据的空间加起来，如果大于survivor区域的50%，然后把等于或大于该年龄的对象， 放入到老年代。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240111231159647-1710490965.png" alt="image-20240111231204744" />



- **-XX:SurvivorRatio**：伊甸园区和幸存者区的大小比例，默认值为8。

两个Subrvivor区与一个Eden区的比值为2:8，一个Survivor区占整个新生代的十分之一。



- **-XX:+AggressiveOpts**：编译速度加快

[Oracle官网](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BGBCIEFC) 原话：允许使用积极的性能优化功能，预计这些功能将在即将发布的版本中成为默认功能。默认情况下，此选项处于禁用状态，并且不使用实验性性能功能。





#### 垃圾回收参数

- **-XX:+UseSerialGC**：新生代、老年代都使用串行回收器。现在基本很少使用。



- **-XX:+UseParNewGC**：新生代使用ParNew（多线程 / 并行）回收器， 老年代使用串行回收器（即Serial Old）。

默认开启的线程数量与 CPU 数量相同，可以使用 `-XX:ParallelGCThreads` 参数来设置线程数。

- **-XX:+UseConcMarkSweepGC**：新生代使用并行（ParNew），老年代使用CMS。
- **-XX:+UseCmsInitiatingOccupancyOnly**：使用手动触发或者自定义触发cms 收集，同时也会禁止hostspot 自行触发CMS GC。默认情况下，此选项被禁用。
- **-XX:CMSInitiatingOccupancyFraction**：老年代大小达到了百分之几的阈值后自动进行CMS回收老年代。在JDK8中，此参数默认值是-1。

> 注意：该参数设置完是不会生效的，必须开启 `-XX:+UseCmsInitiatingOccupancyOnly` 参数。
>
> 同时：该参数是通过另外几个参数计算出的阈值
>
> - （（100 - MinHeapFreeRatio）+（double）（CMSTriggerRatio * MinHeapFreeRatio）/ 100.0）

```bash
# 示例：老年代占用率使用了 20%：则使用CMS回收老年代
-XX:CMSInitiatingOccupancyFraction=20
```

- **-XX:MinHeapFreeRatio**：GC后允许的最小可用堆空间百分比。如果可用堆空间低于该值，则堆将被扩展。JDK8中默认值为 40%。

> `-XX:+PrintFlagsInitial` 参数：显示在处理参数之前所有可设置的参数及它们的值，然后直接退出程序。

```bash
# 示例：将最小空闲堆比率设置为 25%：
-XX:MinHeapFreeRatio=25
```

- **-XX:CMSTriggerRatio**：设置在 CMS 收集周期开始之前分配的 `-XX:MinHeapFreeRatio` 指定值的百分比。JDK8中此值默认值为80%。

```bash
-XX:CMSTriggerRatio=75
```



- **-XX:+DisableExplicitGC**：禁止在代码中使用System.gc()

System.gc()可能会引起Full GC，在代码中尽量不要使用。使用 DisableExplicitGC参数可以禁止使用System.gc()方法调用。



- **-XX:+HeapDumpOnOutOfMemoryError**：发生OutOfMemoryError错误时，自动生成hprof内存快照文件。
  **-XX:HeapDumpPath=<path>**：指定hprof文件的输出路径。



- **打印GC日志**

```ba
JDK8及之前 ： -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:文件路径

JDK9及之后 ： -Xlog:gc*:file=文件路径

-XX:+PrintGCDateStamps：将时间和日期也加入到GC日志中。默认是禁用的
```



参数模板：

```bash
-Xms1g													# 初始堆内存1g
-Xmx1g													# 最大堆内存1g
-Xss256k												# 每个线程的栈内存最大256k
-XX:MaxMetaspaceSize=512m								# 最大元空间大小512m
-XX:+DisableExplicitGC									# 代码中System.gc()无效
-XX:+HeapDumpOnOutOfMemoryError							# OutOfMemory错误时生成堆内存快照
-XX:HeapDumpPath=/opt/dumps/my-service.hprof			# 堆内存快照生成位置
-XX:+PrintGCDetails										# 打印详细垃圾回收日志
-XX:+PrintGCDateStamps									# 打印垃圾回收时间
-Xloggc:文件路径										 # 日志文件输出位置
```

注意：

- JDK9及之后gc日志输出修改为 `-Xlog:gc*:file=文件名`。
- 堆内存大小和栈内存大小根据实际情况灵活调整。







# 性能调优

> 这里面的内容其实在前面知识体系中已经概括进去了，这里单独弄出来方便初学者理清思路。



> 性能调优的思路

和内存排查、GC调优思路差不多。因为修复问题和测试验证需要具体开发场景演示，因此这里主要说明发现问题和诊断问题。

1. **发现问题**：通过监控、测试工具发现性能问题。
2. **诊断问题**：通过分析工具定位到某一部分代码存在的性能问题。
3. **修复问题**：内存调优、GC调优、业务代码优化、SQL优化、架构优化等等。
4. **测试验证**：在测试环境运行之后验证问题是否解决。



> 较常见的性能问题现象

1. **CPU占有率高**：通过top命令查看CPU占用率高，接近100甚至多核CPU下超过100都是有可能的。
2. **服务处理时间特别长**：请求单个服务处理时间特别长，多服务（微服务）可以直接使用 [Apache SkyWalking](https://skywalking.apache.org/) 等监控系统来判断是哪一个环节性能低下。这个问题可以直接去前面的Arthas的实践内容：[重要：找到最耗时的方法调用?](#重要：找到最耗时的方法调用?)
3. **线程被耗尽问题**：程序启动之后运行正常，但是在运行一段时间之后无法处理任何的请求（内存和GC正常）。



> 线程转储（Thread Dump）：对所有运行中的线程当前状态的快照。
>
> 线程转储可以通过jstack、visualvm等工具获取。其中包含了线程名、优先级、线程ID、线程状态、线程栈信息等等内容，可以用来解决CPU占用率高、死锁等问题。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240113182532429-342733415.png" alt="image-20240113182538277" />

线程转储（Thread Dump）中的几个核心内容：

1. **名称**： 线程名称，通过给线程设置合适的名称更容易“见名知意”。
2. **优先级**（prio）：线程的优先级。
3. **Java ID**（tid）：JVM中线程的唯一ID。
4. **本地** **ID** (nid)：操作系统分配给线程的唯一ID。
5. **状态**：线程的状态，分为

- NEW – 新创建的线程，尚未开始执行。
- RUNNABLE –正在运行或准备执行。
- BLOCKED – 等待获取监视器锁以进入或重新进入同步块/方法。
- WAITING – 等待其他线程执行特定操作，没有时间限制。
- TIMED_WAITING – 等待其他线程在指定时间内执行特定操作。
- TERMINATED – 已完成执行

6. 栈追踪： 显示整个方法的栈帧信息



线程转储的可视化在线分析平台：

1. https://jstack.review/
2. https://fastthread.io/





## CPU占有率高问题的大致思路

> 这里说明的是最基础的一种，也麻烦的方式，只起引导思路的作用。

1. 通过 `top –c` 命令找到CPU占用率高的进程，获取它的进程ID。

![image-20240113182955199](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240113182949128-120502860.png)

2. 使用 `top -p 进程ID` 单独监控某个进程，按H查看所有的线程以及线程对应的CPU使用率，找到CPU使用率特别高的线程，获取此线程的线程ID（即下图PID，上一步获取的是进程ID，这里查看的是某进程的所有线程）。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240113183230494-2096379167.png" alt="image-20240113183235931" />

3. 使用 `jstack 进程ID` 命令查看所有线程正在执行的栈信息。使用 `jstack 进 程ID > 文件名` 保存到文件中方便查看。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240113183406078-1303275662.png" alt="image-20240113183411823" />

4. 找到nid与线程ID相同的栈信息，从而就可以找到栈信息对应的源代码位置了。

> 注意：需要将之前记录下的十进制线程ID号（第2步获取的）转换成16进制。
>
> 可以通过计算器，也可以通过 `printf ‘%x\n’ 线程ID` 命令直接获得16进制下的线程ID。



另外什么更简单、方法嵌套排查、性能问题等等直接去前面的Arthas的内容：[Arthas场景实战](#Arthas场景实战)。





## 线程被耗尽问题

> 线程耗尽问题：程序在启动运行一段时间之后，就无法接受任何请求了。将程序重启之后继续运行，依然会出现相同的情况。

线程耗尽问题，一般是由于执行时间过长：

1. 检测是否有死锁产生，无法自动解除的死锁会将线程永远阻塞。
2. 若无死锁，再使用前面“CPU占有率高的问题”中的打印线程栈的方法检测线程正在执行哪个方法，一般这些大量出现的方法就是慢方法。



> 线程死锁问题的定位方式

1. **第一种方式（适合线上环境）**：`jstack -l 进程ID > 文件名` 将线程栈保存到本地。在文件中搜索"deadlock"即可找到死锁位置。
2. **第二种方式（适合开发与测试环境使用）**：使用visual vm或者Jconsole工具。可以自动检测出死锁问题。生产环境这种方式一般没相应权限。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240113193453671-1607359720.png" alt="image-20240113193459841" />

3. **第三种方式（适合开发与测试环境）**：生成线程转储文件（Thread Dump），然后使用 [fastthread](https://fastthread.io/) 自动检测线程问题







## 如何准确判断一个方法耗时多久

这个问题会想到什么方式？

1. 方法上打印开始时间和结束时间，他们的差值就是方法的执行耗时。
2. 通过postman或者jmeter发起一笔请求，在控制台上看输出的时间。或者使用前面Arthas的 [重要：找到最耗时的方法调用?trace](#重要：找到最耗时的方法调用?)。

> 这样做是不准确的。
>
> 第一：测试时有些对象创建是懒加载的，所以会影响第一次的请求时间；
>
> 第二：因为虚拟机中JIT（即时编译器）会优化你的代码，所以上面的测试方式得出的时间并不一定是最终用户处理的时间。

OpenJDK中提供了一款叫 [JMH](https://github.com/openjdk/jmh)（Java Microbenchmark Harness）的工具，可以准确地对Java代码进行基准测试，量化方法的执行性能。

JMH会首先执行预热过程，确保JIT对代码进行优化之后再进行真正的迭代测试，最后输出测试的结果。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240113215545233-1315756950.png" alt="image-20240113215550673" />

> 简单搭建JMH环境

1. 依赖：版本看GitHub官网 [JMH](https://github.com/openjdk/jmh)

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>${jmh.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>${jmh.version}</version>
    <scope>test</scope>
</dependency>
```

2. 编写测试代码

```java
import org.openjdk.jmh.annotations.*;

import java.util.concurrent.TimeUnit;

// 会将测试方法（@Benchmark标识的方法）执行5轮预热，每次持续1秒
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
// 启动多少个进程      jvmArgsAppend追加JVM参数
@Fork(value = 1, jvmArgsAppend = {"-Xms1g", "-Xmx1g"})
// 指定显示结果   可选：平均时间（Mode.AverageTime 单位纳秒）、吞吐量（Mode.Throughput）、所有内容（Mode.All）等
@BenchmarkMode(Mode.AverageTime)
// 指定显示结果的单位    方法执行时间快就纳秒、慢就毫秒.........
@OutputTimeUnit(TimeUnit.NANOSECONDS)
// 变量共享范围   整个测试环境中共享（Scope.Benchmark）、单个线程中共享（Scope.Thread）
@State(Scope.Benchmark)
public class HelloWorldBench {

    @Benchmark  // 标志是一个测试方法
    public int test1() {
        int i = 0;
        i++;
		/*
        * 需要将变量返回。因为上面的i没用（即：死代码），所以JIT就会将其去掉，从而造成实际测试时间的出入
        *
        * 要是有多个变量怎么做？
        *   使用黑洞方法：new Blackhole().consume(...variable);
        * */
        return i;
    }
}
```

3. 开始测试

> 测试结果可以通过 https://jmh.morethan.io/ 转为可视化。格式是JSON格式(`new OptionsBuilder().resultFormat(ResultFormatType.JSON)`可以构建为JSON格式)

方式一（推荐）：通过maven的verify命令，检测代码问题并打包成jar包。然后通过 `java -jar 打成的jar包` 命令执行基准测试。

方式二：使用Java代码。但是，这种方式测试出来的时间会有出入。

```java
public static void main(String[] args) throws RunnerException {
    Options opt = new OptionsBuilder()
        // 要测试的类
        .include(HelloWorldBench.class.getSimpleName())
        // 结果的格式
        .resultFormat(ResultFormatType.JSON)
        // 启用多少进程（当前main方法使用）   jar就是用类上的@Fork(value = 1, jvmArgsAppend = {"-Xms1g", "-Xmx1g"})
        .forks(1)
        .build();

    new Runner(opt).run();
}
```

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240114163632057-473509994.png" alt="image-20240114163638444" />



更多openjdk jmh专业场景示例去官网：[jmh-samples](https://github.com/openjdk/jmh/tree/master/jmh-samples/src/main/java/org/openjdk/jmh/samples)





## 综合案例：Java层面性能调优思路

1. 假设从数据库获取的数据如下

```java
package com.zixieqing.jvmoptimize.performance.practice.dao.impl;

import com.zixieqing.jvmoptimize.performance.practice.dao.UserDao;
import com.zixieqing.jvmoptimize.performance.practice.entity.UserDetails;
import org.springframework.stereotype.Repository;

import javax.annotation.PostConstruct;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

@Repository
public class UserDaoImpl implements UserDao {

    // 提前准备好数据
    private List<UserDetails> users = new ArrayList<>();

    @PostConstruct
    public void init(){
        // 初始化时生成数据		这里模拟获取的数据很多
        for (int i = 1; i <= 10000; i++) {
            users.add(new UserDetails((long) i, LocalDateTime.now(),new Date()));
        }
    }

    /*
       模拟下数据库查询接口，主要目的是只优化Java代码，屏蔽SQL相关的内容
     */
    @Override
    public List<UserDetails> findUsers() {
        return users;
    }
}
```

```java
package com.zixieqing.jvmoptimize.performance.practice.service.impl;

import com.zixieqing.jvmoptimize.performance.practice.dao.UserDao;
import com.zixieqing.jvmoptimize.performance.practice.entity.User;
import com.zixieqing.jvmoptimize.performance.practice.entity.UserDetails;
import com.zixieqing.jvmoptimize.performance.practice.service.UserService;
import org.apache.commons.lang3.RandomStringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.annotation.PostConstruct;
import java.util.ArrayList;
import java.util.List;

@Service
public class UserServiceImpl implements UserService {

    private List<User> users = new ArrayList<>();

    @PostConstruct
    public void init(){
        // 初始化时生成数据
        for (int i = 1; i <= 10000; i++) {
            users.add(new User((long) i, RandomStringUtils.randomAlphabetic(10)));
        }
    }


    @Autowired
    private UserDao userDao;

    @Override
    public List<UserDetails> getUserDetails() {
        return userDao.findUsers();
    }

    @Override
    public List<User> getUsers() {
        return users;
    }
}
```

2. 假设服务器中程序员写的代码如下，还未优化

```java
package com.zixieqing.jvmoptimize.performance.practice.controller;

import com.zixieqing.jvmoptimize.performance.practice.entity.User;
import com.zixieqing.jvmoptimize.performance.practice.entity.UserDetails;
import com.zixieqing.jvmoptimize.performance.practice.service.UserService;
import com.zixieqing.jvmoptimize.performance.practice.vo.UserVO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.text.SimpleDateFormat;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/puser")
public class UserController {

    @Autowired
    private UserService userService;

    private final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    // 初始代码
    public List<UserVO> user1(){
        // 1.从数据库获取前端需要的详情数据
        List<UserDetails> userDetails = userService.getUserDetails();

        // 2.获取缓存中的用户数据
        List<User> users = userService.getUsers();

        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        // 3.遍历详情集合，从缓存中获取用户名，生成VO进行填充
        ArrayList<UserVO> userVOS = new ArrayList<>();
        // 嵌套循环
        for (UserDetails userDetail : userDetails) {
            UserVO userVO = new UserVO();
            // 可以使用BeanUtils对象拷贝
            userVO.setId(userDetail.getId());
            userVO.setRegister(simpleDateFormat.format(userDetail.getRegister2()));
            // 填充name
            for (User user : users) {
                if(user.getId().equals(userDetail.getId())){
                    userVO.setName(user.getName());
                }
            }
            // 加入集合
            userVOS.add(userVO);
        }

        return userVOS;

    }
}
```

3. 使用Arthas的trace定位性能

```bash
# trace 类名 方法名 --skipJDKMethod false

trace com.zixieqing.jvmoptimize.performance.practice.controller.UserController user1 --skipJDKMethod false
```

![image-20240114174734995](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240114174728126-467498094.png)

发现问题就出在上面的循环嵌套里面（第2步user1()方法），共循环了（count）100 0000次。

通过多次对2中代码的优化、多次3的测试，发现的问题如下：

- 两次循环，则减少循环次数：去掉内层循环，使用map

```java
        // 2.获取缓存中的用户数据
        List<User> users = userService.getUsers();        

		// 将list转换成hashmap
        HashMap<Long, User> map = new HashMap<>();
        for (User user : users) {
            map.put(user.getId(), user);
        }

        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        // 3.遍历详情集合，从缓存中获取用户名，生成VO进行填充
        ArrayList<UserVO> userVOS = new ArrayList<>();
        for (UserDetails userDetail : userDetails) {
            UserVO userVO = new UserVO();
            // 可以使用BeanUtils对象拷贝
            userVO.setId(userDetail.getId());
            userVO.setRegister(simpleDateFormat.format(userDetail.getRegister2()));
            // 填充name
            userVO.setName(map.get(userDetail.getId()).getName());
            // 加入集合
            userVOS.add(userVO);
        }




```

- 优化日期格式化：

Date对象（`new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());`）：的SimpleDateFormatter是线程不安全的，所以每次需要重新创建对象。优化的方式：将对象放入ThreadLocal中进行保存。

LocalDateTime对象（`LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));`）：DateTimeFormatter线程安全，并且性能较好。

测试代码：

```java
import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.results.format.ResultFormatType;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.text.SimpleDateFormat;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Date;
import java.util.concurrent.TimeUnit;

// 执行5轮预热，每次持续1秒
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
// 执行一次测试
@Fork(value = 1, jvmArgsAppend = {"-Xms1g", "-Xmx1g"})
// 显示平均时间，单位纳秒
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@State(Scope.Thread)
public class DateBench {

    private static String sDateFormatString = "yyyy-MM-dd HH:mm:ss";
    private Date date = new Date();
    private LocalDateTime localDateTime = LocalDateTime.now();
    private static ThreadLocal<SimpleDateFormat> simpleDateFormatThreadLocal = new ThreadLocal();
    private static final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    @Setup
    public void setUp() {

        SimpleDateFormat sdf = new SimpleDateFormat(sDateFormatString);
        simpleDateFormatThreadLocal.set(sdf);
    }

    @Benchmark
    public String date() {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat(sDateFormatString);
        return simpleDateFormat.format(date);
    }

    @Benchmark
    public String localDateTime() {
        return localDateTime.format(formatter);
    }
    @Benchmark
    public String localDateTimeNotSave() {
        return localDateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
    }

    @Benchmark
    public String dateThreadLocal() {
        return simpleDateFormatThreadLocal.get().format(date);
    }


    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(DateBench.class.getSimpleName())
                .resultFormat(ResultFormatType.JSON)
                .forks(1)
                .build();

        new Runner(opt).run();
    }
}
```

将JSON结果用 https://jmh.morethan.io/ 转为可视化的对比结果如下：

![image-20240114201842422](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240114201835675-70171434.png)

可见如下方式的LocalDateTime胜出：

```java
private LocalDateTime localDateTime = LocalDateTime.now();
private static final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

@Benchmark
public String localDateTime() {
    return localDateTime.format(formatter);
}
```

- 使用并发流（`xxxx.parallelStream()`，`xxxxx.Stream()`的方式性能低）优化外层循环

所以经过上面的优化之后，源代码就被优化为如下的形式：

```java
package com.zixieqing.jvmoptimize.performance.practice.controller;

import com.zixieqing.jvmoptimize.performance.practice.entity.User;
import com.zixieqing.jvmoptimize.performance.practice.entity.UserDetails;
import com.zixieqing.jvmoptimize.performance.practice.service.UserService;
import com.zixieqing.jvmoptimize.performance.practice.vo.UserVO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.text.SimpleDateFormat;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/puser")
public class UserController {

    @Autowired
    private UserService userService;

    private final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    // 初始代码
    // public List<UserVO> user1() {
    //     // 1.从数据库获取前端需要的详情数据
    //     List<UserDetails> userDetails = userService.getUserDetails();
    //
    //     // 2.获取缓存中的用户数据
    //     List<User> users = userService.getUsers();
    //
    //     SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    //     // 3.遍历详情集合，从缓存中获取用户名，生成VO进行填充
    //     ArrayList<UserVO> userVOS = new ArrayList<>();
    //     for (UserDetails userDetail : userDetails) {
    //         UserVO userVO = new UserVO();
    //         // 可以使用BeanUtils对象拷贝
    //         userVO.setId(userDetail.getId());
    //         userVO.setRegister(simpleDateFormat.format(userDetail.getRegister2()));
    //         // 填充name
    //         for (User user : users) {
    //             if (user.getId().equals(userDetail.getId())) {
    //                 userVO.setName(user.getName());
    //             }
    //         }
    //         // 加入集合
    //         userVOS.add(userVO);
    //     }
    //
    //     return userVOS;
    //
    // }

    // 使用并行流优化性能
    public List<UserVO> user5() {
        // 1.从数据库获取前端需要的详情数据
        List<UserDetails> userDetails = userService.getUserDetails();

        // 2.获取缓存中的用户数据
        List<User> users = userService.getUsers();
        // 将list转换成hashmap
        Map<Long, User> map = users.parallelStream().collect(Collectors.toMap(User::getId, o -> o));

        // 3.遍历详情集合，从缓存中获取用户名，生成VO进行填充
        return userDetails.parallelStream().map(userDetail -> {
            UserVO userVO = new UserVO();
            // 可以使用BeanUtils对象拷贝
            userVO.setId(userDetail.getId());
            userVO.setRegister(userDetail.getRegister().format(formatter));
            // 填充name
            userVO.setName(map.get(userDetail.getId()).getName());
            return userVO;
        }).collect(Collectors.toList());

    }
}
```

4. JMH再次测试

```java
package com.zixieqing.jvmoptimize;

import com.zixieqing.jvmoptimize.performance.practice.controller.UserController;
import org.junit.jupiter.api.Test;
import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.infra.Blackhole;
import org.openjdk.jmh.results.format.ResultFormatType;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.OptionsBuilder;
import org.springframework.boot.SpringApplication;
import org.springframework.context.ApplicationContext;

import java.io.IOException;
import java.util.concurrent.TimeUnit;

// 执行5轮预热，每次持续1秒
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
// 执行一次测试
@Fork(value = 1, jvmArgsAppend = {"-Xms1g", "-Xmx1g"})
// 显示平均时间，单位纳秒
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@State(Scope.Benchmark)
public class PracticeBenchmarkTest {

    private UserController userController;
    private ApplicationContext context;

    // 初始化将springboot容器启动 端口号随机
    @Setup
    public void setup() {
        this.context = new SpringApplication(JvmOptimizeApplication.class).run();
        userController = this.context.getBean(UserController.class);
    }

    // 启动这个测试用例进行测试
    @Test
    public void executeJmhRunner() throws RunnerException, IOException {

        new Runner(new OptionsBuilder()
                .shouldDoGC(true)
                .forks(0)
                .resultFormat(ResultFormatType.JSON)
                .shouldFailOnError(true)
                .build()).run();
    }

    // 用黑洞消费数据，避免JIT消除代码
    @Benchmark
    public void test1(final Blackhole bh) {
        bh.consume(userController.user1());
    }

    @Benchmark
    public void test2(final Blackhole bh) {

        bh.consume(userController.user2());
    }

    @Benchmark
    public void test3(final Blackhole bh) {

        bh.consume(userController.user3());
    }

    @Benchmark
    public void test4(final Blackhole bh) {

        bh.consume(userController.user4());
    }

    @Benchmark
    public void test5(final Blackhole bh) {

        bh.consume(userController.user5());
    }
}
```

JMH测试对比结果如下：

![image-20240114202925544](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240114202918632-2010969851.png)

> 若是需要再提升，追求更高的性能、更小的CPU和内存开销，则可以使用Oracle提供的另一款高性能JDK：[GraalVM](https://www.graalvm.org/) 。这玩意儿有两种模式：

1. JIT（Just-In-Time）模式：可跨平台，就是“一次编写，到处运行”。预热之后，通过内置的Graal即时编译器优化热点代码，生成比Hotspot JIT更高性能的机器码。这种模式提高性能，社区版对CPU占有率和内存提高不明显（企业版有显著提高）。

2. AOP（Ahead-Of-Time）模式：不可跨平台，而是为特定平台创建可执行文件，本质是本地镜像（Native Image）。这种模式对启动速度、CPU、内存开销都有了显著提高。但是此模式也有另外问题：

- 跨平台问题：在不同平台下运行需要编译多次。编译平台的依赖库等环境要与运行平台保持一致。
- 消耗大量的CPU和内存：使用框架之后，编译本地镜像的时间比较长，同时也需要消耗大量的CPU和内存。
- AOT 编译器在编译时，需要知道运行时所有可访问的所有类。但是Java中有一些技术可以在运行时创建类，例如反射、动态代理等。这些技术在很多框架比如Spring中大量使用，所以框架需要对AOT编译器进行适配解决类似的问题。

**GraalVM的使用场景**：传统的系统架构有人力、机房开销大；流量激增需要大量服务器，流量过后很多服务器就闲置造成资源浪费，而因为虚拟化技术、云原生技术，所以GraalVM+云服务商提供的Serverless无服务器化架构就是其使用场景了。


























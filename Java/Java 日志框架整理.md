# [#](#JUL) JUL

> 指的是Java Util Logging包，它是Java原生的日志框架，使用时不需要另外引用第三方的类库，相对其他的框架使用方便，学习简单，主要是使用在小型应用中。@紫邪情

## [#](#JUL的组成结构) JUL的组成结构

![image](https://img2023.cnblogs.com/blog/2421736/202304/2421736-20230411030126527-213085672.png)



**Logger**：被称为记录器，应用程序通过获取Logger对象，调用其API来发布日志信息。Logger通常被认为是访问日志系统的入口程序

**Handler**：处理器，每个Logger都会关联一个或者是一组Handler，Logger会将日志交给关联的Handler去做处理，由Handler负责将日志做记录。Handler具体实现了日志的输出位置，比如可以输出到控制台或者是文件中等等

**Filter**：过滤器，根据需要定制哪些信息会被记录，哪些信息会被略过

**Formatter**：格式化组件，它负责对日志中的数据和信息进行转换和格式化，所以它决定了我们输出日志最终的形式

**Level**：日志的输出级别，每条日志消息都有一个关联的级别。我们根据输出级别的设置，用来展现最终所呈现的日志信息。根据不同的需求，去设置不同的级别





## [#](#入门) 入门

可以直接去JDK文档中查看java.util.logging这个包

![image](https://img2023.cnblogs.com/blog/2421736/202304/2421736-20230411030125922-1435401450.png)





```java
public class JULTest {

    @Test
    public void test01(){
        /* 日志入口程序 java.util.logging.Logger */
        
        /* 	Logger对象的创建方式，不能直接new对象
			取得对象的方法参数，需要引入当前类的全路径字符串
				当前先这么用，以后根据包结构有Logger父子关系，后续详细介绍 */
        Logger logger = Logger.getLogger("cn.zixieqing.jul.test.JULTest");

        /*
            对于日志的输出，有两种方式

            第一种方式：
                直接调用日志级别相关的方法，方法中传递日志输出信息
                假设现在我们要输出info级别的日志信息
         */
        //logger.info("输出info信息1");
        
        /*
            第二种方式：
                调用通用的log方法，然后在里面通过Level类型来定义日志的级别参数，以及搭配日志输出信息的参数
         */
        //logger.log(Level.INFO,"输出info信息2");


        /*
            输出学生信息
                姓名
                年龄
         */
        /*String name = "zs";
        int age = 23;
        logger.log(Level.INFO,"学生的姓名为："+name+"；年龄为："+age);*/

        /*
            对于输出消息中，字符串的拼接弊端很多：
                1.麻烦
                2.程序效率低
                3.可读性不强
                4.维护成本高

            我们应该使用动态生成数据的方式，生产日志
            我们使用的就是占位符的方式来进行操作
         */
        String name = "zs";
        int age = 23;
        logger.log(Level.INFO,"学生的姓名：{0},年龄:{1}",new Object[]{name,age});
    }
}
```




## [#](#日志级别) 日志级别

```java
@Test
public void test02(){

    /*
        日志的级别：通过源码查看，非常简单

          SEVERE : 错误 --- 最高级的日志级别
          WARNING : 警告
          INFO : （默认级别）消息
                    源码：Logger.getLogger() --->demandLogger -----> getLogManager() -----> ensureLogManagerInitialized() --->350行左右有一个 defaultlevel属性
          CONFIG : 配置
          FINE : 详细信息（少）
          FINER : 详细信息（中）
          FINEST : 详细信息（多） --- 最低级的日志级别

        两个特殊的级别
           OFF 可用来关闭日志记录
           ALL 启用所有消息的日志记录

        对于日志的级别，重点关注的是new对象的时候的第二个参数，是一个数值(源码中有)
            OFF Integer.MAX_VALUE 整型最大值
            SEVERE 1000
            WARNING 900
            ...
            ...
            FINEST 300
            ALL Integer.MIN_VALUE 整型最小值

        这个数值的意义在于，如果设置的日志的级别是INFO -- 800
        那么最终展现的日志信息，必须是数值大于800的所有的日志信息
        最终展现的就是：
            SEVERE
            WARNING
            INFO
     */

    Logger logger = Logger.getLogger("cn.zixieqing.jul.test.JULTest");

    /*
        通过打印结果，可以看到仅仅只是输出了info级别以及比info级别高的日志信息
        比info级别低的日志信息没有输出出来
        证明了info级别的日志信息，它是系统默认的日志级别
        在默认日志级别info的基础上，打印比它级别高的信息

        如果仅仅只是通过以下形式来设置日志级别
        那么不能够起到效果
        将来需要搭配处理器handler共同设置才会生效
     */

    logger.setLevel(Level.CONFIG);

    logger.severe("severe信息");
    logger.warning("warning信息");
    logger.info("info信息");
    logger.config("config信息");
    logger.fine("fine信息");
    logger.finer("finer信息");
    logger.finest("finest信息");
}
```




## [#](#自定义日志级别) 自定义日志级别

### [#](#输出信息在console上) 输出信息在console上

```java
@Test
public void test03(){

    /* 自定义日志的级别 */
    
    
    // 日志记录器
    Logger logger = Logger.getLogger("cn.zixieqing.jul.test.JULTest");

    /*
        将默认的日志打印方式关闭掉
        参数设置为false，打印日志的方式就不会按照父logger默认的方式去进行操作
    */
    logger.setUseParentHandlers(false);

    /*
        处理器Handler
        这里使用的是控制台日志处理器，取得处理器对象
    */
    ConsoleHandler handler = new ConsoleHandler();
    // 创建日志格式化组件对象
    SimpleFormatter formatter = new SimpleFormatter();

    // 在处理器中设置输出格式
    handler.setFormatter(formatter);
    // 在记录器中添加处理器
    logger.addHandler(handler);

    /* 
        设置日志的打印级别
        此处必须将日志记录器和处理器的级别进行统一的设置，才会达到日志显示相应级别的效果
            logger.setLevel(Level.CONFIG);
            handler.setLevel(Level.CONFIG);
    */

    // 设置如下的all级别之后，那么下面所有的输出信息都会打印到控制台，在需要的时候改成对应的日志级别即可
    logger.setLevel(Level.ALL);
    handler.setLevel(Level.ALL);

    logger.severe("severe信息");
    logger.warning("warning信息");
    logger.info("info信息");
    logger.config("config信息");
    logger.fine("fine信息");
    logger.finer("finer信息");
    logger.finest("finest信息");

}
```



### [#](#输出信息在磁盘上) 输出信息在磁盘上

```java
@Test
public void test04() throws IOException {

    /*
        将日志输出到具体的磁盘文件中
        这样做相当于是做了日志的持久化操作
     */
    Logger logger = Logger.getLogger("cn.zixieqing.jul.test.JULTest");
    logger.setUseParentHandlers(false);

    // 文件日志处理器
    FileHandler handler = new FileHandler("D:\\test\\" + this.getClass().getSimpleName() + ".log");
    SimpleFormatter formatter = new SimpleFormatter();
    handler.setFormatter(formatter);
    logger.addHandler(handler);

    // 也可以同时在控制台和文件中进行打印
    ConsoleHandler handler2 = new ConsoleHandler();
    handler2.setFormatter(formatter);
    // 可以在记录器中同时添加多个处理器，这样磁盘和控制台中都可以输出对应级别的日志信息了
    logger.addHandler(handler2);

    logger.setLevel(Level.ALL);
    handler.setLevel(Level.ALL);
    handler2.setLevel(Level.CONFIG);

    logger.severe("severe信息");
    logger.warning("warning信息");
    logger.info("info信息");
    logger.config("config信息");
    logger.fine("fine信息");
    logger.finer("finer信息");
    logger.finest("finest信息");


    /*
        总结：
            用户使用Logger来进行日志的记录，Logger可以持有多个处理器Handler
            	日志的记录使用的是Logger，日志的输出使用的是Handler

            添加了哪些handler对象，就相当于需要根据所添加的handler将日志输出到指定的位置上，
            	例如控制台、文件..
     */
}
```




## [#](#logger的父子关系) logger的父子关系

```java
@Test
public void test05(){

    /*
        Logger之间的父子关系
            JUL中Logger之间是存在"父子"关系的
            
            值得注意的是，这种父子关系不是我们普遍认为的类之间的继承关系
                关系是通过树状结构存储的
     */



    /*
        从下面创建的两个logger对象看来
        我们可以认为logger1是logger2的父亲
     */

    /* 
        父亲是RootLogger，名称默认是一个空的字符串
        RootLogger可以被称之为所有logger对象的顶层logger
    */
    // 这就是父
    Logger logger1 = Logger.getLogger("cn.zixieqing.jul.test");

    // 这是子，甚至cn.zixieqing.jul也是这个logger2的父
    Logger logger2 = Logger.getLogger("cn.zixieqing.jul.test.JULTest");

    // System.out.println(logger2.getParent()==logger1); //true

    System.out.println("logger1的父Logger引用为:"
            + logger1.getParent() + "; 名称为"+logger1.getName() + "; 父亲的名称为" + logger1.getParent().getName());


    System.out.println("logger2的父Logger引用为:"
            +logger2.getParent() + "; 名称为" + logger2.getName() + "; 父亲的名称为" + logger2.getParent().getName());


    /*
        父亲所做的设置，也能够同时作用于儿子
            对logger1做日志打印相关的设置，然后使用logger2进行日志的打印
     */
    // 父亲做设置
    logger1.setUseParentHandlers(false);
    ConsoleHandler handler = new ConsoleHandler();
    SimpleFormatter formatter = new SimpleFormatter();
    handler.setFormatter(formatter);
    logger1.addHandler(handler);
    handler.setLevel(Level.ALL);
    logger1.setLevel(Level.ALL);

    /* 儿子做打印
    	结果就是：儿子logger2没做配置，但是父亲logger1做了，所以儿子logger2的输出级别就是父亲的级别
	*/
    logger2.severe("severe信息");
    logger2.warning("warning信息");
    logger2.info("info信息");
    logger2.config("config信息");
    logger2.fine("fine信息");
    logger2.finer("finer信息");
    logger2.finest("finest信息");
}
```




> 小结：看源码的结果

```java
// JUL在初始化时会创建一个顶层RootLogger作为所有Logger的父Logger，java.util.logging.LogManager$RootLogger，默认的名称为空串

// 查看源码Logger.getLogger() --->demandLogger -----> getLogManager() -----> ensureLogManagerInitialized（）--->350行左右：
owner.rootLogger = owner.new RootLogger();
// RootLogger是LogManager的内部类

/*
    以上的RootLogger对象作为树状结构的根节点存在的
        将来自定义的父子关系通过路径来进行关联
        父子关系，同时也是节点之间的挂载关系
*/

// 350行左右
owner.addLogger(owner.rootLogger);
addLogger ----> LoggerContext cx = getUserContext(); 
/* 
    LoggerContext	一种用来保存节点的Map关系,
    WeakHashMap<Object, LoggerContext> contextsMap
		点进WeakHashMap<Object, LoggerContext>的LoggerContext，
		里面的结构为：Hashtable<String,LoggerWeakRef> namedLoggers 
*/


// 再点开Hashtable<String,LoggerWeakRef> namedLoggers 的LoggerWeakRef，得到的信息如下：
private String                name;       // for namedLoggers cleanup，这玩意就是节点，也就是说父子关系也就是节点挂载关系
private LogNode               node;       // for loggerRef cleanup
private WeakReference<Logger> parentRef;  // for kids cleanup
```



## [#](#使用配置文件) 使用配置文件

### [#](#默认配置文件所在地) 默认配置文件所在地


查看源码

```json
// Logger.getLogger() --->demandLogger -----> getLogManager() -----> ensureLogManagerInitialized（）--->345行左右：

// 有这么一行代码：owner.readPrimordialConfiguration();

// 点击readPrimordialConfiguration()，在340行左右有一句readConfiguration();

// 点击readConfiguration();在1290行左右，有如下的代码
    String fname = System.getProperty("java.util.logging.config.file");
    if (fname == null) {
        fname = System.getProperty("java.home");
        if (fname == null) {
            throw new Error("Can't find java.home ??");
        }
        File f = new File(fname, "lib");
        f = new File(f, "logging.properties");
        fname = f.getCanonicalPath();
    }
```




> 结论：默认配置文件所在地
>
> java.home（即：安装JDK的目录）--> 找到jre文件夹 --> lib --> logging.properties

分析一下上述目录中的logging.properties文件，找到这个目录下的此文件打开

```properties
# RootManager默认使用的处理器
# 若是想要配置多个处理器，可以使用 逗号 进行分开
# 	如：java.util.logging.ConsoleHandler, java.util.logging.FileHandler
handlers= java.util.logging.ConsoleHandler


# RootManager的默认日志级别，这是全局的日志级别
# 若不手动进行级别配置，那么默认使用INFO及更高的级别进行输出
.level= INFO


# 文件处理器设置
# 日志文件的路径
#     %h/java%u.log		h指的是用户目录	不分window还是linux
#     java%u.log		是生成的文件名		其中：u相当于是自增，从0开始的
#       如：java0.log、java1.log、java2.log......
# 			这里生成多少份由 java.util.logging.FileHandler.count = 1 这个配置决定
java.util.logging.FileHandler.pattern = %h/java%u.log
# 日志文件的限制	默认50000字节
java.util.logging.FileHandler.limit = 50000
# 日志文件的数量	默认1份
java.util.logging.FileHandler.count = 1
# 日志文件的格式	默认XML格式，也可以采用SimpleFormatter
java.util.logging.FileHandler.formatter = java.util.logging.XMLFormatter


# 控制台处理器设置
# 控制台默认级别
java.util.logging.ConsoleHandler.level = INFO
# 控制台默认输出格式
java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter

# 这是这个源文档中举的一个例子，意思是像下面这样，可将日志级别设定到具体的某个包下
# 	com.xyz.foo.level = SEVERE
```



### [#](#修改成更友好点的配置文件) 修改成更友好点的配置文件

```properties
# 自定义Logger
cn.zixieqing.handlers=java.util.logging.FileHandler
# 自定义Logger日志级别
cn.zixieqing.level=SERVER
# 屏蔽父logger的日志设置	相当于前面玩的 logger.setUseParentHandlers(false);
cn.zixieqing.useParentHandlers=false

# RootManager默认使用的处理器
# 若是想要配置多个处理器，可以使用逗号进行分开
# 	如：java.util.logging.ConsoleHandler,java.util.logging.FileHandler
handlers= java.util.logging.ConsoleHandler

# RootManager的默认日志级别，这是全局的日志级别
.level= SERVER

# 文件处理器设置
# 日志文件的路径
#     %h/java%u.log		h指的是用户目录	不分window还是linux
#     java%u.log		是生成的文件名		其中：u相当于是自增，从0开始的
# 		如：java0.log、java1.log、java2.log......
# 		这里生成多少份由 java.util.logging.FileHandler.count = 1 这个配置决定
java.util.logging.FileHandler.pattern = %h/java%u.log
# 日志文件的限制	50000字节
java.util.logging.FileHandler.limit = 50000
# 日志文件的数量
java.util.logging.FileHandler.count = 1
# 日志文件的格式
java.util.logging.FileHandler.formatter = java.util.logging.SimpleFormatter

# 控制台处理器设置
# 控制台默认级别
java.util.logging.ConsoleHandler.level = SERVER
# 控制台默认输出格式
java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter

# 下一次生成的日志文件默认是覆盖了上一此的文件	改为在原日志上进行追加
java.util.logging.FileHandler.append=true
```




### [#](#使用自定义的配置文件) 使用自定义的配置文件

```java
@Test
public void test06() throws Exception {

    // 加载自定义的配置文件
    InputStream input = new FileInputStream("D:\\test\\logging.properties");

    // 取得日志管理器对象
    LogManager logManager = LogManager.getLogManager();

    // 读取自定义的配置文件
    logManager.readConfiguration(input);

    Logger logger = Logger.getLogger("cn.zixieqing.jul.test.JULTest");

    logger.severe("severe信息");
    logger.warning("warning信息");
    logger.info("info信息");
    logger.config("config信息");
    logger.fine("fine信息");
    logger.finer("finer信息");
    logger.finest("finest信息");
}
```



## [#](#总结：JUL原理) 总结：JUL原理


**1、初始化LogManager**
- LogManager加载logging.properties配置文件
- 添加Logger到LogManager

**2、从单例的LogManager获取Logger，即：LogManager.getLogManager();**

**3、设置日志级别Level，在打印的过程中使用到了日志记录的LogRecord类，源码找寻如下：**

 ```java
// 1、点击 logger.severe("severe信息"); 中的severe，当然点击其他warning、info、config也是可以进去的，之后会看到如下的代码
        public void severe(String msg) {
            log(Level.SEVERE, msg);
        }


// 2、点击上述的log()，看到如下的代码
        public void log(Level level, String msg) {
            if (!isLoggable(level)) {
                return;
            }
            // 这里就是目的地
            LogRecord lr = new LogRecord(level, msg);
            doLog(lr);
        }
 ```

**4、Filter作为过滤器提供了日志级别之外更细粒度的控制**

**5、Handler日志处理器，决定日志的输出位置，例如控制台、文件...**

**6、Formatter是用来格式化输出的**




# [#](#LOG4J) LOG4J

> 全称：log for java。	for音通4的英文four。@紫邪情

## [#](#LOG4J的组成) LOG4J的组成

**Loggers (日志记录器)：** 控制日志的输出以及输出级别(JUL做日志级别Level)
**Appenders（输出控制器）：** 指定日志的输出方式（输出到控制台、文件等）
**Layout（日志格式化器）：** 控制日志信息的输出格式



### [#](#Loggers日志记录器) Loggers日志记录器

> 负责收集处理日志记录，实例的命名就是类的全限定名，如：`cn.zixieqing.test`
>
> 同时：Logger的名字大小写敏感，其命名有继承机制，和JUL中的父子关系一样，以包路径来区分的 ；
>
> 另外：root logger是所有logger的根，上辈所做的日志属性设置，会直接地影响到子辈

```java
// root logger的获取
Logger.getRootLogger();
```




> **日志级别**

```txt
DEBUG
INFO
WARN
ERROR
........
```

**大小关系：**ERROR > WARN > INFO > DEBUG



**输出规则：**输出日志的规则是“只输出级别不低于设定级别的日志信息”

如：假设Loggers级别设定为INFO，则INFO、WARN、ERROR级别的日志信息都会输出，而级别比INFO低的DEBUG则不会输出



### [#](#Appenders输出控制器) Appenders输出控制器

> 允许把日志输出到不同的地方，如控制台（Console）、文件（Files）等，可以根据时间或者文件大小产生新的文件，可以以流的形式发送到其它地方等等。@紫邪情


**常用Appenders输出控制器类型:**
1. **ConsoleAppender** 将日志输出到控制台

2. **FileAppender** 将日志输出到文件中

3. **DailyRollingFileAppender**  根据**指定时间**输出到一个新的文件， 将日志输出到一个日志文件

4. **RollingFileAppender**  根据**指定文件大小**，当文件大小达到指定大小时，会自动把文件改名，产生一个新的文件，将日志信息输出到一个日志文件

5. **JDBCAppender** 把日志信息保存到数据库中




### [#](#Layouts日志格式化器) Layouts日志格式化器

> 有时希望根据自己的喜好格式化自己的日志输出，Log4j可以在Appenders的后面附加Layouts来完成这个功能

**常用Layouts类型：**

1. **HTMLLayout**  格式化日志输出为HTML表格形式

2. **SimpleLayout** 简单的日志输出格式化，打印的日志格式如默认INFO级别的消息

3. **PatternLayout**  最强大的格式化组件( **要用LOG4J，那这种类型是用的最多的** ），可以根据自定义格式输出日志，如果没有指定转换格式， 就是用默认的转换格式




> **日志输出格式说明：特别是使用PatternLayout类型时**

用PatternLayout可以自定义格式输出，是我们最常用的方式，这种格式化输出采用类似于 C 语言的 printf 函数的打印格式格式化日志信息

```txt
%m 输出代码中指定的日志信息

%p 输出优先级，及 DEBUG、INFO 等

%n 换行符（Windows平台的换行符为 "\n"，Unix 平台为 "\n"）

%r 输出自应用启动到输出该 log 信息耗费的毫秒数

%c 输出打印语句所属的类的全名

%t 输出产生该日志的线程全名

%d 输出服务器当前时间，默认为 ISO8601，也可以指定格式，如：%d{yyyy年MM月dd日 HH:mm:ss}

%l 输出日志时间发生的位置，包括类名、线程、及在代码中的行数。如：Test.main(Test.java:10)

%F 输出日志消息产生时所在的文件名称

%L 输出代码中的行号

%% 输出一个 "%" 字符
    可以在 % 与字符之间加上修饰符来控制最小宽度、最大宽度和文本的对其方式。如：
    %5c 输出category名称，最小宽度是5，category<5，默认的情况下右对齐

    %-5c 输出category名称，最小宽度是5，category<5，"-"号指定左对齐,会有空格

    %.5c 输出category名称，最大宽度是5，category>5，就会将左边多出的字符截掉，<5不会有空格

    %20.30c category名称<20补空格，并且右对齐，>30字符，就从左边交远销出的字符截掉
```





## [#](#玩LOG4J) 玩LOG4J

### [#](#入门) 入门
依赖

```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

使用


```java
public class Log4jTest01 {

    @Test
    public void test01(){

        // 加载初始化配置
        BasicConfigurator.configure();
        // 注意：这个logger是apache下的，前面玩的JUL中的是java。util.logging包下的
        Logger logger = Logger.getLogger(Log4jTest01.class);

        logger.fatal("fatal信息");
        logger.error("error信息");
        logger.warn("warn信息");
        logger.info("info信息");
        logger.debug("debug信息");
        logger.trace("trace信息");
    }
}
```

> 注意加载初始化信息：`BasicConfigurator.configure();`，不加这一句代码就报错（没有添加Appenders输出控制器），加上不报错是因为：源码中有这样一句代码`rootManager.addAppenders(XxxxAppender(PatternLayout layout))`

configure源码如下：

```java
public static void configure() {
    Logger root = Logger.getRootLogger();
    root.addAppender(new ConsoleAppender(new PatternLayout("%r [%t] %p %c %x - %m%n")));
}
```




> **LOG4J日志级别**

**Log4j提供了8个级别的日志输出，分别为如下级别：**

1. **ALL** 最低等级 用于打开所有级别的日志记录

2. **TRACE** 程序推进下的追踪信息，这个追踪信息的日志级别非常低，一般情况下是不会使用的

3. **DEBUG** 指出细粒度信息事件对调试应用程序是非常有帮助的，主要是配合开发，在开发过程中打印一些重要的运行信息，**在没有进行设置的情况下，默认的日志输出级别**

4. **INFO** 消息的粗粒度级别运行信息

5. **WARN** 表示警告，程序在运行过程中会出现的有可能会发生的隐形的错误

注意，有些信息不是错误，但是这个级别的输出目的就是为了给程序员以提示

6. **ERROR** 系统的错误信息，发生的错误不影响系统的运行

一般情况下，如果不想输出太多的日志，则使用该级别即可

7. **FATAL** 表示严重错误，它是那种一旦发生系统就不可能继续运行的严重错误

如果这种级别的错误出现了，表示程序可以停止运行了

8. **OFF** 最高等级的级别，用户关闭所有的日志记录



### [#](#分析源码) 分析源码

```java
BasicConfigurator.configure();

Logger logger = Logger.getLogger(Log4jTest01.class);

logger.fatal("fatal信息");
logger.error("error信息");
logger.warn("warn信息");
logger.info("info信息");
logger.debug("debug信息");
logger.trace("trace信息");
```

> 分析 `BasicConfigurator.configure();` 点击 `configure()`

得到的代码如下

```java
public static void configure() {
    Logger root = Logger.getRootLogger();
    root.addAppender(new ConsoleAppender(new PatternLayout("%r [%t] %p %c %x - %m%n")));
}
```

**从中可以得到几个信息：**

1. 创建根节点的对象`Logger root = Logger.getRootLogger();`

2. 根节点添加了`ConsoleAppender对象(表示默认打印到控制台，自定义的格式化输出PatternLayout)`



**那么想要自定义配置文件来实现上述源代码的功能呢？通过上面这个源代码分析，我们需要满足的条件是：**我们的配置文件需要提供Logger、Appender、Layout这3个组件信息





> 分析`Logger logger = Logger.getLogger(Log4jTest01.class);` 点击 `getLogger()`

```java
public static Logger getLogger(Class clazz) {
    return LogManager.getLogger(clazz.getName());
}
```

**发现：`LogManager.getLogger(clazz.getName());`**，其中：`LogManager`就是日志管理器




> 查看LogManager

首先看到如下信息，一堆常量

```java
public static final String DEFAULT_CONFIGURATION_FILE = "log4j.properties";
static final String DEFAULT_XML_CONFIGURATION_FILE = "log4j.xml";
/** @deprecated */
public static final String DEFAULT_CONFIGURATION_KEY = "log4j.configuration";
/** @deprecated */
public static final String CONFIGURATOR_CLASS_KEY = "log4j.configuratorClass";
/** @deprecated */
public static final String DEFAULT_INIT_OVERRIDE_KEY = "log4j.defaultInitOverride";
private static Object guard = null;
private static RepositorySelector repositorySelector;
```

**这些东西代表的就是不同形式（不同后缀名）的配置文件，其中`log4j.properties属性`是我们最常用的，因为它语法简单、使用方便**



> **log4j.properties的加载时机**
> 加载：那就是static
>
> 观察LogManager中的代码，找到其中的静态代码块static

发现如下的一堆源码

```java
static {
    Hierarchy h = new Hierarchy(new RootLogger(Level.DEBUG));
    repositorySelector = new DefaultRepositorySelector(h);
    String override = OptionConverter.getSystemProperty("log4j.defaultInitOverride", (String)null);
    if (override != null && !"false".equalsIgnoreCase(override)) {
        LogLog.debug("Default initialization of overridden by log4j.defaultInitOverrideproperty.");
    } else {
        String configurationOptionStr = OptionConverter.getSystemProperty("log4j.configuration", (String)null);
        String configuratorClassName = OptionConverter.getSystemProperty("log4j.configuratorClass", (String)null);
        URL url = null;
        if (configurationOptionStr == null) {
            url = Loader.getResource("log4j.xml");
            if (url == null) {
                // 前面就是文件格式的一堆判断，这里才是log4j.propertie格式做的事情
                url = Loader.getResource("log4j.properties");
            }
        } else {
            try {
                url = new URL(configurationOptionStr);
            } catch (MalformedURLException var7) {
                url = Loader.getResource(configurationOptionStr);
            }
        }

        if (url != null) {
            LogLog.debug("Using URL [" + url + "] for automatic log4j configuration.");

            try {
                // 这里又是一个信息：selectAndConfigure() 翻译就是选择配置文件
                OptionConverter.selectAndConfigure(url, configuratorClassName, getLoggerRepository());
            } catch (NoClassDefFoundError var6) {
                LogLog.warn("Error during default initialization", var6);
            }
        } else {
            LogLog.debug("Could not find resource: [" + configurationOptionStr + "].");
        }
    }
}
```

**从源码中，发现`url = Loader.getResource("log4j.properties");`**

从这句代码得到的信息：**系统默认是从当前的类路径下找到`log4j.properties`，而若是maven工程，那么就应该在resources路径下去找**



**同时在上面的源码中发现` OptionConverter.selectAndConfigure(url, configuratorClassName, getLoggerRepository());`**




> 查看`selectAndConfigure()`

发现如下源码

```java
public static void selectAndConfigure(URL url, String clazz, LoggerRepository hierarchy) {
    Configurator configurator = null;
    String filename = url.getFile();
    if (clazz == null && filename != null && filename.endsWith(".xml")) {
        clazz = "org.apache.log4j.xml.DOMConfigurator";
    }

    if (clazz != null) {
        LogLog.debug("Preferred configurator class: " + clazz);
        configurator = (Configurator)instantiateByClassName(clazz, Configurator.class, (Object)null);
        if (configurator == null) {
            LogLog.error("Could not instantiate configurator [" + clazz + "].");
            return;
        }
    } else {
        // 有用信息在这里，即 new PropertyConfigurator(); 创建了一个properties配置对象
        configurator = new PropertyConfigurator();
    }

    ((Configurator)configurator).doConfigure(url, hierarchy);
}
```





> 查看PropertyConfigurator类

首先看到的就是如下的常量信息

```java
static final String CATEGORY_PREFIX = "log4j.category.";
static final String LOGGER_PREFIX = "log4j.logger.";
static final String FACTORY_PREFIX = "log4j.factory";
static final String ADDITIVITY_PREFIX = "log4j.additivity.";
static final String ROOT_CATEGORY_PREFIX = "log4j.rootCategory";
// 这是一个重要信息
static final String ROOT_LOGGER_PREFIX = "log4j.rootLogger";
// 这也是一个重要信息
static final String APPENDER_PREFIX = "log4j.appender.";
static final String RENDERER_PREFIX = "log4j.renderer.";
static final String THRESHOLD_PREFIX = "log4j.threshold";
private static final String THROWABLE_RENDERER_PREFIX = "log4j.throwableRenderer";
private static final String LOGGER_REF = "logger-ref";
private static final String ROOT_REF = "root-ref";
private static final String APPENDER_REF_TAG = "appender-ref";
public static final String LOGGER_FACTORY_KEY = "log4j.loggerFactory";
private static final String RESET_KEY = "log4j.reset";
private static final String INTERNAL_ROOT_NAME = "root";
```

通过前面的基础，从这源码中，发现有两个信息是要进行配置的

```java
static final String ROOT_LOGGER_PREFIX = "log4j.rootLogger";
static final String APPENDER_PREFIX = "log4j.appender.";
```

那么这二者是怎么进行配置的？





> 找寻`static final String APPENDER_PREFIX = "log4j.appender."`中的appender配置方式：直接在当前源码页面搜索appender

发现如下的源代码

```java
Appender parseAppender(Properties props, String appenderName) {
    Appender appender = this.registryGet(appenderName);
    if (appender != null) {
        LogLog.debug("Appender \"" + appenderName + "\" was already parsed.");
        return appender;
    } else {
        // 重要信息就在这里，这里告知了一件事：上面找到的 log4j.appender. 配置方式为如下的方式
        String prefix = "log4j.appender." + appenderName;
        // 这也是重要信息，layout日志格式化的配置方式
        String layoutPrefix = prefix + ".layout";
        appender = (Appender)OptionConverter.instantiateByKey(props, prefix, Appender.class, (Object)null);
        if (appender == null) {
            LogLog.error("Could not instantiate appender named \"" + appenderName + "\".");
            return null;
        } else {
            appender.setName(appenderName);
            if (appender instanceof OptionHandler) {
                if (appender.requiresLayout()) {
                    Layout layout = (Layout)OptionConverter.instantiateByKey(props, layoutPrefix, Layout.class, (Object)null);
                    if (layout != null) {
                        appender.setLayout(layout);
                        LogLog.debug("Parsing layout options for \"" + appenderName + "\".");
                        PropertySetter.setProperties(layout, props, layoutPrefix + ".");
                        LogLog.debug("End of parsing for \"" + appenderName + "\".");
                    }
                }

                String errorHandlerPrefix = prefix + ".errorhandler";
                String errorHandlerClass = OptionConverter.findAndSubst(errorHandlerPrefix, props);
                if (errorHandlerClass != null) {
                    ErrorHandler eh = (ErrorHandler)OptionConverter.instantiateByKey(props, errorHandlerPrefix, ErrorHandler.class, (Object)null);
                    if (eh != null) {
                        appender.setErrorHandler(eh);
                        LogLog.debug("Parsing errorhandler options for \"" + appenderName + "\".");
                        this.parseErrorHandler(eh, errorHandlerPrefix, props, this.repository);
                        Properties edited = new Properties();
                        String[] keys = new String[]{errorHandlerPrefix + "." + "root-ref", errorHandlerPrefix + "." + "logger-ref", errorHandlerPrefix + "." + "appender-ref"};
                        Iterator iter = props.entrySet().iterator();

                        while(true) {
                            if (!iter.hasNext()) {
                                PropertySetter.setProperties(eh, edited, errorHandlerPrefix + ".");
                                LogLog.debug("End of errorhandler parsing for \"" + appenderName + "\".");
                                break;
                            }

                            Entry entry = (Entry)iter.next();

                            int i;
                            for(i = 0; i < keys.length && !keys[i].equals(entry.getKey()); ++i) {
                            }

                            if (i == keys.length) {
                                edited.put(entry.getKey(), entry.getValue());
                            }
                        }
                    }
                }

                PropertySetter.setProperties(appender, props, prefix + ".");
                LogLog.debug("Parsed \"" + appenderName + "\" options.");
            }

            this.parseAppenderFilters(props, appenderName, appender);
            this.registryPut(appender);
            return appender;
        }
    }
}
```

通过上述的源码，发现配置`log4j.appender.`的方式：`log4j.appender.+appenderName`
- 其中：appenderName就是输出控制器名字



继而：推导出`log4j.properties`配置文件中的一个配置项appender输出方式为：`log4j.appender.+appenderName=某一种输出控制器名字`

- 其中：输出控制器名字在前面一开始就接触过了



![image](https://img2023.cnblogs.com/blog/2421736/202304/2421736-20230411030125900-566089202.png)



**因此`Log4j.properties`的appender输出方式配置方式举例就是`log4j.appender.console=org.apache.log4j.ConsoleAppender`**



**同样道理，通过第二句代码` String layoutPrefix = prefix + ".layout";`，也就知道了layout输出格式的配置方式**

![image](https://img2023.cnblogs.com/blog/2421736/202304/2421736-20230411030125873-211822014.png)



layout日志输出格式配置举例：

```properties
log4j.appender.console.layout=org.log4j.SimpleLayout
```




> 小小总结一波：log4j.properties配置文件中的appender输出控制器 和 layout日志输出格式的配置方式

```properties
log4j.appender,console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.log4j.SimpleLayout
```



> 继续找第二个配置`static final String ROOT_LOGGER_PREFIX = "log4j.rootLogger"`中的rootLogger配置方式：通过log4j.rootLogge进行搜索

发现如下的方法

```java
void configureRootCategory(Properties props, LoggerRepository hierarchy) {
    String effectiveFrefix = "log4j.rootLogger";
    String value = OptionConverter.findAndSubst("log4j.rootLogger", props);
    if (value == null) {
        value = OptionConverter.findAndSubst("log4j.rootCategory", props);
        effectiveFrefix = "log4j.rootCategory";
    }

    if (value == null) {
        LogLog.debug("Could not find root logger information. Is this OK?");
    } else {
        Logger root = hierarchy.getRootLogger();
        synchronized(root) {
            // 这里面执行了这个方式
            this.parseCategory(props, root, effectiveFrefix, "root", value);
        }
    }

}
```



> 查看`parseCategory()`


在这里找到了想要的配置方式

```java
void parseCategory(Properties props, Logger logger, String optionKey, String loggerName, String value) {
    LogLog.debug("Parsing for [" + loggerName + "] with value=[" + value + "].");
    // 配置方式就在这里，这个操作的意思就是：表示要以逗号的方式来切割字符串
    // 		证明了log4j.rootLogger的取值，可以有多个值，使用逗号进行分隔
    StringTokenizer st = new StringTokenizer(value, ",");
    if (!value.startsWith(",") && !value.equals("")) {
        if (!st.hasMoreTokens()) {
            return;
        }

        // 把字符串通过逗号切割之后，第一个值的用途就在这里
        // 		levelStr、level，即：切割后的第一个值是日志的级别
        String levelStr = st.nextToken();
        LogLog.debug("Level token is [" + levelStr + "].");
        if (!"inherited".equalsIgnoreCase(levelStr) && !"null".equalsIgnoreCase(levelStr)) {
            logger.setLevel(OptionConverter.toLevel(levelStr, Level.DEBUG));
        } else if (loggerName.equals("root")) {
            LogLog.warn("The root logger cannot be set to null.");
        } else {
            logger.setLevel((Level)null);
        }

        LogLog.debug("Category " + loggerName + " set to " + logger.getLevel());
    }

    logger.removeAllAppenders();

    // 字符串切割之后的第一个值是level日志级别，而剩下的值的用途就在这里
    while(st.hasMoreTokens()) {
        // 通过这句代码得知：第2 - 第n个值，就是我们配置的其他信息，这个信息就是appenderName
        String appenderName = st.nextToken().trim();
        if (appenderName != null && !appenderName.equals(",")) {
            LogLog.debug("Parsing appender named \"" + appenderName + "\".");
            Appender appender = this.parseAppender(props, appenderName);
            if (appender != null) {
                logger.addAppender(appender);
            }
        }
    }

}
```

**通过上述的代码分析，得知`log4j.rootLogger`的配置方式为：**

```properties
log4j.rootLogger=日志级别,appenderName1,appenderName2,appenderName3....
# 表示可以同时在根节点上配置多个日志输出的途径
```




### [#](#最基本的log4j.properties配置) 最基本的log4j.properties配置

通过前面的源码分析之后，得出`BasicConfigurator.configure();`替代品的properties配置如下：

```properties
# rootLogger所有logger的根配置
# 	log4j.rootLogger=日志级别,appenderName1,appenderName2,appenderName3....
# 这里的例子没用日志输出路径，这个日志输出路径后续再加
log4j.rootLogger=debug,console

# appender输出控制器配置
# 	log4j.appender.+appenderName=某一种输出类型 - 采用Console控制台的方式举例
log4j.appender.console=org.apache.log4j.ConsoleAppender
# 输出的格式配置
# 	log4j.appender.+appenderName+layout=某种layout格式类型
log4j.appender.console.layout=org.apache.log4j.SimpleLayout
```

测试

```java
// 注掉这一句就可以了，这句代码的配置由上面的自定义配置进行代替了
// BasicConfigurator.configure();

Logger logger = Logger.getLogger(Log4jTest01.class);

logger.fatal("fatal信息");
logger.error("error信息");
logger.warn("warn信息");
logger.info("info信息");
logger.debug("debug信息");
logger.trace("trace信息");
```



### [#](#打开日志输出的详细信息) 打开日志输出的详细信息

```java
@Test
public void test03(){

    /*
        通过Logger中的开关
            打开日志输出的详细信息
            查看LogManager类中的方法getLoggerRepository()
            找到代码 LogLog.debug(msg, ex);
                LogLog会使用debug级别的输出为我们展现日志输出详细信息
                Logger是记录系统的日志，那么LogLog就是用来记录Logger的日志

            进入到 LogLog.debug(msg, ex); 方法中
            通过代码：if (debugEnabled && !quietMode) 
                观察到if判断中的这两个开关都必须开启才行
                !quietMode是已经启动的状态，不需要我们去管
                debugEnabled默认是关闭的

            所以我们只需要设置debugEnabled为true就可以了
     */
    // 开启 debugEnabled
    LogLog.setInternalDebugging(true);

    Logger logger = Logger.getLogger(Log4jTest01.class);

    logger.fatal("fatal信息");
    logger.error("error信息");
    logger.warn("warn信息");
    logger.info("info信息");
    logger.debug("debug信息");
    logger.trace("trace信息");
}
```

若未开启debugEnabled，那么输出信息就是如下的样子：如下是举的一个例子而已

```java
0 [main] FATAL cn.zixieqing.HotelJavaApplicationTests  - fatal信息
1 [main] ERROR cn.zixieqing.HotelJavaApplicationTests  - error信息
1 [main] WARN cn.zixieqing.HotelJavaApplicationTests  - warn信息
1 [main] INFO cn.zixieqing.HotelJavaApplicationTests  - info信息
1 [main] DEBUG cn.zixieqing.HotelJavaApplicationTests  - debug信息
```

开启之后，信息就会更全

```java
log4j: Trying to find [log4j.xml] using context classloader sun.misc.Launcher$AppClassLoader@18b4aac2.
log4j: Trying to find [log4j.xml] using sun.misc.Launcher$AppClassLoader@18b4aac2 class loader.
log4j: Trying to find [log4j.xml] using ClassLoader.getSystemResource().
log4j: Trying to find [log4j.properties] using context classloader sun.misc.Launcher$AppClassLoader@18b4aac2.
log4j: Trying to find [log4j.properties] using sun.misc.Launcher$AppClassLoader@18b4aac2 class loader.
log4j: Trying to find [log4j.properties] using ClassLoader.getSystemResource().
log4j: Could not find resource: [null].
0 [main] FATAL cn.zixieqing.HotelJavaApplicationTests  - fatal信息
0 [main] ERROR cn.zixieqing.HotelJavaApplicationTests  - error信息
1 [main] WARN cn.zixieqing.HotelJavaApplicationTests  - warn信息
1 [main] INFO cn.zixieqing.HotelJavaApplicationTests  - info信息
1 [main] DEBUG cn.zixieqing.HotelJavaApplicationTests  - debug信息
```





### [#](#自定义输出格式 patternLayout) 自定义输出格式 patternLayout

自定义配置的也就是`Layout`而已，而自定义就是玩的`PatternLayout`，这个类有一个`setConversionPattern()`方法，查看这个方法的源码

```java
public void setConversionPattern(String conversionPattern) {
    this.pattern = conversionPattern;
    this.head = this.createPatternParser(conversionPattern).parse();
}
```

从中发现，需要配置的就是`String conversionPattern`。因此：在`log4j.properties`配置文件中添加上`conversionPattern`属性配置即可，当然：这个属性配置遵循一定的写法，写法如下：

```txt
%m 输出代码中指定的日志信息
%p 输出优先级，及 DEBUG、INFO 等
%n 换行符（Windows平台的换行符为 "\n"，Unix 平台为 "\n"）
%r 输出自应用启动到输出该 log 信息耗费的毫秒数
%c 输出打印语句所属的类的全名
%t 输出产生该日志的线程全名
%d 输出服务器当前时间，默认为 ISO8601，也可以指定格式，如：%d{yyyy年MM月dd日 HH:mm:ss}
%l 输出日志时间发生的位置，包括类名、线程、及在代码中的行数。如：Test.main(Test.java:10)
%F 输出日志消息产生时所在的文件名称
%L 输出代码中的行号
%% 输出一个 "%" 字符
可以在 % 与字符之间加上修饰符来控制最小宽度、最大宽度和文本的对其方式
    [%10p]：[]中必须有10个字符，由空格来进行补齐，信息右对齐
    [%-10p]：[]中必须有10个字符，由空格来进行补齐，信息左对齐，应用较广泛



上述举例：[%-10p]%r %c%t%d{yyyy-MM-dd HH:mm:ss:SSS} %m%n
```

修改了`log4j.properties`的配置如下：做的修改就是最后两个配置项

```properties
# rootLogger所有logger的根配置
# 	log4j.rootLogger=日志级别,appenderName1,appenderName2,appenderName3....
log4j.rootLogger=debug,console

# appender输出控制器配置
# 	log4j.appender.+appenderName=某一种输出类型	采用Console控制台的方式举例
log4j.appender.console=org.apache.log4j.ConsoleAppender
# 输出的格式配置
# 	log4j.appender.+appenderName+layout=某种layout格式类型
# 	注意：这里类型改了，是PatternLayout，即自定义
log4j.appender.console.layout=org.apache.log4j.PatternLayout
# 编写自定义输出格式，直接把上面举例的拿过来		注意：加上了刚刚说的conversionPattern属性
log4j.appender.console.layout.conversionPattern=[%-10p]%r %c%t%d{yyyy-MM-dd HH:mm:ss:SSS} %m%n
```

测试代码

```java
public void test04(){

    LogLog.setInternalDebugging(true);
    Logger logger = Logger.getLogger(Log4jTest01.class);

    logger.fatal("fatal信息");
    logger.error("error信息");
    logger.warn("warn信息");
    logger.info("info信息");
    logger.debug("debug信息");
    logger.trace("trace信息");
}
```





### [#](#将日志输出到文件中) 将日志输出到文件中

#### [#](#改造log4j.properties文件) 改造log4j.properties文件

```properties
# rootLogger所有logger的根配置	这里再加一个file
log4j.rootLogger=debug,console,file

# 控制台的appender输出控制器配置
log4j.appender.console=org.apache.log4j.ConsoleAppender
# 采用自定义控制台输出的格式
log4j.appender.console.layout=org.apache.log4j.PatternLayout
# 编写控制台自定义输出格式
log4j.appender.console.layout.conversionPattern=[%-10p]%r %c%t%d{yyyy-MM-dd HH:mm:ss:SSS} %m%n


# 再来一份，变为file的配置	把console改为file即可
log4j.appender.file=org.apache.log4j.FileAppender
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.conversionPattern=[%-10p]%r %c%t%d{yyyy-MM-dd HH:mm:ss:SSS} %m%n
```





#### [#](#指定文件输出位置 及 字符编码设置) 指定文件输出位置 及 字符编码设置


查看`FileAppender`源码，首先看到的是四个属性

```java
// 表示日志文件是否采用内容追加的方式 - 源码中有一个构造方法，这个的默认值是true
protected boolean fileAppend;
protected String fileName;
protected boolean bufferedIO;
// 日志文件的大小 - 源码的构造方法中默认值是8192
protected int bufferSize;

// 构造方法源码

public FileAppender() {
    this.fileAppend = true;
    this.fileName = null;
    this.bufferedIO = false;
    this.bufferSize = 8192;
}
```

在`FlieAppender`中还有一个`setFile()`方法，得知这个就是设置文件的方法 ，也就是文件日志文件存放路径位置

```java
public void setFile(String file) {
    String val = file.trim();
    this.fileName = val;
}
```

这里面需要传一个file参数进去，因此：通过`MyBatis`的知识就知道`log4j.properties`配置中的这个对应属性就是file了（截掉set，得到属性名，这句话其实是我在鬼扯、吹牛批）

```properties
# rootLogger所有logger的根配置	这里再加一个file
log4j.rootLogger=debug,console,file
# 控制台的appender输出控制器配置
log4j.appender.console=org.apache.log4j.ConsoleAppender
# 采用自定义控制台输出的格式
log4j.appender.console.layout=org.apache.log4j.PatternLayout
# 编写控制台自定义输出格式
log4j.appender.console.layout.conversionPattern=[%-10p]%r %c%t%d{yyyy-MM-dd HH:mm:ss:SSS} %m%n


# 再来一份，变为file的配置	把console改为file即可
log4j.appender.file=org.apache.log4j.FileAppender
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.conversionPattern=[%-10p]%r %c%t%d{yyyy-MM-dd HH:mm:ss:SSS} %m%n
# 日志文件保存路径	注意：
# 	前一个file为自定义的appenderName；
# 	后一个file是日志文件输出路径的属性，要是怕看错眼，可以把后者换为File大写也没错
log4j.appender.file.file=D:\log4j\zixieqing\log4j.log
```

现在看`FileAppender`的父类`WriterAppender`，去看字符编码设置

```java
protected boolean immediateFlush;
// 这个属性就是编码设置
protected String encoding;
protected QuietWriter qw;
```

因此：加上日志文件输出路径和字符编码之后的`log4.properties`配置

```properties
# 控制台日志输出设置
log4j.rootLogger=debug,console,file
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.conversionPattern=[%-10p]%r %c%t%d{yyyy-MM-dd HH:mm:ss:SSS} %m%n


# 日志文件输出设置
log4j.appender.file=org.apache.log4j.FileAppender
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.conversionPattern=[%-10p]%r %c%t%d{yyyy-MM-dd HH:mm:ss:SSS} %m%n
log4j.appender.file.file=D:\log4j\zixieqing\log4j.log
log4j.appender.file.encoding=UTF-8
```




#### [#](#拆分日志文件) 拆分日志文件

> 一份日志不可能永远一直记录下去，那样的话，日志文件体积就太大了，毕竟都知道太大了遭不住嘛。@紫邪情

继续查看`FileAppender`源码，看实现类

![image](https://img2023.cnblogs.com/blog/2421736/202304/2421736-20230411030125971-602683827.png)



##### [#](#RollingFileAppender实现类) RollingFileAppender实现类

> 这个玩意儿就是利用文件大小来进行日志拆分

源码中有两个属性需要关注

```java
// 达到多大文件时进行日志拆分
protected long maxFileSize = 10485760L;
// 一共能够拆分出多少份日志文件
protected int maxBackupIndex = 1;
```

因此：现在将`log4j.properties`配置文件修改一下即可实现日志根据文件大小进行拆分

```properties
# 注意：记得在 log4j.rootLoger=debug,console,file 这一句中加上下面的这个文件大小进行拆分的配置
# 	如：log4j.rootLoger=debug,console,rollingFile
log4j.appender.rollingFile=org.apache.log4j.RollingFileAppender
log4j.appender.rollingFile.layout=org.apache.log4j.PatternLayout
log4j.appender.rollingFile.layout.conversionPattern=[%-10p]%r %c%t%d{yyyy-MM-dd HH:mm:ss:SSS} %m%n
log4j.appender.rollingFile.file=D:\log4j\zixieqing\log4j.log
# 文件多大时进行日志拆分
log4j.appender.rollingFile.maxFileSize=10MB
# 最多可以拆分多少份
log4j.appender.rollingFile.maxBackupIndex=50
```



##### [#](#DailyRollingFileAppender实现类) DailyRollingFileAppender实现类：建议用

> 这个东西就是根据时间来进行日志拆分

看源码，有这么一个属性

```java
// 时间格式，默认值就是如下的天
private String datePattern = "'.'yyyy-MM-dd";
```

在`log4j.properties`配置文件中修改成如下的配置即可使用

```properties
# 一样的，要使用这种方式：记得在 log4j.rootLogger=debug,console,file 这一句中加上下面的这个文件大小进行拆分的配置
# 	如：log4j.rootLogger=debug,console,dateRollingFile
log4j.appender.dateRollingFile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.dateRollingFile.layout=org.apache.log4j.PatternLayout
log4j.appender.dateRollingFile.layout.conversionPattern=[%-10p]%r %c%t%d{yyyy-MM-dd HH:mm:ss:SSS} %m%n
log4j.appender.dateRollingFile.file=D:\log4j\zixieqing\log4j.log
# 加上时间格式	下面的值根据实际情况即可
log4j.appender.dateRollingFile.datePattern='.'yyyy-MM-dd
```




### [#](#日志持久化到数据库) 日志持久化到数据库

建表SQL语句：瞎鸡巴写的

```sql
CREATE TABLE tbl_log(
    id int(11) NOT NULL AUTO_INCREMENT,
    name varchar(100) DEFAULT NULL COMMENT '项目名称',
    createTime varchar(100) DEFAULT NULL COMMENT '创建时间',
    level varchar(10) DEFAULT NULL COMMENT '日志级别',
    category varchar(100) DEFAULT NULL COMMENT '所在类的全路径',
    fileName varchar(100) DEFAULT NULL COMMENT '文件名称',
    message varchar(255) DEFAULT NULL COMMENT '日志消息',
    PRIMARY KEY(id)
)
```

依赖

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>${mysql.version}</version>
    <scope>runtime</scope>
</dependency>
```

log4.properties配置

```properties
# 配置appender输出方式 输出到数据库表
# 	注意：在rootManager中加上logDB，如：log4j.rootLogger=debug,console,logDB
log4j.appender.logDB=org.apache.log4j.jdbc.JDBCAppender
log4j.appender.logDB.layout=org.apache.log4j.PatternLayout
log4j.appender.logDB.Driver=com.mysql.jdbc.Driver
log4j.appender.logDB.URL=jdbc:mysql://localhost:3306/test
log4j.appender.logDB.User=root
log4j.appender.logDB.Password=072413
log4j.appender.logDB.Sql=INSERT INTO tbl_log(name,createTime,level,category,fileName,message) values('project_log','%d{yyyy-MM-dd HH:mm:ss}','%p','%c','%F','%m')
```

测试代码

```java
@Test
public void test07(){
    Logger logger = Logger.getLogger(Log4jTest01.class);
    logger.fatal("fatal信息");
    logger.error("error信息");
    logger.warn("warn信息");
    logger.info("info信息");
    logger.debug("debug信息");
    logger.trace("trace信息");
}
```





### [#](#自定义logger) 自定义logger


前面配置文件中使用的都是`rootManager`的logger，接下来就自定义一个logger，看`PropertyConfigurator`类的源码，它里面有一个属性

```java
// 自定义logger配置的写法	这后面拼接的就是自定义的logger名字
static final String LOGGER_PREFIX = "log4j.logger.";
```

**其中：上述说的自定义logger名字遵循父子关系，也就是包关系**

```txt
如：cn.zixieqing.log4j.test.Log4jTest01	它的父logger就是上层的路径或者是更上层的路径
    例如：
        cn.zixieqing.log4j.test
        cn.zixieqing.log4j
        ...
        cn
```

修改log4j.properties

```properties
# 根logger，输出级别是trace，在console控制台进行输出
log4j.rootLogger=trace,console
# 自定义logger，级别为info，在file文件中输出
log4j.logger.cn.zixieqing.log4j.test=info,file
# 自定义logger，是apache的，级别为error
log4j.logger.org.apache=error
```




> **自定义logger的注意点**
>
> 1. 如果根节点的logge（即：rootManager ）和 自定义logger配置的输出位置是不同的，则取二者的并集，配置的位置都会进行输出操作
>
> 2. 如果二者配置的日志级别不同，以我们自定义logger的输出级别为主




### [#](#一份简单的log4j.properties配置) 一份简单的log4j.properties配置

```properties
log4j.rootLogger=DEBUG,console,file

# 控制台输出的相关设置
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n

# 文件输出的相关设置
log4j.appender.file = org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./1og/zixieqing.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]‰m%n

# 日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sq1=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```



# [#](#JCL) JCL

> 全称为Jakarta Commons Logging，是Apache提供的一个通用日志API
>
> 注意：这个玩意儿本身没有记录日志的实现功能，而是相当于一个门面。我们可以自由选择第三方的日志组件（ log4j、JUL）作为具体实现，common-logging会通过动态查找的机制，在程序运行时自动找出真正使用的日志库

![image](https://img2023.cnblogs.com/blog/2421736/202304/2421736-20230411030126064-490523710.png)






**JCL的组成**：JCL 有两个基本的抽象类

1. **Log**：日志记录器
2. **LogFactory**：日志工厂（负责创建Log实例）



## [#](#玩JCL) 玩JCL

依赖

```xml
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.2</version>
</dependency>
```

测试

```java
@Test
void jclQuickStartTest() {

    // LogFactory是org.apache.commons.logging.LogFactory
    Log log = LogFactory.getLog(JCLTest01.class);
    log.info("info信息");
}
```

运行之后看效果会发现：输出格式是JUL格式。即：**如果没有任何第三方日志框架的时候，默认使用的就是JUL**





## [#](#引入log4j) 引入log4j

```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

然后使用`log4j.properties`配置文件：**当再次进行测试时就会变成log4j的输出配置**




### [#](#JCL源码分析) JCL源码分析

```java
Log接口的4个实现类
    JDk13
    JDK14	正常java.util.logging
    Log4j	我们集成的log4j
    Simple	JCL自带实现类


    （1）查看Jdk14Logger证明里面使用的是JUL日志框架		看import引入的就可以得知
    （2）查看Log4JLogger证明里面使用的是Log4j日志框架	看import引入的就可以得知

    （3）观察LogFactory，看看如何加载的Logger对象
	        这是一个抽象类，无法实例化	需要观察其实现类LogFactoryImpl

    （4）观察LogFactoryImpl
	        真正加载日志实现使用的就是这个实现类LogFactoryImpl

    （5）进入getLog		采用打断点debug能够更清晰地看清楚

			进入getInstance

			找到instance = this.newInstance(name);，继续进入

			找到instance = this.discoverLogImplementation(name); 表示发现一个日志的实现

                for(int i = 0; i < classesToDiscover.length && result == null; ++i) {
                    result = this.createLogFromClass(classesToDiscover[i], logCategory, true);
                }
			遍历我们拥有的日志实现框架
				遍历的是一个数组，这个数组是按照
                    log4j
                    jdk14
                    jdk13
                    SimpleLogger
                    的顺序依次遍历
                表示的是，第一个要遍历的就是log4j，如果有log4j则执行该日志框架
                如果没有，则遍历出来第二个，使用jdk14的JUL日志框架
                以此类推

            result = this.createLogFromClass(classesToDiscover[i], logCategory, true);
                表示帮我们创建Logger对象
                在这个方法中，我们看到了 c = Class.forName(logAdapterClassName, true, currentCL);
                    是取得该类型的反射类型对象

                使用反射的形式帮我们创建logger对象
                constructor = c.getConstructor(this.logConstructorSignature);
```



# [#](#SLF4J) SLF4J

> 这也是一个日志门面（门面模式 / 外观模式），其核心为：外部与一个子系统的通信必须通过一个统一的外观对象进行，使得子系统更易于使用



**常见的日志门面和日志实现**

- 常见的日志实现：JUL、log4j、logback、log4j2

- 常见的日志门面 ：JCL、slf4j



出现顺序 ：log4j -->JUL-->JCL--> slf4j --> logback --> log4j2



**了解SLF4J**

- 全称：Simple Logging Facade For Java，简单Java日志门面

- 主要是为了给Java日志访问提供一套标准、规范的API框架，其主要意义在于提供接口，具体的实现可以交由其他日志框架，例如log4j和logback等

- **SLF4J最重要的两个功能就是对于日志框架的绑定以及日志框架的桥接**




### [#](#玩SLF4J) 玩SLF4J

#### [#](#快速上手) 快速上手

> **准备知识**

**SLF4J对日志的级别划分**：

```json
trace、debug、info、warn、error五个级别

    trace：日志追踪信息
    debug：日志详细信息
    info：日志的关键信息 默认打印级别
    warn：日志警告信息
    error：日志错误信息
```

**在没有任何其他日志实现框架集成的基础之上，slf4j使用的就是自带的框架slf4j-simple**

**注意点：slf4j-simple也必须以单独依赖的形式导入进来**

```xml
<!--slf4j 自带的简单日志实现 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.25</version>
</dependency>
```



> **入门**

依赖

```xml
<!--slf4j 核心依赖-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>

<!--slf4j 自带的简单日志实现 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.25</version>
</dependency>
```

测试代码

```java
@Test
public void test01(){

    // 是slf4j包下的
    Logger logger = LoggerFactory.getLogger(SLF4JTest01.class);
    logger.trace("trace信息");
    logger.debug("debug信息");
    logger.info("info信息");
    logger.warn("warn信息");
    logger.error("error信息");
}
```



#### [#](#动态信息输出) 动态信息输出

> **本质**：使用占位符
>
> **有时候输出的日志信息，需要我们搭配动态的数据，这些数据有可能是信息，有可能是数据库表中的数据**

```java
@Test
public void test02(){

    Logger logger = LoggerFactory.getLogger(SLF4JTest01.class);
    String name = "zs";
    int age = 23;
    
    /* 
        字符串拼接的形式：logger.info("学生信息-姓名："+name+"；年龄："+age);
        使用JCL的形式：logger.info("学生信息-姓名：{}，年龄：{}",new Object[]{name,age});

        这上面两者虽然都可以做到相应的输出，但是：麻烦
    */
    // 使用SLF4J的形式
    logger.info("学生信息-姓名：{}，年龄：{}",name,age);
}
```

> **注意点**
>
> 如果后面拼接的字符串是一个对象，那么`{}`并不能充当占位，进行字符串拼接




#### [#](#输出异常信息) 输出异常信息

```java
@Test
public void test03(){

    /*
        日志对于异常信息的处理

            一般情况下，我们在开发中的异常信息，也会记录在控制台上
            我们会根据异常信息提取出有用的线索，来调试BUG

            但是在真实生产环境中（项目上线），对于服务器或者是系统相关的问题
            在控制台上虽然也会提供相应的异常或者错误信息的输出
            可但是这种错误输出方式（输出的时间，位置，格式...）都是服务器系统默认的

            我们可以通过日志技术，选择将异常以日志打印的方式，进行输出查看
            输出的时间，位置（控制台，文件），格式，完全由我们自己去进行定义
     */

    Logger logger = LoggerFactory.getLogger(SLF4JTest01.class);

    try {
        Class.forName("aaa");
    } catch (ClassNotFoundException e) {
        // e.printStackTrace();
        logger.info("XXX类中的XXX方法出现了异常，请及时关注信息");
        /* e是引用类型对象，不能跟前面的{}做有效的字符串拼接
			logger.info("具体错误是：{}",e);
			我们不用加{},直接后面加上异常对象e即可
				这是利用了重载方法info(String message, Throwable throwable)
        */
        logger.info("具体错误是：",e);
    }
}
```






#### [#](#SLF4J与日志绑定) SLF4J与日志绑定

> 官网中有一张图，官网地址：https://www.slf4j.org

![image](https://img2023.cnblogs.com/blog/2421736/202304/2421736-20230411030125963-1787098714.png)



图中分为了三部分

1、在没有绑定任何日志实现的基础之上，日志是不能够绑定实现任何功能的
- slf4j-simple是slf4j官方提供的
- 使用的时候，也是需要导入依赖，自动绑定到slf4j门面上
- 如果不导入，slf4j 核心依赖是不提供任何实现的

2、log4j和JUL
- 都是slf4j门面时间线前面的日志实现，所以API不遵循slf4j进行设计
- 通过**适配桥接**的技术，完成的与日志门面的衔接

3、logback和simple（包括nop）
- 都是slf4j门面时间线后面提供的日志实现，所以API完全遵循slf4j进行的设计
- 只需要导入想要使用的日志实现依赖，即可与slf4j无缝衔接
- 注意：nop虽然也划分到实现中了，但是它是指不实现日志记录




#### [#](#绑定logback) 绑定logback

依赖

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.11</version>
</dependency>
```

测试

```java
public class SLF4JTest {

    @Test
    public void bingingLogTest() {

        Logger logger = LoggerFactory.getLogger(SLF4JTest.class);

        try {
            Class.forName("aaa");
        } catch (ClassNotFoundException e) {
            logger.info("具体错误是：",e);
        }
    }
}
```

结果

```json
10:50:15.391 [main] INFO com.zixieqing.SLF4JTest - 具体错误是：
java.lang.ClassNotFoundException: aaa
    at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
    at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
    at java.lang.Class.forName0(Native Method)
    at java.lang.Class.forName(Class.java:264)
    .......
```

这种就看起来很清爽，如果加上`slf4j-simple`那输出结果又是另一回事，测试跳过
上面这种就是不用去管底层到底采用的是哪一种日志实现，可是上面的源代码完全没有改变，照常写，这就是日志门面的好处



#### [#](#slf4j-nop禁止日志打印) slf4j-nop禁止日志打印

依赖

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>
<!--
    注意：有坑儿，和依赖导入顺序有关
		如·：要是将这个依赖放到logback的下面，那么屁用都没有
		但是：把这个依赖放在logback的前面就可以实现日志禁止打印了

		原因：在slf4j环境下，要是同时出现了多个日志实现，默认使用先导入的日志实现
-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-nop</artifactId>
    <version>1.7.30</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.11</version>
</dependency>
```

测试

```java
@Test
public void slf4jNopTest() {

    Logger logger = LoggerFactory.getLogger(SLF4JTest.class);

    try {
        Class.forName("aaa");
    } catch (ClassNotFoundException e) {
        logger.info("具体错误是：",e);
    }
}
```

结果

```json
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/D:/install/maven/apache-maven-3.6.1/maven-repo/org/slf4j/slf4j-nop/1.7.30/slf4j-nop-1.7.30.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/D:/install/maven/apache-maven-3.6.1/maven-repo/ch/qos/logback/logback-classic/1.2.11/logback-classic-1.2.11.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.helpers.NOPLoggerFactory]
```

日志内容的相关打印就没了

> **注意**
>
> 如果想要让nop发挥效果，禁止所有日志的打印，那么就必须要将slf4j-nop的依赖放在所有日志实现依赖的上方




#### [#](#绑定log4j) 绑定log4j

> **玩这个就需要注意日志框架时间线的问题了**
>
> **出现顺序 ：log4j -->JUL-->JCL--> slf4j --> logback --> log4j2**

也就是在slf4j之后出现的（如：logback、log4j2），这些都遵循slf4j的规范，所以直接导入对应的依赖之后就可以使用

但是在slf4j之前的，并没有预料到会出现后续这些规范嘛，而slf4j想要绑定log4j就需要一个`slf4j-log4j12`的适配器




> **玩玩绑定log4j**

依赖

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
<!--slf4j和log4j的适配器 - 想要slf4j能够绑定log4j就需要这个-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.30</version>
</dependency>
```


测试

```java
@Test
public void slf4jBindingLog4jTest() {
    Logger logger = LoggerFactory.getLogger(SLF4JTest.class);
    logger.info("info信息");
}
```


结果

```json
# 虽然没有输出消息，但是有如下这个话就足够了No appenders could be found for logger
# 这表示没有appender嘛，加上log4j.properties配置文件就可以使用了
log4j:WARN No appenders could be found for logger (com.zixieqing.SLF4JTest).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
```




> **其他日志框架的绑定就看官网的那张图，在那张图中对绑定谁时导入哪一个依赖都有备注**

![image](https://img2023.cnblogs.com/blog/2421736/202304/2421736-20230411030125902-1337051079.png)






#### [#](#slf4j源码分析流程) slf4j源码分析流程

```json
进入到getLogger
            看到Logger logger = getLogger(clazz.getName());

            进入重载的getLogger
                ILoggerFactory iLoggerFactory = getILoggerFactory(); 用来取得Logger工厂实现的方法

            进入getILoggerFactory()
                看到以双重检查锁的方式去做判断
                执行performInitialization(); 工厂的初始化方法

            进入performInitialization()
                bind()就是用来绑定具体日志实现的方法

            进入bind()
                看到Set集合 Set<URL> staticLoggerBinderPathSet = null;
                因为当前有可能会有N多个日志框架的实现
                看到staticLoggerBinderPathSet = findPossibleStaticLoggerBinderPathSet();

            进入findPossibleStaticLoggerBinderPathSet()
                看到创建了一个有序不可重复的集合对象
                    LinkedHashSet staticLoggerBinderPathSet = new LinkedHashSet();
                声明了枚举类的路径，经过if else判断，以获取系统中都有哪些日志实现
                看到Enumeration paths;
                if (loggerFactoryClassLoader == null) {
                    paths = ClassLoader.getSystemResources(STATIC_LOGGER_BINDER_PATH);
                } else {
                    paths = loggerFactoryClassLoader.getResources(STATIC_LOGGER_BINDER_PATH);
                }

            我们主要观察常量STATIC_LOGGER_BINDER_PATH
                通过常量我们会找到类StaticLoggerBinder
                这个类是以静态的方式绑定Logger实现的类
                来自slf4j-JDK14的适配器

            进入StaticLoggerBinder
                看到new JDK14LoggerFactory();
                进入JDK14LoggerFactory类的无参构造方法
                看到java.util.logging.Logger.getLogger("");
                使用的就是jul的Logger

            接着观察findPossibleStaticLoggerBinderPathSet
                看到以下代码，表示如果还有其他的日志实现
                while(paths.hasMoreElements()) {
                    URL path = (URL)paths.nextElement();
                    将路径添加进入
                    staticLoggerBinderPathSet.add(path);
                }

            回到bind方法
                表示对于绑定多实现的处理
                reportMultipleBindingAmbiguity(staticLoggerBinderPathSet);
                如果出现多日志实现的情况
                则会打印
                Util.report("Class path contains multiple SLF4J bindings.");
```




> **通过源码总结：**
>
> 在真实生产环境中，slf4j只绑定一个日志实现框架就可以了。绑定多个，默认使用导入依赖的第一个，而且会产生没有必要的警告信息。@紫邪情



#### [#](#slf4j日志重构) slf4j日志重构

> 有这么一个情况：项目原本使用的是log4j日志，但是随着技术的迭代，需要使用另外的日志框架，如：slf4j+logback，因此：此时不可能说去改源码，把所有使用log4j的地方都改成slf4j。@紫邪情

这种情况，在slf4j官网中就提供了对应的解决方式，就是加个依赖而已，加的东西就叫做桥接器

![image](https://img2023.cnblogs.com/blog/2421736/202304/2421736-20230411030126306-891039216.png)





##### [#](#玩一下桥接器) 玩一下桥接器

> 演示bug：先保证项目没有其他任何的干扰，如：另外的依赖、另外日志的代码

依赖

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```


log4j.properties配置文件

```properties
log4j.rootLogger=debug,console
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.lo4j.SimpleLayout
```

测试代码

```java
import org.apache.log4j.LogManager;
import org.apache.log4j.Logger;
import org.junit.Test;



    @Test
    public void bridgingTest() {

        // 假设项目用的是log4j
        Logger logger = LogManager.getLogger(SLF4JTest.class);

        logger.info("inf信息");
    }
```


**此时，项目升级，改用slf4j+logback来当日志**

1. 去掉log4j的依赖

2. 添加slf4j的桥接组件

所以现在的依赖就变成如下的样子

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>
<!--这个就是log4j和slf4j的桥接组件-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>log4j-over-slf4j</artifactId>
    <version>1.7.25</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.11</version>
</dependency>
```

要引入什么桥接组件，看要怎么重构，然后根据官网的说明来弄就行

![image](https://img2023.cnblogs.com/blog/2421736/202304/2421736-20230411030126306-891039216.png)




测试：源代码直接不用动

````java
import org.apache.log4j.LogManager;
import org.apache.log4j.Logger;
import org.junit.Test;


    @Test
    public void bridgingTest() {

        // 假设项目用的是log4j
        Logger logger = LogManager.getLogger(SLF4JTest.class);

        logger.info("inf信息");
    }
````


结果

```json
14:45:09.901 [main] INFO com.zixieqing.SLF4JTest - inf信息
```





# [#](#LOGBACK) LOGBACK

> 这是log4j的作者跳槽之后开发的另一款日志框架，比log4j出色很多

**Logback当前分成三个模块：logback-core、logback- classic、logback-access**

- logback-core是其它两个模块的基础模块
- logback-classic是log4j的一个改良版本。此外logback-classic完整实现SLF4J API。可以很方便地更换成其它日志系统如log4j或JDK14 Logging
- logback-access访问模块与[Servlet](https://baike.baidu.com/item/Servlet/477555)容器集成提供通过Http来访问日志的功能



**logback的组成**

1. **Logger**:  日志的记录器，主要用于存放日志对象，也可以定义日志类型、级别

2. **Appender**：用于指定日志输出的目的地，目的地可以是控制台、文件、数据库等等

3. **Layout**:  负责把事件转换成字符串，格式化的日志信息的输出

> **注意点**
>
> 在Logback中Layout对象被封装在encoder中，即：未来使用的encoder其实就是Layout




**logback配置文件的类型**：Logback提供了3种配置文件

- logback.groovy
- logback-test.xml
- logback.xml



我们用的一般是xml的格式



**logback的日志输出格式**

```json
日志输出格式：
    %-10level  级别 案例为设置10个字符，左对齐
    %d{yyyy-MM-dd HH:mm:ss.SSS} 日期
    %c  当前类全限定名
    %M  当前执行日志的方法
    %L  行号
    %thread 线程名称
    %m或者%msg    信息
    %n  换行
```




## [#](#玩logback) 玩logback

### [#](#logback的日志级别) logback的日志级别

```txt
error > warn > info > debug > trace

其中：logback的默认日志级别就是debug
```



### [#](#快速上手) 快速上手

依赖

```xml
<!-- logback-classic 中包含了基础模板 logback-core-->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.11</version>
</dependency>
<!--logback是日志实现，也需要通过日志门面来集成-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
```

测试

```java
public class LogBackTest {

    @Test
    public void quickStartTest() {

        // 这个log4j中的
        Logger logger = LoggerFactory.getLogger(LogBackTest.class);
        logger.error("error信息");
        logger.warn("warn信息");
        logger.info("info信息");
        logger.debug("debug信息");
        logger.trace("trace信息");
    }
}
```


结果

```json
15:44:27.032 [main] ERROR com.zixieqing.LogBackTest - error信息
15:44:27.033 [main] WARN com.zixieqing.LogBackTest - warn信息
15:44:27.033 [main] INFO com.zixieqing.LogBackTest - info信息
15:44:27.033 [main] DEBUG com.zixieqing.LogBackTest - debug信息
```



> 得出信息，**logback的默认日志级别就是debug**



### [#](#简单认识logback的配置文件) 简单认识logback的配置文件

```xml
<?xml version="1.0" encoding="utf-8" ?>
<!--logback的所有配置都需要在 <configuration></configuration> 标签中进行-->
<configuration>

    <!--通用属性配置	目的：在下面配置需要的地方使用 ${name} 的形式，方便的取得value值
            name	起的名字 见名知意即可
            value	值
    -->
    <!--配置日志输出格式-->
    <property name="pattern" value="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} %c %M %L %thread %m%n"/>

    <!--配置日志输出类型
            name	乱取，见名知意即可
            class	类型全类路径
    -->
    <appender name = "consoleAppender" class = "ch.qos.logback.core.ConsoleAppender">
        <!--配置日志的颜色-->
        <target>
            <!--还可以用system.out就是常见的黑白色-->
            system.err
        </target>

        <!--配置日志输出格式，直接引用前面配置的property通用属性-->
        <encoder class = "ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${pattern}</pattern>
        </encoder>
    </appender>

    <!--配置root Logger-->
    <root level="DEBUG">
        <appender-ref ref="consoleAppender"/>
    </root>

</configuration>
```

`<property name="" value = "" ` 中 `vlaue`的参数配置参考

```txt
日志输出格式：
    %-10level	级别 案例为设置10个字符，左对齐
    %d{yyyy-MM-dd HH:mm:ss.SSS}		日期
    %c		当前类全限定名
    %M		当前执行日志的方法
    %L		行号
    %thread			线程名称
    %m 或者 %msg     信息
    %n  	换行
```

测试

```java
public class LogBackTest {

    @Test
    public void quickStartTest() {

        // 这个log4j中的
        Logger logger = LoggerFactory.getLogger(LogBackTest.class);
        logger.error("error信息");
        logger.warn("warn信息");
        logger.info("info信息");
        logger.debug("debug信息");
        logger.trace("trace信息");
    }
}
```



### [#](#将日志输出到文件中) 将日志输出到文件中

在`logback.xml`中加入如下的配置即可

```xml
<!--配置全局的文件输出路径-->
<property name="filePath" value="D:\test"/>

<!--定义文件的appender输出类型-->
<appender name="fileAppender" class="ch.qos.logback.core.FileAppender">
    <!--文件路径 和 文件名 -->
    <file>${filePath}/logback.log</file>

    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <!--文件输出的格式-->
        <pattern>${pattern}</pattern>
    </encoder>
</appender>

<!--记得在root logger中引用一下fileAppender的配置-->
<!--配置root Logger-->
<root level="DEBUG">
    <appender-ref ref="consoleAppender"/>
    <!--引用配置的文件appender输出配置-->
    <appender-ref ref="fileAppender"/>
</root>
```

测试代码

```java
@Test
public void fileAppenderTest() {
    // 这个log4j中的
    Logger logger = LoggerFactory.getLogger(LogBackTest.class);
    logger.error("error信息");
    logger.warn("warn信息");
    logger.info("info信息");
    logger.debug("debug信息");
    logger.trace("trace信息");
}
```

> 通过结果观察会发现：**logback的日志文件输出默认是以 <span style="color:purple">追加</span> 的形式添加新日志内容**




### [#](#以HTML格式记录日志文件) 以HTML格式记录日志文件

> 当日志文件不是很多的时候，可以采用这种方式，因为这种HTML里面的样式和格式都可以让logback进行生成，而里面的内容是我们加进去的而已，因此：这样一看就知道，HTML文件占用的内存容量就蛮大了

`logback.xml`配置文件编写

```xml
<!--定义HTML文件输出格式，本质还是文件，所以class是FileAppender-->
<appender name="HTMLFileAppender" class="ch.qos.logback.core.FileAppender">
    <file>${filePath}\logback.log</file>

    <!--注意：这里采用的是LayoutWrappingEncoder-->
    <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
        <!--注意：采用layout标签包裹，格式类型为HTMLLayout-->
        <layout class="ch.qos.logback.classic.html.HTMLLayout">
            <pattern>${pattern}</pattern>
        </layout>
    </encoder>
</appender>

<!--配置root Logger-->
<root level="DEBUG">
    <!--<appender-ref ref="consoleAppender"/>-->
    <!--<appender-ref ref="fileAppender"/>-->
    <appender-ref ref="HTMLFileAppender"/>
</root>
```

测试

```java
@Test
public void htmlFileAppenderTest() {
    // 这个log4j中的
    Logger logger = LoggerFactory.getLogger(LogBackTest.class);
    logger.error("error信息");
    logger.warn("warn信息");
    logger.info("info信息");
    logger.debug("debug信息");
    logger.trace("trace信息");
}
```

运行程序即可生成HTML文件



### [#](#拆分日志和归档压缩) 拆分日志 和 归档压缩

```xml
<!-- 配置文件的appender 可拆分归档的文件 -->
<appender name="roll" class="ch.qos.logback.core.rolling.RollingFileAppender">

    <!-- 输入格式 -->
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <pattern>${pattern}</pattern>
    </encoder>
    <!-- 引入文件位置 -->
    <file>${logDir}/roll_logback.log</file>

    <!-- 指定拆分规则 -->
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">

        <!-- 按照时间和压缩格式声明文件名 压缩格式gz -->
        <fileNamePattern>${logDir}/roll.%d{yyyy-MM-dd}.log%i.gz</fileNamePattern>

        <!-- 按照文件大小来进行拆分 -->
        <maxFileSize>1KB</maxFileSize>
    </rollingPolicy>
</appender>

<!--配置root Logger-->
<root level="DEBUG">
    <!--<appender-ref ref="consoleAppender"/>-->
    <!--<appender-ref ref="fileAppender"/>-->
    <!--<appender-ref ref="HTMLFileAppender"/>-->
    <appender-ref ref="roll"/>
</root>
```

> **注意点**
>
> 只要使用到拆分日志的事情，那么`<fileNamePattern></fileNamePattern>`标签必须有，源码中有说明

```java
// 在TimeBasedRollingPolicy类中有一个常量
static final String FNP_NOT_SET ="The FileNamePattern option must be set before using TimeBasedRollingPolicy. ";

// 这个常量值告知的结果就是上面的注意点
```

另外：其他有哪些属性，而每一个class对应下有哪些标签可以用，那就是class指定值中的属性名，也就是标签名，看源码即可


```java
@Test
public void rollFileAppenderTest() {
    for (int i = 0; i < 1000; i++) {
        // 这个log4j中的
        Logger logger = LoggerFactory.getLogger(LogBackTest.class);
        logger.error("error信息");
        logger.warn("warn信息");
        logger.info("info信息");
        logger.debug("debug信息");
        logger.trace("trace信息");
    }
}
```





### [#](#过滤器) 过滤器

```xml
<!-- 配置控制台的appender 使用过滤器 -->
<appender name="consoleFilterAppender" class="ch.qos.logback.core.ConsoleAppender">

    <target>
        System.out
    </target>

    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <pattern>${pattern}</pattern>
    </encoder>

    <!-- 配置过滤器 -->
    <filter class="ch.qos.logback.classic.filter.LevelFilter">

        <!-- 设置日志的输出级别 -->
        <level>ERROR</level>

        <!-- 高于level中设置的级别，则打印日志 -->
        <onMatch>ACCEPT</onMatch>

        <!-- 低于level中设置的级别，则屏蔽日志 -->
        <onMismatch>DENY</onMismatch>

    </filter>

</appender>


<!--配置root Logger-->
<root level="DEBUG">
    <!--<appender-ref ref="consoleAppender"/>-->
    <!--<appender-ref ref="fileAppender"/>-->
    <!--<appender-ref ref="HTMLFileAppender"/>-->
    <!--<appender-ref ref="roll"/>-->
    <!--解开console日志打印logger-->
    <appender-ref ref="consoleAppender"/>
</root>
```

测试代码照常用，然后在console控制台输出的内容会根据上述配置的filter过滤器罢对应的内容过滤掉，从而不打印出来



### [#](#异步日志) 异步日志

```xml
<!-- 1、配置异步日志 -->
<appender name="asyncAppender" class="ch.qos.logback.classic.AsyncAppender">
     <!-- 引用appender配置，即：让什么输出类型的日志进行异步操作，可以选择consoleAppender、fileAppender.... -->
    <appender-ref ref="fileAppender"/>
</appender>

<!--配置root Logger-->
<root level="DEBUG">
    <!--<appender-ref ref="consoleAppender"/>-->
    <!--<appender-ref ref="fileAppender"/>-->
    <!--<appender-ref ref="HTMLFileAppender"/>-->
    <!--<appender-ref ref="roll"/>-->
    <!--解开console日志打印logger-->
    <!--<appender-ref ref="consoleAppender"/>-->
    
    <!--2、在root logger中引用一下配置的异步日志-->
    <appender-ref ref="asyncAppender"/>
</root>
```

另外：在上面的`1、配置异步日志`里面还有两个配置属性

```xml
<!--下面这个配置的是一个阈值
		当队列的剩余容量小于这个阈值的时候，当前日志的级别 trace、debug、info这3个级别的日志将被丢弃
		设置为0，说明永远都不会丢弃trace、debug、info这3个级别的日志
-->
<discardingThreshold>0</discardingThreshold>


<!--配置队列的深度，这个值会影响记录日志的性能，默认值就是256-->
<queueSize>256</queueSize>
```

> **提示**
>
> 关于这两个属性，一般情况下，使用默认值即可
>
> 这两个属性不要乱配置，会影响系统性能，了解其功能即可


```java
@Test
public void asyncAppenderTest(){

    Logger logger = LoggerFactory.getLogger(LOGBACKTest01.class);

    // 日志打印操作
    for (int i = 0; i < 100; i++) {

        logger.error("error信息");
        logger.warn("warn信息");
        logger.info("info信息");
        logger.debug("debug信息");
        logger.trace("trace信息");

    }

    // 系统本身业务相关的其他操作
    System.out.println("系统本身业务相关的操作1");
    System.out.println("系统本身业务相关的操作2");
    System.out.println("系统本身业务相关的操作3");
    System.out.println("系统本身业务相关的操作4");
}
```





### [#](#自定义logger) 自定义logger

还是老样子，自定义logger就是为了替换下面这一堆

```xml
<!--配置root Logger-->
<root level="DEBUG">
    <!--<appender-ref ref="consoleAppender"/>-->
    <!--<appender-ref ref="fileAppender"/>-->
    <!--<appender-ref ref="HTMLFileAppender"/>-->
    <!--<appender-ref ref="roll"/>-->
    <!--解开console日志打印logger-->
    <!--<appender-ref ref="consoleAppender"/>-->
    
    <!--2、在root logger中引用一下配置的异步日志-->
    <appender-ref ref="asyncAppender"/>
</root>
```

自定义logger配置

```xml
<!-- additivity="false" 表示不继承rootlogger -->
<logger name="com.bjpowernode" level="info" additivity="false">
    <!-- 在自定义logger中配置appender -->
    <appender-ref ref="consoleAppender"/>
</logger>
```

测试代码和前面一样，跳过




# [#](#LOG4J2) LOG4J2

> 这玩意儿虽然叫log4j2，也就是对log4j做了增强，但是更多的其实是对logback不足做了优化。@紫邪情



**log4j2的特征**：

1. **性能提升**

Log4j2包含基于LMAX Disruptor库的下一代异步记录器。在多线程场景中，异步记录器的吞吐量比Log4j 1.x和Logback高18倍，延迟低

2. **自动重新加载配置**

与Logback一样，Log4j2可以在修改时自动重新加载其配置。与Logback不同，它会在重新配置发生时不会丢失日志事件

3. **高级过滤**

与Logback一样，Log4j2支持基于Log事件中的上下文数据，标记，正则表达式和其他组件进行过滤

此外，过滤器还可以与记录器关联。与Logback不同，Log4j2可以在任何这些情况下使用通用的Filter类

4. **插件架构**

Log4j使用插件模式配置组件。因此，无需编写代码来创建和配置Appender，Layout，Pattern Converter等。在配置了的情况下，Log4j自动识别插件并使用它们

5. **无垃圾机制**

在稳态日志记录期间，Log4j2 在独立应用程序中是无垃圾的，在Web应用程序中是低垃圾。这减少了垃圾收集器的压力，并且可以提供更好的响应性能




> **log4j2的最佳搭配**：采用`log4j2 + slf4j`的方式。@紫邪情



## [#](#玩log4j2) 玩log4j2

### [#](#快速上手) 快速上手

依赖

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.17.1</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.17.1</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
```

测试

```java
public class Log4j2Test {

    @Test
    public void quickStartTest() {

        // 是org.apache.logging.log4j包下的
        Logger logger = LogManager.getLogger(Log4j2Test.class);
        logger.fatal("fatal信息");
        logger.error("error信息");
        logger.warn("warn信息");
        logger.info("info信息");
    }
}
```

结果

```json
21:18:43.909 [main] FATAL com.zixieqing.Log4j2Test - fatal信息
21:18:43.911 [main] ERROR com.zixieqing.Log4j2Test - error信息
```



> 得出结论：**log4j2的默认级别是error级别**



### [#](#简单了解log4j2的配置文件) 简单了解log4j2的配置文件

> log4j2是参考logback创作出来的，所以配置文件也是使用xml
>
> log4j2同样是默认加载类路径（resources）下的log4j2.xml文件中的配置
>
> 但是：log4j2.xml和logback.xml有区别
>
> 1. 第一点就是log4j2.xml中的标签名字是<span style="color:purple">首字母大写</span>；
> 2. 第二点就是多个单词采用的是驼峰命名法，还有其他的区别

**简单的log4j2.xml配置文件**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--一样的，所有的日志配置都需要在 <Configuration></Configuration> 标签中进行

    还可以跟两个属性配置
        status="级别"			日志框架本身的日志输出级别	如 <Configuration status="DEBUG"/>
							一般都不需要配置，因为加了之后，输出信息会多一些其实没多大用的内容

        monitorInterval="数值" 自动加载配置文件的间隔时间 如：monitorInterval="5" 就是5秒

    注意：这个标签不要加xmlns属性，加上之后就有表空间约束，指不定会搞出问题
-->
<Configuration>
    <!--配置Appender输出类型-->
    <Appenders>
        <Console name="consoleAppender" target="SYSTEM_OUT"/>
    </Appenders>

    <!--配置logger-->
    <Loggers>
        <!--配置root logger-->
        <Root level="DEBUG">
            <AppenderRef ref="consoleAppender"/>
        </Root>
    </Loggers>
</Configuration>
```

测试代码

```java
@Test
public void log4j2XmlTest() {

    // 是org.apache.logging.log4j包下的
    Logger logger = LogManager.getLogger(Log4j2Test.class);
    logger.fatal("fatal信息");
    logger.error("error信息");
    logger.warn("warn信息");
    logger.info("info信息");
}
```

结果

```txt
直接输出如下内容：

    fatal信息
    error信息
    warn信息
    info信息
```



### [#](#SLF4J + LOG4J2) SLF4J + LOG4J2：从此开始都是重点

> slf4j门面调用的是log4j2的门面，再由log4j2的门面调用log4j2的实现


**步骤**

1. 导入slf4j的日志门面
2. 导入log4j2的适配器
3. 导入log4j2的日志门面
4. 导入log4j2的日志实现
5. 编写log4j2.xml



依赖

```xml
<!--slf4j门面-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>
<!--log4j-slf4j适配器-->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.17.1</version>
</dependency>
<!--log4j门面-->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.17.1</version>
</dependency>
<!--log4j日志实现-->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.17.1</version>
</dependency>
```

log4j2.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--一样的，所有的日志配置都需要在 <Configuration></Configuration> 标签中进行

    还可以跟两个属性配置
        status="级别"			日志框架本身的日志输出级别	如 <Configuration status="DEBUG"/>
							一般都不需要配置，因为加了之后，输出信息会多一些其实没多大用的内容

        monitorInterval="数值" 自动加载配置文件的间隔时间 如：monitorInterval="5" 就是5秒

    注意：这个标签不要加xmlns属性，加上之后就有表空间约束，指不定会搞出问题
-->
<Configuration>
    <!--配置Appender输出类型-->
    <Appenders>
        <Console name="consoleAppender" target="SYSTEM_OUT"/>
    </Appenders>

    <!--配置logger-->
    <Loggers>
        <!--配置root logger-->
        <Root level="DEBUG">
            <AppenderRef ref="consoleAppender"/>
        </Root>
    </Loggers>
</Configuration>
```

测试：保证没有其他的测试代码 或者说 保证`import`导入是`org.slf4j`门面的

```java
@Test
public void log4j2AndSlf4jTest() {
    Logger logger = LoggerFactory.getLogger(Log4j2Test.class);
    logger.error("error信息");
    logger.warn("warn信息");
    logger.info("info信息");
    logger.debug("debug信息");
    logger.trace("trace信息");
}
```

结果

```sjon
error信息
warn信息
info信息
debug信息
```



### [#](#将日志输出到文件) 将日志输出到文件

> **提示**
>
> 目前的代码都是基于前面配置好的log4j2 + slf4j的形式

log4j2.xml的配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--一样的，所有的日志配置都需要在 <Configuration></Configuration> 标签中进行

    还可以跟两个属性配置
        status="级别"			日志框架本身的日志输出级别	如 <Configuration status="DEBUG"/>
							一般都不需要配置，因为加了之后，输出信息会多一些其实没多大用的内容

        monitorInterval="数值" 自动加载配置文件的间隔时间 如：monitorInterval="5" 就是5秒

    注意：这个标签不要加xmlns属性，加上之后就有表空间约束，指不定会搞出问题
-->
<Configuration>

    <!--配置全局通用属性-->
    <properties>
        <property name="logPath">D:\test</property>
    </properties>

    <!--配置Appender输出类型-->
    <Appenders>
        <Console name="consoleAppender" target="SYSTEM_OUT"/>

        <!--配置文件输出-->
        <File name="fileAppender" fileName="${logPath}/log4j2.log">
            <!-- 配置文件输出格式 一样遵循logback中的格式 -->
            <PatternLayout pattern="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} %m%n"/>
        </File>
    </Appenders>

    <!--配置logger-->
    <Loggers>
        <!--配置root logger-->
        <Root level="DEBUG">
            <AppenderRef ref="consoleAppender"/>
            <AppenderRef ref="fileAppender"/>
        </Root>
    </Loggers>
</Configuration>
```

测试代码


```java
@Test
public void fileAppenderTest() {
    Logger logger = LoggerFactory.getLogger(Log4j2Test.class);
    logger.error("error信息");
    logger.warn("warn信息");
    logger.info("info信息");
    logger.debug("debug信息");
    logger.trace("trace信息");
}
```



### [#](#拆分日志) 拆分日志

log4j2.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--一样的，所有的日志配置都需要在 <Configuration><C/onfiguration> 标签中进行

	这个标签中除了xmlns，还可以跟两个属性配置
		status="级别"		日志框架本身的日志输出级别
						 如：<Configuration xmlns="http://logging.apache.org/log4j/2.0/config" status="DEBUG"/>
						一般都不需要配置，因为加了之后，输出信息会多一些其实没多大用的内容

        monitorInterval="数值"	自动加载配置文件的间隔时间 如：monitorInterval="5" 就是5秒
-->
<Configuration>

    <!--配置全局通用属性-->
    <properties>
        <property name="logPath">D:\test</property>
    </properties>

    <!--配置Appender输出类型-->
    <Appenders>
        <Console name="consoleAppender" target="SYSTEM_OUT"/>

        <!--配置文件输出-->
        <File name="fileAppender" fileName="${logPath}/log4j2.log">
            <!-- 配置文件输出格式 一样遵循logback中的格式 -->
            <PatternLayout pattern="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} %m%n"/>
        </File>

        <!--按照指定规则来拆分日志文件

           fileName		日志文件的名字
           filePattern	日志文件拆分后文件的命名规则
						$${date:yyyy-MM-dd}		根据日期当天，创建一个文件夹
                             例如：2021-01-01这个文件夹中，记录当天的所有日志信息（拆分出来的日志放在这个文件夹中）
                                  2021-01-02这个文件夹中，记录当天的所有日志信息（拆分出来的日志放在这个文件夹中）
						rollog-%d{yyyy-MM-dd-HH-mm}-%i.log
                             为文件命名的规则：%i表示序号，从0开始，目的是为了让每一份文件名字不会重复
       -->
        <RollingFile name="rollingFile" 
                     fileName="${logPath}/rollingLog.log"
                     filePattern="${logPath}/$${date:yyyy-MM-dd}/rollingLog-%d{yyyy-MM-dd-HH-mm}-%i.log">

            <!-- 日志消息格式 -->
            <PatternLayout pattern="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} %m%n"/>

            <Policies>
                <!-- 在系统启动时，触发拆分规则，产生一个日志文件 -->
                <OnStartupTriggeringPolicy/>

                <!-- 按照文件的大小进行拆分
				    注：这里虽然限制是10KB，但是真实生成出来的文件一般都是比这数值大1KB
			    -->
                <SizeBasedTriggeringPolicy size="10KB"/>

                <!-- 按照时间节点进行拆分 拆分规则就是filePattern-->
                <TimeBasedTriggeringPolicy/>
            </Policies>

            <!-- 在同一目录下，文件的个数限制，如果超出了设置的数值，则根据时间以新的覆盖旧-->
            <DefaultRolloverStrategy max="30"/>
        </RollingFile>
    </Appenders>

    <!--配置logger-->
    <Loggers>
        <!--配置root logger-->
        <Root level="DEBUG">
            <AppenderRef ref="consoleAppender"/>
            <AppenderRef ref="fileAppender"/>
            <AppenderRef ref="rollingFile"/>
        </Root>
    </Loggers>
</Configuration>
```

测试

```java
@Test
public void rollingLogTest() {
    Logger logger = LoggerFactory.getLogger(Log4j2Test.class);

    for (int i = 0; i < 1000; i++) {
        logger.error("error信息");
        logger.warn("warn信息");
        logger.info("info信息");
        logger.debug("debug信息");
        logger.trace("trace信息");
    }
}
```






### [#](#异步日志) 异步日志

> **这个技术就是log4j2最骚的一个点**
>
> Log4j2提供了两种实现日志的方式，一个是通过**AsyncAppender**，一个是通过**AsyncLogger**，分别对应前面我们说的Appender组件和Logger组件
>
> 注意：这是两种不同的实现方式，在设计和源码上都是不同的体现





#### [#](#AsyncAppender方式) AsyncAppender方式：了解

> 是通过引用别的Appender来实现的：这种方式使用的就是`<Async></Async>`标签

当有日志事件到达时，会开启另外一个线程来处理它们

需要注意的是：如果在Appender的时候出现异常，对应用来说是无法感知的

AsyncAppender应该在它引用的Appender之后配置，默认使用 `java.util.concurrent.ArrayBlockingQueue` 实现而不需要其它外部的类库。当使用此Appender的时候，在多线程的环境下需要注意：阻塞队列容易受到锁争用的影响，这可能会对性能产生影响。这时候，我们应该考虑使用无锁的异步记录器（AsyncLogger）




> **实现步骤**

1、添加异步日志依赖

```xml
<!--asyncAppender异步日志依赖-->
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.4.2</version>
</dependency>
```

2、在Appenders标签中，对于异步进行配置：使用Async标签

```xml
<!-- 配置异步日志 -->
<Async name="asyncAppender">
  <!-- 引用在前面配置的appender -->
  <AppenderRef ref="fileAppender"/>
</Async>
```

3、rootlogger引用Async

```xml
<!--配置logger-->
<Loggers>
    <!--配置root logger-->
    <Root level="DEBUG">
        <AppenderRef ref="asyncAppender"/>
    </Root>
</Loggers>
```




> **完整的log4j2.xml配置**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!-- 一样的，所有的日志配置都需要在 <Configuration><C/onfiguration> 标签中进行

	这个标签中除了xmlns，还可以跟两个属性配置
		status="级别"		日志框架本身的日志输出级别
						 如：<Configuration xmlns="http://logging.apache.org/log4j/2.0/config" status="DEBUG"/>
						 一般都不需要配置，因为加了之后，输出信息会多一些其实没多大用的内容

        monitorInterval="数值"	自动加载配置文件的间隔时间 如：monitorInterval="5" 就是5秒
-->
<Configuration>

    <!--配置全局通用属性-->
    <properties>
        <property name="logPath">D:\test</property>
    </properties>

    <!--配置Appender输出类型-->
    <Appenders>
        <Console name="consoleAppender" target="SYSTEM_OUT"/>

        <!--配置文件输出-->
        <File name="fileAppender" fileName="${logPath}/log4j2.log">
            <!-- 配置文件输出格式 一样遵循logback中的格式 -->
            <PatternLayout pattern="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} %m%n"/>
        </File>

        <!--按照指定规则来拆分日志文件

           fileName		日志文件的名字
           filePattern	日志文件拆分后文件的命名规则
						$${date:yyyy-MM-dd}		根据日期当天，创建一个文件夹
                             例如：2021-01-01这个文件夹中，记录当天的所有日志信息（拆分出来的日志放在这个文件夹中）
                                  2021-01-02这个文件夹中，记录当天的所有日志信息（拆分出来的日志放在这个文件夹中）
						rollog-%d{yyyy-MM-dd-HH-mm}-%i.log
                             为文件命名的规则：%i表示序号，从0开始，目的是为了让每一份文件名字不会重复
       -->
        <RollingFile name="rollingFile"
                     fileName="${logPath}/rollingLog.log"
                     filePattern="${logPath}/$${date:yyyy-MM-dd}/rollingLog-%d{yyyy-MM-dd-HH-mm}-%i.log">

            <!-- 日志消息格式 -->
            <PatternLayout pattern="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} %m%n"/>

            <Policies>
                <!-- 在系统启动时，触发拆分规则，产生一个日志文件 -->
                <OnStartupTriggeringPolicy/>

                <!-- 按照文件的大小进行拆分
				    注：这里虽然限制是10KB，但是真实生成出来的文件一般都是比这数值大1KB
			   -->
                <SizeBasedTriggeringPolicy size="10KB"/>

                <!-- 按照时间节点进行拆分 拆分规则就是filePattern-->
                <TimeBasedTriggeringPolicy/>
            </Policies>

            <!-- 在同一目录下，文件的个数限制，如果超出了设置的数值，则根据时间以新的覆盖旧-->
            <DefaultRolloverStrategy max="30"/>
        </RollingFile>

        <!-- 配置异步日志 -->
        <Async name="asyncAppender">
            <!-- 引用在前面配置的appender -->
            <AppenderRef ref="fileAppender"/>
        </Async>
    </Appenders>

    <!--配置logger-->
    <Loggers>
        <!--配置root logger-->
        <Root level="DEBUG">
            <AppenderRef ref="consoleAppender"/>
            <AppenderRef ref="fileAppender"/>
            <AppenderRef ref="rollingFile"/>
            <AppenderRef ref="asyncAppender"/>
        </Root>
    </Loggers>
</Configuration>
```

测试

```java
@Test
public void asyncAppenderTest() {
    Logger logger = LoggerFactory.getLogger(Log4j2Test.class);

    for (int i = 0; i < 1000; i++) {
        logger.error("error信息");
        logger.warn("warn信息");
        logger.info("info信息");
        logger.debug("debug信息");
        logger.trace("trace信息");
    }

    System.out.println("其他要执行的异步任务1");
    System.out.println("其他要执行的异步任务2");
    System.out.println("其他要执行的异步任务3");
    System.out.println("其他要执行的异步任务4");
    System.out.println("其他要执行的异步任务5");
}
```




#### [#](#AsyncLogger方式) AsyncLogger方式 ：必须会

> AsyncLogger才是log4j2实现异步最重要的功能体现，也是官方推荐的异步方式
>
> 它可以使得调用Logger.log返回的更快。你可以有两种选择：全局异步和混合异步



##### [#](#全局异步) 全局异步：了解

> 所有的日志都异步的记录，在配置文件上不用做任何改动，只需要在jvm启动的时候增加一个参数即可实现
>
> 
>
> **设置方式**：只需要**在类路径resources下添加一个properties属性文件**，做一步配置即可，**文件名要求必须是：`log4j2.component.properties`**

这个properties配置文件内容如下：

```properties
Log4jContextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector
```

**操作一波**

1. 在`resources`目录中新建`log4j2.component.properties`文件，并配置如下内容

```properties
Log4jContextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector
```

2. `log4j2.xml`文件配置：不配置任何的async异步设置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!-- 一样的，所有的日志配置都需要在 <Configuration><C/onfiguration> 标签中进行

	这个标签中除了xmlns，还可以跟两个属性配置
		status="级别"		日志框架本身的日志输出级别
						 如：<Configuration xmlns="http://logging.apache.org/log4j/2.0/config" status="DEBUG"/>
						 一般都不需要配置，因为加了之后，输出信息会多一些其实没多大用的内容

        monitorInterval="数值"	自动加载配置文件的间隔时间 如：monitorInterval="5" 就是5秒
-->
<Configuration>

    <!--配置全局通用属性-->
    <properties>
        <property name="logPath">D:\test</property>
    </properties>

    <!--配置Appender输出类型-->
    <Appenders>
        <Console name="consoleAppender" target="SYSTEM_OUT"/>

        <!--配置文件输出-->
        <File name="fileAppender" fileName="${logPath}/log4j2.log">
            <!-- 配置文件输出格式 一样遵循logback中的格式 -->
            <PatternLayout pattern="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} %m%n"/>
        </File>

        <!--按照指定规则来拆分日志文件

           fileName		日志文件的名字
           filePattern	日志文件拆分后文件的命名规则
						$${date:yyyy-MM-dd}		根据日期当天，创建一个文件夹
                             例如：2021-01-01这个文件夹中，记录当天的所有日志信息（拆分出来的日志放在这个文件夹中）
                                  2021-01-02这个文件夹中，记录当天的所有日志信息（拆分出来的日志放在这个文件夹中）
						rollog-%d{yyyy-MM-dd-HH-mm}-%i.log
                             为文件命名的规则：%i表示序号，从0开始，目的是为了让每一份文件名字不会重复
       -->
        <RollingFile name="rollingFile"
                     fileName="${logPath}/rollingLog.log"
                     filePattern="${logPath}/$${date:yyyy-MM-dd}/rollingLog-%d{yyyy-MM-dd-HH-mm}-%i.log">

            <!-- 日志消息格式 -->
            <PatternLayout pattern="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} %m%n"/>

            <Policies>
                <!-- 在系统启动时，触发拆分规则，产生一个日志文件 -->
                <OnStartupTriggeringPolicy/>

                <!-- 按照文件的大小进行拆分
					注：这里虽然限制是10KB，但是真实生成出来的文件一般都是比这数值大1KB
			   -->
                <SizeBasedTriggeringPolicy size="10KB"/>

                <!-- 按照时间节点进行拆分 拆分规则就是filePattern-->
                <TimeBasedTriggeringPolicy/>
            </Policies>

            <!-- 在同一目录下，文件的个数限制，如果超出了设置的数值，则根据时间以新的覆盖旧-->
            <DefaultRolloverStrategy max="30"/>
        </RollingFile>

        <!-- 配置异步日志 -->
        <!--<Async name="asyncAppender">-->
        <!--    &lt;!&ndash; 引用在前面配置的appender &ndash;&gt;-->
        <!--    <AppenderRef ref="fileAppender"/>-->
        <!--</Async>-->
    </Appenders>

    <!--配置logger-->
    <Loggers>
        <!--配置root logger-->
        <Root level="DEBUG">
            <AppenderRef ref="consoleAppender"/>
            <AppenderRef ref="fileAppender"/>
            <AppenderRef ref="rollingFile"/>
            <!--<AppenderRef ref="asyncAppender"/>-->
        </Root>
    </Loggers>
</Configuration>
```


测试

```java
@Test
public void globalAsync() {
    Logger logger = LoggerFactory.getLogger(Log4j2Test.class);

    for (int i = 0; i < 1000; i++) {
        logger.error("error信息");
        logger.warn("warn信息");
        logger.info("info信息");
        logger.debug("debug信息");
        logger.trace("trace信息");
    }

    System.out.println("其他要执行的异步任务1");
    System.out.println("其他要执行的异步任务2");
    System.out.println("其他要执行的异步任务3");
    System.out.println("其他要执行的异步任务4");
    System.out.println("其他要执行的异步任务5");
}
```





##### [#](#混合异步) 混合异步：必须会

> 可以在应用中同时使用同步日志和异步日志，这使得日志的配置方式更加灵活
>
> 混合异步的方式需要通过修改配置文件来实现，使用`AsyncLogger`标签来配置

log4j2.xml配置

> **注意点**
>
> 记得把全局异步的properties内容注释掉，全局异步和混合异步不能同时出现

下面配置的需求是：`cn.zixieqing`包下的日志输出到文件进行异步操作，而`root logger`日志是同步操作

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!-- 一样的，所有的日志配置都需要在 <Configuration><C/onfiguration> 标签中进行

	这个标签中除了xmlns，还可以跟两个属性配置
		status="级别"		日志框架本身的日志输出级别
						 如：<Configuration xmlns="http://logging.apache.org/log4j/2.0/config" status="DEBUG"/>
						 一般都不需要配置，因为加了之后，输出信息会多一些其实没多大用的内容

        monitorInterval="数值"	自动加载配置文件的间隔时间 如：monitorInterval="5" 就是5秒
-->
<Configuration>

    <!--配置全局通用属性-->
    <properties>
        <property name="logPath">D:\test</property>
    </properties>

    <!--配置Appender输出类型-->
    <Appenders>
        <Console name="consoleAppender" target="SYSTEM_OUT"/>

        <!--配置文件输出-->
        <File name="fileAppender" fileName="${logPath}/log4j2.log">
            <!-- 配置文件输出格式 一样遵循logback中的格式 -->
            <PatternLayout pattern="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} %m%n"/>
        </File>

        <!--按照指定规则来拆分日志文件

           fileName		日志文件的名字
           filePattern	日志文件拆分后文件的命名规则
						$${date:yyyy-MM-dd}		根据日期当天，创建一个文件夹
                             例如：2021-01-01这个文件夹中，记录当天的所有日志信息（拆分出来的日志放在这个文件夹中）
                                  2021-01-02这个文件夹中，记录当天的所有日志信息（拆分出来的日志放在这个文件夹中）
						rollog-%d{yyyy-MM-dd-HH-mm}-%i.log
                             为文件命名的规则：%i表示序号，从0开始，目的是为了让每一份文件名字不会重复
       -->
        <RollingFile name="rollingFile"
                     fileName="${logPath}/rollingLog.log"
                     filePattern="${logPath}/$${date:yyyy-MM-dd}/rollingLog-%d{yyyy-MM-dd-HH-mm}-%i.log">

            <!-- 日志消息格式 -->
            <PatternLayout pattern="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} %m%n"/>

            <Policies>
                <!-- 在系统启动时，触发拆分规则，产生一个日志文件 -->
                <OnStartupTriggeringPolicy/>

                <!-- 按照文件的大小进行拆分
					注：这里虽然限制是10KB，但是真实生成出来的文件一般都是比这数值大1KB
				-->
                <SizeBasedTriggeringPolicy size="10KB"/>

                <!-- 按照时间节点进行拆分 拆分规则就是filePattern-->
                <TimeBasedTriggeringPolicy/>
            </Policies>

            <!-- 在同一目录下，文件的个数限制，如果超出了设置的数值，则根据时间以新的覆盖旧-->
            <DefaultRolloverStrategy max="30"/>
        </RollingFile>

        <!-- 配置异步日志 -->
        <!--<Async name="asyncAppender">-->
        <!--    &lt;!&ndash; 引用在前面配置的appender &ndash;&gt;-->
        <!--    <AppenderRef ref="fileAppender"/>-->
        <!--</Async>-->

        <!-- 自定义logger，让自定义的logger为异步logger -->
        <!--
            name	就是包路径
					注意点：这个异步配置只限于这个name指定的包下的日志有异步操作，要是另有一个包cn.xiegongzi
					那么：cn.xiegongzi包下的日志还是同步的，并不会异步

            includeLocation="false"		表示去除日志记录中的行号信息，这个行号信息非常的影响日志记录的效率（生产中都不加这个行号）
									 严重的时候可能记录的比同步的日志效率还有低

            additivity="false"			表示不继承rootlogger

        -->
        <AsyncLogger name="cn.zixieqing"
                     level="debug"
                     includeLocation="false"
                     additivity="false">

            <!-- 文件输出fileAppender，设置为异步打印 -->
            <AppenderRef ref="fileAppender"/>

        </AsyncLogger>
    </Appenders>

    <!--配置logger-->
    <!--现在这个root logger中的日志是同步的-->
    <Loggers>
        <!--配置root logger-->
        <Root level="DEBUG">
            <AppenderRef ref="consoleAppender"/>
            <AppenderRef ref="fileAppender"/>
            <AppenderRef ref="rollingFile"/>
            <!--<AppenderRef ref="asyncAppender"/>-->
        </Root>
    </Loggers>
</Configuration>
```

测试

```java
@Test
public void blendAsync() {
    Logger logger = LoggerFactory.getLogger(Log4j2Test.class);

    for (int i = 0; i < 1000; i++) {
        logger.error("error信息");
        logger.warn("warn信息");
        logger.info("info信息");
        logger.debug("debug信息");
        logger.trace("trace信息");
    }

    System.out.println("其他要执行的异步任务1");
    System.out.println("其他要执行的异步任务2");
    System.out.println("其他要执行的异步任务3");
    System.out.println("其他要执行的异步任务4");
    System.out.println("其他要执行的异步任务5");
}
```



##### [#](#AsyncAppender、AsyncLogge两种方式的建议) AsyncAppender、AsyncLogge两种方式的建议

> 如果使用异步日志，AsyncAppender、AsyncLogger不要同时出现，不会有这种奇葩需求，同时效果也不会叠加，如果同时出现，那么效率会以AsyncAppender为主
>
> 
>
> 同样的，AsyncLogger的全局异步和混合异步也不要同时出现，也不会有这种奇葩，效果也不会叠加



# [#](#SpringBoot相关) SpringBoot相关

> SpringBoot 默认使用SLF4J作为日志门面，Logback作为日志实现来记录日志



项目初始依赖，只需要一个web启动器就可以了，我的SpringBoot版本`2.3.12.RELEASE`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

在这里面有这log日志相关的依赖

![image](https://img2023.cnblogs.com/blog/2421736/202304/2421736-20230411030126979-880647920.png)




## [#](#快速上手) 快速上手

测试

```java
@Test
void quickStartSpringBootLog() {
    // 是org.slf4j包下的
    Logger logger = LoggerFactory.getLogger(SpringbootLogApplicationTests.class);
    logger.error("error信息");
    logger.warn("warn信息");
    logger.info("info信息");
    logger.debug("debug信息");
    logger.trace("trace信息");
}
```

结果

```json
ERROR 8208 ---[           main] c.z.SpringbootLogApplicationTests        : error信息
WARN 8208 --- [           main] c.z.SpringbootLogApplicationTests        : warn信息
INFO 8208 --- [           main] c.z.SpringbootLogApplicationTests        : info信息
```



> **从这个输出内容得到的结果：**
>
> 1. 这种输出格式，就是典型的logback输出格式，即：SpringBoot的默认日志实现是logback
> 2. 日志默认级别是info级别




## [#](#桥接器) 桥接器

看一下SpringBoot中的日志桥接器是什么

```java
@Test
void bridgeTest() {
    // 是org.apache.logging.log4j包下的
    Logger logger = LogManager.getLogger(SpringbootLogApplicationTests.class);
    logger.info("info信息");
}
```


结果

```json
INFO 20948 --- [           main] c.z.SpringbootLogApplicationTests        : info信息
```



> **从这个效果就可以看出来：**
>
> 1. SpringBoot中桥接器是好使的
> 2. 使用的桥接器就是基础篇中的slf4j+logback的形式




## [#](#使用application.yml进行日志配置) 使用application.yml进行日志配置

### [#](#认识YAML配置日志的方式) 认识YAML配置日志的方式

```yaml
spring:
  application:
    name: SpringBootLog-server
# 日志配置
logging:
  # 日志级别 此示例为com.zixieqing包下的日志级别
  level:
    com:
      zixieqing: debug
  # 格式设置
  pattern:
    # 控制台输出格式设置
    console: "%d{yyyy-MM-dd} [%level] -%m%n"
```

测试

```java
@Test
void ymlConfigLogTest() {
    // 是org.slf4j包下的
    Logger logger = LoggerFactory.getLogger(SpringbootLogApplicationTests.class);
    logger.error("error信息");
    logger.warn("warn信息");
    logger.info("info信息");
    logger.debug("debug信息");
    logger.trace("trace信息");
}
```

结果

```json
2022-05-24 [ERROR] -error信息
2022-05-24 [WARN] -warn信息
2022-05-24 [INFO] -info信息
2022-05-24 [DEBUG] -debug信息
```




### [#](#YAML配置日志输出到文件中) YAML配置日志输出到文件中

```yml
spring:
  application:
    name: SpringBootLog-server
# 日志配置
logging:
  # 日志级别 此示例为com.zixieqing包下的日志级别
  level:
    com:
      zixieqing: debug
  # 格式设置
  pattern:
    # 控制台输出格式设置
    console: "%d{yyyy-MM-dd} [%level] -%m%n"
  # 将日志输出到文件中
  file:
    path: "d:/test/SpringBootLog"
```

> **注意**
>
> YAML配置文件中的`path: "d:/test/SpringBootLog"`只是路径，而默认的文件名叫做`spring.log`


测试

```java
@Test
void fileLog() {
    // 是org.slf4j包下的
    org.slf4j.Logger logger = LoggerFactory.getLogger(SpringbootLogApplicationTests.class);
    logger.error("error信息");
    logger.warn("warn信息");
    logger.info("info信息");
    logger.debug("debug信息");
    logger.trace("trace信息");
}
```



## [#](#使用logback实现日志拆分) 使用logback实现日志拆分

> 如果是需要配置日志拆分等相对高级的功能，那么`application.yml`就达不到需求了，所以：需要使用日志实现相应的配置文件。@紫邪情

这里的示例就是`logback`日志实现，和基础篇中一样，需要在类路径`resources`下，配置`logback.xml`



logback.xml测试配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <property name="pattern" value="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} %c %M %L %thread %m%n"/>
    <property name="logPath" value="d:/test" />

    <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
        <target>System.out</target>

        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${pattern}</pattern>
        </encoder>
    </appender>

    <!-- 配置文件的appender 可拆分归档的文件 -->
    <appender name="rollingLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 输入格式 -->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${pattern}</pattern>
        </encoder>
        <!-- 引入文件位置 -->
        <file>${logPath}/roll_logback.log</file>

        <!-- 指定拆分规则 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">

            <!-- 按照时间和压缩格式声明文件名 压缩格式gz -->
            <fileNamePattern>${logPath}/roll.%d{yyyy-MM-dd}.log%i.gz</fileNamePattern>

            <!-- 按照文件大小来进行拆分 -->
            <maxFileSize>1KB</maxFileSize>
        </rollingPolicy>
    </appender>

    <logger name="com.zixieqing" level="debug" additivity="false">
        <appender-ref ref="rollingLog"/>
    </logger>

</configuration>
```
> 想要配置其他的，基础篇中怎么配置，搬过来即可

测试

```java
@Test
void rollingLOg() {
    // 是org.slf4j包下的
    org.slf4j.Logger logger = LoggerFactory.getLogger(SpringbootLogApplicationTests.class);
    for (int i = 0; i < 1000; i++) {
        logger.error("error信息");
        logger.warn("warn信息");
        logger.info("info信息");
        logger.debug("debug信息");
        logger.trace("trace信息");
    }
}
```



## [#](#使用slf4j + log4j2) 使用slf4j + log4j2：重点

这两个搭配才干活不累




> **步骤**

1、因为SpringBoot默认是采用的slf4j + logback的形式，因此：去除web启动中的logging，从而就去除logback日志实现了

为什么剔除logging就可以了，看maven的依赖树，logback的上一级就是logging

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <!-- 排除掉原始依赖 以此去除logback引用 -->
        <exclusion>
            <artifactId>spring-boot-starter-logging</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>
```

2、导入SpringBoot和log4j2的整合启动器

```xml
<!-- 添加log4j2依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

3、在`resources`目录下新建`log4j2.xml`配置文件，并编写想要的内容即可

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--status			    框架本身的日志级别
	monitorInterval		指定热更新间隔 最小值5秒	我IDEA不生效，可能是我版本太低（IDEA 2020.3 老古董）
-->
<Configuration status="warn" monitorInterval="5">

    <!--此文件中的全局属性配置-->
    <properties>
        <property name="LOG_PATH" value="d:/test/SpringBootLog"/>
    </properties>

    <Appenders>
        <Console name="consoleAppender" target="SYSTEM_OUT">
            <PatternLayout pattern="[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] %l %c{36} - %m%n"/>
        </Console>
    </Appenders>

    <Appenders>
        <!--拆分日志-->
        <RollingFile name="rollingFile"
                     fileName="${LOG_PATH}/app.log"
                     filePattern="${LOG_PATH}/$${date:yyyy-MM}/app.%d{dd-HH-mm} %i.log">

            <!--拦截器 如果等级为debug的放行 否则就禁止 可以单独禁止某一项-->
            <ThresholdFilter level="debug" onMatch="ACCEPT" onMismatch="DENY"/>

            <!--自定义日志输出格式-->
            <PatternLayout pattern="[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] %l %c{36} - %m%n"/>

            <!--拆分策略-->
            <Policies>
                <!--系统启动时 触发拆分规则 产生一个新的日志文件-->
                <OnStartupTriggeringPolicy/>

                <!--按照时间节点拆分 就是filePattern定义的-->
                <TimeBasedTriggeringPolicy/>

                <!--单文件最大100m-->
                <SizeBasedTriggeringPolicy size="1MB"/>
            </Policies>

            <!--统一目录下 最多生成10个文件-->
            <DefaultRolloverStrategy max="10"/>
        </RollingFile>
    </Appenders>

    <Loggers>
        <Root level="info">
            <AppenderRef ref="consoleAppender"/>
            <AppenderRef ref="rollingFile"/>
        </Root>
    </Loggers>
</Configuration>
```

> 想要其他的配置，就把基础篇中对应的内容搬过来即可

**注意点：这个`log4j2.xml`文件中配置了如下内容的话**

```xml
<properties>
    <property name="LOG_PATH" value="d:/test/SpringBootLog"/>
</properties>
```

那么别傻不拉几滴在yml中又配置如下内容（或者要都配置的话，那么保证两个路径一样），不然报N多`ERROR / cause by: not found xxxxxxx `具体报的是什么忘了，也懒得重现事故了，总之貌似是`Failed to load log4j2 configurationerror detected `，具体英文是什么忘了，大概意思就是`检测到log4j2.xml加载失败`

```yml
# 日志配置
logging:
  file:
    path: "d:/test/SpringBootLog"
```

测试

```java
@Test
void slf4jAndLog4j2Test() {
    // 是org.slf4j包下的
    org.slf4j.Logger logger = LoggerFactory.getLogger(SpringbootLogApplicationTests.class);
    logger.error("error信息");
    logger.warn("warn信息");
    logger.info("info信息");
    logger.debug("debug信息");
    logger.trace("trace信息");
}
```


结果

![image](https://img2023.cnblogs.com/blog/2421736/202304/2421736-20230411030125875-272637278.png)







# Spring Boot 自定义注解：AOP Log

> 既然上面都整到了Spring Boot了，而本内容又是日志，那么就顺便弄个开发中的日志小知识呗，谁让我P话多勒。@紫邪情
>
> 
>
> 场景：记录日志。在方法执行前 / 后 / 环绕，将一些记录插入数据库
>
> 使用Spring AOP做增强：前置增强、后置增强、环绕增强

1. 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>3.0.3</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.47</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.6</version>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.8.20</version>
</dependency>
```

1. YAML配置

```yaml
server:
  port: 10010

# 项目名称配置 用于日志记录   也可以直接使用 spring.application.name
project:
  name: spring-boot3-AOP-log

spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/aop_log?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: "072413"

```

2. 实体类

```java
package com.zixq.entity;

import lombok.Data;
import lombok.experimental.Accessors;

import java.time.LocalDateTime;

/**
 * <p>
 * 日志实体类
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

@Data
@Accessors(chain = true)
public class LogEntity implements java.io.Serializable {
    
    /**
     * 主键
     */
    private Long id;
    /**
     * 模块名
     */
    private String module;
    /**
     * 用户名
     */
    private String username;
    /**
     * 操作类型：增删改查
     */
    private String operation;
    /**
     * 业务执行的耗时时长
     */
    private Long time;
    /**
     * 请求的ip地址
     */
    private String ip;
    /**
     * 请求方式（Get、Post....）、类名、方法名
     */
    private String method;
    /**
     * 参数：参数名 参数值
     */
    private String params;
    /**
     * 添加时间
     */
    private LocalDateTime createTime;
}
```

4. 自定义注解

```java
package com.zixq.annotation;

import com.zixq.CustomEnum.OperatorType;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * <p>
 * 自定义注解    用于方法执行前后进行日志记录
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {

    /**
     * 用户名
     */
    String username() default "系统";

    /**
     * 操作类型     增删改查、其他
     */
    OperatorType operation() default OperatorType.OTHER;
}
```

OperatorType枚举类：

```java
package com.zixq.CustomEnum;

/**
 * <p>
 * 操作类型枚举
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */
public enum OperatorType {
    ADD,		// 增
    REMOVE,		// 删
    UPDATE,		// 改
    GET,        // 查
    OTHER       // 其他
    ;
}
```



5. 增强逻辑：使用环绕增强做示例

```java
package com.zixq.aspect;

import cn.hutool.core.util.IdUtil;
import com.zixq.annotation.Log;
import com.zixq.entity.LogEntity;
import com.zixq.mapper.LogMapper;
import com.zixq.util.HttpContextUtils;
import com.zixq.util.IpUtils;
import jakarta.servlet.http.HttpServletRequest;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

/**
 * <p>
 * Log注解的逻辑处理
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

@Component
@Aspect
public class LogAspect {
    @Autowired
    private LogMapper logMapper;

    @Value("${project.name}")
    private String projectName;
    
    
    /**
     * 环绕增强
     *      前置增强：@Before()
     *      后置增强：@After()
     *
     * @param joinPoint joinPoint 可以获取到目标方法的参数、返回值等一系列东西
     *                  "@annotation(com.zixq.annotation.Log)" 是切点 pointcut
     */
    @Around("@annotation(com.zixq.annotation.Log)")
    public void around(ProceedingJoinPoint joinPoint) throws Throwable {

        long beginTime = System.nanoTime();

        try {
            // 执行业务方法
            joinPoint.proceed();
        } catch (Throwable e) {
            // 防止事务失效 抛出 RuntimeException
            throw new RuntimeException(e);
        }

        long endTime = System.nanoTime();

        // 保存日志
        saveLog(joinPoint, (endTime - beginTime));
    }

    /**
     * 保存日志
     *
     * @param time 业务执行的时长
     */
    private void saveLog(ProceedingJoinPoint joinPoint, long time) {

        LogEntity logEntity = new LogEntity();
        logEntity.setId(IdUtil.getSnowflakeNextId());

        HttpServletRequest request = HttpContextUtils.getHttpServletRequest();
        // 获取ip
        String ipAddr = IpUtils.getIpAddr(request);
        logEntity.setIp(ipAddr);

        // 获取注解上的用户名、操作类型
        MethodSignature method = (MethodSignature) joinPoint.getSignature();
        Log logAnnotation = method.getMethod().getAnnotation(Log.class);
        logEntity.setUsername(logAnnotation.username())
                .setOperation(logAnnotation.operation().name());

        // 获取请求参数 和 其值
        Object[] argsValue = joinPoint.getArgs();    // 获取参数值
        String[] parameterNames = method.getParameterNames();
        if (argsValue != null && parameterNames != null) {
            String params = "";
            for (int i = 0; i < argsValue.length; i++) {
                params += " " + parameterNames[i] + ": " + argsValue[i];
            }
            logEntity.setParams(params);
        }

        // 获取请求方式
        String requestMethod = request.getMethod();

        // 获取类名
        String className = joinPoint.getTarget().getClass().getName();
        // 获取方法名
        String methodName = method.getName();
        logEntity.setMethod(requestMethod + " " + className + "." + methodName + "()");

        // 模块名、业务耗时、创建时间
        logEntity.setModule(projectName)
                .setTime(time)
                .setCreateTime(LocalDateTime.now());

        // 保存日志
        logMapper.saveLog(logEntity);
    }
}
```



> **提示**
>
> 有时需要在如上面LogAspect中操作数据库，但又不可能在LogAspect所在模块引入数据库相关的东西，那么可以采用：
>
> - LogAspect所在模块定义操作数据库的service接口；
> - 然后在真正操作数据库的模块（如：xxx-manager）实现该接口操作数据库；
> - 最后在LogAspect中`装配`（`@Autowired` 或 `@resource`）其service接口类即可

涉及的两个工具类：

```java
package com.zixq.util;

import jakarta.servlet.http.HttpServletRequest;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

/**
 * <p>
 * Http工具类
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */
public class HttpContextUtils {
    /**
     * 获取HttpServletRequest
     */
    public static HttpServletRequest getHttpServletRequest() {
        return ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
    }
}
```



```java
package com.zixq.util;

import jakarta.servlet.http.HttpServletRequest;

/**
 * <p>
 * ip工具类
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */
public class IpUtils {
    /**
     * 获取IP地址
     *
     * 使用Nginx等反向代理软件， 则不能通过request.getRemoteAddr()获取IP地址
     * 如果使用了多级反向代理的话，X-Forwarded-For的值并不止一个，而是一串IP地址，X-Forwarded-For中第一个非unknown的有效IP字符串，则为真实IP地址
     */
    public static String getIpAddr(HttpServletRequest request) {

        String ip = request.getHeader("x-forwarded-for");
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
        }
        return "0:0:0:0:0:0:0:1".equals(ip) ? "127.0.0.1" : ip;
    }
}
```



6. Mapper

```java
package com.zixq.mapper;

import com.zixq.entity.LogEntity;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Mapper;

/**
 * <p>
 * log Mapper 接口
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */
@Mapper
public interface LogMapper {
    /**
     * 保存日志
     */
    @Insert("insert into log(id,module,username,operation,time,ip,method,params,create_time)" +
            "values(#{id},#{module},#{username},#{operation},#{time},#{ip},#{method},#{params},#{createTime})")
    void saveLog(LogEntity logEntity);
}
```



7. 【可选】让增强逻辑能在其他业务服务中使用

想让LogAspect这个切面类在其他的业务服务中进行使用，那么就需要该切面类纳入到Spring容器中。Spring Boot默认会扫描和启动类所在包相同包中的bean以及子包中的bean

假如LogAspect切面类不满足扫描条件，那么无法直接在业务服务中进行使用。因此此时可以通过自定义注解进行实现

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Import(value = LogAspect.class)	// 通过Import注解导入日志切面类到Spring容器中
public @interface EnableLogAspect {
    
}
```



8. 使用

1）、在需要使用的业务服务中引入前面1 - 4模块所在【不在同一模块的时候】

```xml
 <dependency>
     <groupId>com.zixieqing</groupId>
     <artifactId>common-log</artifactId>
     <version>1.0-SNAPSHOT</version>
 </dependency>
```

2）、启动类加上4中自定义的`@EnableLogAspect  // 开启日志增强功能`

```java
@EnableLogAspect
@SpringBootApplication
public class ManagerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ManagerApplication.class , args) ;
    }
}
```

3）、测试【在同一模块】

```java
package com.zixq.controller;

import com.zixq.CustomEnum.OperatorType;
import com.zixq.annotation.Log;
import com.zixq.entity.UserEntity;
import org.springframework.web.bind.annotation.*;

/**
 * <p>
 * user controller
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */
@RestController
@RequestMapping("user")
public class UserController {

    @Log(
            username = "zixieqing",
            operation = OperatorType.GET
    )
    @GetMapping("/get")
    public void getUser() {
        System.out.println("get user");
    }

    @Log(
            username = "zixieqing",
            operation = OperatorType.ADD
    )
    @PostMapping("/addUser")
    public void addUser(@RequestBody UserEntity userEntity) {
        System.out.println("userEntity = " + userEntity);
    }
}
```


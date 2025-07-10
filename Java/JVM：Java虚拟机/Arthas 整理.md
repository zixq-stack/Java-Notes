# Java应用在线调试Arthas整理



## Arthas简介

> 在学习Arthas之前，推荐先看后面的 [@美团技术团队：Java动态调试技术原理](#Java动态调试技术原理)，这样你会对它最底层技术有个了解。可以看下文中最后有个对比图：Greys(Arthas也是基于它做的二次开发)和Java-debug-tool



## Arthas是什么

`Arthas` 是Alibaba开源的Java诊断工具，深受开发者喜爱



## Arthas能解决什么问题

当你遇到以下类似问题而束手无策时，`Arthas`可以帮助你解决：

- 这个类从哪个 jar 包加载的? 为什么会报各种类相关的 Exception?
- 我改的代码为什么没有执行到? 难道是我没 commit? 分支搞错了?
- 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗?
- 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
- 是否有一个全局视角来查看系统的运行状况?
- 有什么办法可以监控到JVM的实时运行状态?

`Arthas`支持JDK 6+，支持Linux/Mac/Windows，采用命令行交互模式，同时提供丰富的 `Tab` 自动补全功能，进一步方便进行问题的定位和诊断





## Arthas资源推荐

- [官方在线教程(推荐)](https://arthas.gitee.io/)
- [常见问题](https://github.com/alibaba/arthas/issues?utf8=✓&q=label%3Aquestion-answered+)





## Arthas基于哪些工具发展而来

- [greys-anatomy](https://github.com/oldmanpushcart/greys-anatomy): Arthas代码基于Greys二次开发而来
- [termd](https://github.com/termd/termd): Arthas的命令行实现基于termd开发，是一款优秀的命令行程序开发框架
- [crash](https://github.com/crashub/crash): Arthas的文本渲染功能基于crash中的文本渲染功能开发，可以从[这里](https://github.com/crashub/crash/tree/1.3.2/shell)看到源码
- [cli](https://github.com/eclipse-vertx/vert.x/tree/master/src/main/Java/io/vertx/core/cli): Arthas的命令行界面基于vert.x提供的cli库进行开发
- [compiler](https://github.com/skalogs/SkaETL/tree/master/compiler) Arthas里的内存编绎器代码来源
- [Apache Commons Net](https://commons.apache.org/proper/commons-net/) Arthas里的Telnet Client代码来源
- `JavaAgent`：运行在 main方法之前的拦截器，它内定的方法名叫 premain ，也就是说先执行 premain 方法然后再执行 main 方法
- `ASM`：一个通用的Java字节码操作和分析框架。它可以用于修改现有的类或直接以二进制形式动态生成类。ASM提供了一些常见的字节码转换和分析算法，可以从它们构建定制的复杂转换和代码分析工具。ASM提供了与其他Java字节码框架类似的功能，但是主要关注性能。因为它被设计和实现得尽可能小和快，所以非常适合在动态系统中使用(当然也可以以静态方式使用，例如在编译器中)





## 同类工具有哪些

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

![dashboard](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240508132927083-483527748.png)





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

![trace](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240508132927013-204520453.png)





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

![web console](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240508132927003-321832597.png)





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



### [#](#如何跟踪某个方法的返回值、入参) 重要：如何跟踪某个方法的返回值、入参.... ?

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



### [#](#如何临时更改代码运行)  重要：如何临时更改代码运行?

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

<img src="https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240508132927735-1030999769.png" alt="image-20240113173653011" />



更多请看这里：[profiler](https://arthas.aliyun.com/doc/profiler.html)



### 更多

请参考: [官方Issue墙](https://github.com/alibaba/arthas/issues?q=label%3Auser-case)



## Arthas源码

首先我们先放出一张整体宏观的模块调用图：

![trace](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240508132927100-205232237.jpg)

源码理解可以这篇文章：https://yq.aliyun.com/articles/704228





## 参考资料

- https://www.cnblogs.com/muxuanchan/p/10097639.html
- https://www.cnblogs.com/yougewe/p/10770690.html
- https://help.aliyun.com/document_detail/112975.html~
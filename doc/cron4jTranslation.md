# Cron4j
- - -
一些必要的单词解释：
1. schedule 安排、调度（安排在有主语的时候使用 调用在没主语的时候使用）
2. scheduler 调度器
3. scheduling pattern 调度模式
4. reschedule 重新调度、安排
5. action/task 动作/任务
6. deschedule 脱离调度、安排
7. Daemon threads 守护线程 （其实daemon本意就有守护进程的意思 加个threads应该是强调线程而非进程）
8. Predictor 先知（故意翻译为这个中二的名字哈哈）、预报器（这才是常规翻译）
9. Crontab/Cron 定时任务工具
10. status tracking 状态跟踪
11. executor 执行器
12. collector 收集器
- - -
<span id="index"></span>
## Index
> 1. [快速开始](#1快速开始)
> 1. [调度模式 scheduling pattern](#2调度模式-scheduling-pattern)
> 1. [如何调度 schedule、重新调度 reschedule、脱离调度 deschedule一个任务](#3如何调度-schedule重新调度-reschedule脱离调度-deschedule一个任务task)
> 1. [如何调度系统程序](#4如何调度系统程序)
> 1. [如何从调度配置文件中调度程序](#5如何从调度配置文件中调度程序)
> 1. [创建自定义的任务 Task](#6创建自定义的任务-task)
> 1. [创建自定义的收集器 Collector](#7创建自定义的收集器-Collector)
> 1. [创建自定义的监听器来监控你的调度器](#8创建自定义的监听器来监控你的调度器)
> 1. [执行器 Executors](#9执行器-executors)
> 1. [手动启动任务](#10手动启动任务)
> 1. [在指定时区下运行](#11)
> 1. [守护线程 Daemon threads](#12)
> 1. [预报器 Predictor](#13)
> 1. [Cron解析器](#14)

- - -
<span id="1快速开始"></span>
### 1、快速开始：
cron4j的主要实体是‘scheduler’，实例化`it.sauronsoftware.cron4j.Scheduler`之后，你可以在一年当中任意的时间段执行任意的任务（Task）。

要使用cron4j scheduler 你需要做下面四步：
1. new一个Scheduler实例
2. 安排（schedule）你的动作（action）。你需要告诉scheduler对象你要做的事情是什么并且是在什么时候发生的。你可以指定（specify）一个实现了`java.lang.Runnable`接口的实例对象或者使用cron4j提供的`it.sauronsoftware.con4j.Task`类来实例一个对象来告诉它你要做什么事情，然后你可以使用一个`it.sauronsoftware.cron4j.SchedulingPattern`类的实例或者是一个字符串来代表指定的‘scheduling pattern’来告诉它你要在什么样的时间执行你的action。
3. 开启（start）你的scheduler实例。
4. 当你不再需要它的时候，停止（stop）它。

```
import it.sauronsoftware.cron4j.Scheduler;

public class Quickstart {

	public static void main(String[] args) {
		// 第一步创建实例
		Scheduler s = new Scheduler();
		// 安排一个每分钟执行一次的任务
		s.schedule("* * * * *", new Runnable() {
			public void run() {
				System.out.println("Another minute ticked away...");
			}
		});
		// 开启你的调度器
		s.start();
		// 10分钟后执行之后的代码
		try {
			Thread.sleep(1000L * 60L * 10L);
		} catch (InterruptedException e) {
			;
		}
		// 10分钟后停止你的调度器
		s.stop();
	}

}
```

上面的代码会每隔一分钟执行一次run方法并打印出"Another minute ticked away..."句子。

你需要知道一些关键的概念：

* 你可以安排任意数量的任务。
* 你可以在任意时间安排任务，即使是在调度器（scheduler）已经被开启过之后。
* 你可以改变已经安排过的任务的‘scheduling pattern（调度模式）’，即使是当调度器正在运行的时候（reschedule operation 重新调度操作）。
* 你可以移除之前安排过的任务，即使是当调度器正在运行的时候（deschedule operation 脱离调度操作）。
* 你可以任意次开启或停止调度器。
* 你可以使用文件配置来安排任务。
* 你可以从任意文件源配置中安排任务。
* 你可以提供一个监听器（listener）给调度器，用来接收执行过的任务的事件。
* 你可以控制任何一个正在进行的任务。
* 你可以不使用任何的‘scheduling pattern'就可以手动启动任务。
* 你可以改变调度器工作的时区。
* 你可以在你的‘scheduling pattern’使用到调度器之前验证它的工作模式。
* 你可以预报出你的‘scheduling pattern’可能造成的任务异常。

[回到索引](#index)
- - -
<span id="2调度模式-scheduling-pattern"></span>
### 2、调度模式 scheduling pattern
'scheduling pattern'是一个 UNIX 的类定时任务模式，由一个以空格分隔为五个部分的字符串组成。每个部分代表着：

分钟子模式（Minutes sub-pattern）：
> 规定一小时中的哪个分钟会执行任务，取值范围为0-59。

小时子模式（Hour sub-pattern）：
> 规定一天中的哪个小时会执行任务，取值范围为0-23。

日期子模式（Days of mouth sub-pattern）：
> 规定一个月中的哪一号会执行任务，取值范围为1-31，特殊值“L”可以代表当月的最后一天。

月份子模式（Months sub-pattern）：
> 规定一年中的哪一月会执行任务，取值范围从1（January）-12（December），这个子模式也允许月份英文缩写如：jan、feb、mar、...、dec。

周几子模式（Days of week sub-pattern）：
> 规定一周中的周几会执行任务，取值范围0（sunday）-6（monday），这个子模式同样允许英文缩写（是否忽略大小写 并未做考究 请按照官方举例 首字母大写）：sun、mon、...、sat。

模式还允许使用星号通配符来代表：小时中的每分钟、日中的每小时、月中的每一天、年中的每一月、一周中的每一天。

一旦调度器被开启，任务会在每一个调度模式匹配为true的时候执行一次。

下面是一些举例：

> 5 * * * *
>
> 每小时的过五分执行一次（1：05、2：05 etc）

> \* * * * *
>
> 每分钟执行一次

> \* 12 * * Mon
>
> 每周一的12时内的每分钟都执行一次

> \* 12 16 * Mon
>
> 每月的16号的12时内的每分钟都执行一次

每个子模式都可以包含一个或多个逗号来分隔模式值
> 59 11 * * 1,2,3,4,5
>
> 每周一、周二、周三、周四、周五的 11:59am 会执行一次

取值间隔也可以使用“-”号
> 59 11 * * 1-5
>
> 和上面的结果一样

斜杠也可以运用到子模式当中，用来识别子模式取值范围内的分步值。

它有两种运用方式：
* */c
* a-b/c

第一种会匹配到子模式范围0到最大值中的每个c增值 包含0值

第二种会匹配到范围a到b中的每个c增值 包含a值

> \*/5 * * * *
>
> 每小时内从0分开始每过5分钟就执行一次（0：00、0：05、0：10、...）

> 3-18/5 * * * *
>
> 每小时中从3分到18分中每过5分钟就执行一次（0：03、0：08、0：13、0：18、1：03、...）

> \*/15 9-17 * * *
>
> 每天的9时到17时中从0分开始每过一刻钟就执行一次（9：00、9：15、...、最后一次执行会是在17：45分）

上述所有规则都可以混合使用

> \* 12 10-16/2 * *
>
> 每月的10号到16号中每过两天中当天12时中的每分钟执行一次（也即10、12、14、16号中...）

> \* 12 1-15,17,20-25 * *
>
> 每月的1到15号、17号、20到25号当天中的12时中的每分钟执行一次

cron4j允许你使用“|”符号连接多个调度模式组成一个调度模式

> 0 5 * * *|8 10 * * *|22 17 * * *
>
> 每天的5：00、10：08、17：22执行一次

[回到索引](#index)
- - -
<span id="3如何调度-schedule重新调度-reschedule脱离调度-deschedule一个任务task"></span>
### 3、如何调度 schedule、重新调度 reschedule、脱离调度 deschedule一个任务（Task）

##### （1）调度
创建Task的最简单最常用的方法就是实现`java.lang.Runnable`接口，任务创建好的时候，它可以被`it.sauronsoftware.cron4j.Scheduler.schedule(String, Runnable)`方法安排进调度器中，如果调度模式有格式异常，将会抛出`it.sauronsoftware.cron4j.InvalidPatternException`异常。

创建Task的另一种方法就是继承抽象方法`it.sauronsoftware.cron4j.Task`，这种方式比上一种方式更加强大，它可以使开发者访问一些cron4j提供的特性。你可以在“[建立自定义的任务 Task](#6)”小节中了解到更多相关用法。Task的实例可以被`schedule(String, Task)`方法和`schedule(SchedulingPattern, Task)`方法安排进调度器中。

##### （2）重新调度/脱离调度
在调度器对象的调度方法`schedule`会返回一个ID值（String类型）用来识别和检索已经安排过的操作。

这个ID可以被用来之后做：
* 重新调度该任务（需要改变它的调度模式）
* 把该任务脱离调度（把任务从调度器中移除）

可以调用这两个方法取重新调度该任务：
* `reschedule(String, String)`
* `reschedule(String, SchedulingPattern)`

可以调用这个方法让任务脱离调度：
* `deschedule(String)`

[回到索引](#index)
- - -
<span id="4如何调度系统程序"></span>
### 4、如何调度系统程序
* 使用类`ProcessTask`可以很简单的完成系统程序的调度
```
ProcessTask task = new ProcessTask("C:\\Windows\\System32\\notepad.exe");
Scheduler scheduler = new Scheduler();
scheduler.schedule("* * * * *", task);
scheduler.start();
// ...
```
* 多个程序参数可以作为字符串数组去代替一条参数
```
String[] command = { "C:\\Windows\\System32\\notepad.exe", "C:\\File.txt" };
ProcessTask task = new ProcessTask(command);
// ...
```
* 程序的环境变量可以作为第二组字符串数组参数传入，其中的对象必须是‘NAME=VALUE’的形式
```
String[] command = { "C:\\tomcat\\bin\\catalina.bat", "start" };
String[] envs = { "CATALINA_HOME=C:\\tomcat", "JAVA_HOME=C:\\jdks\\jdk5" };
ProcessTask task = new ProcessTask(command, envs);
// ...
```
* 默认工作目录可以通过传入第三组参数去改变
```
String[] command = { "C:\\tomcat\\bin\\catalina.bat", "start" };
String[] envs = { "CATALINA_HOME=C:\\tomcat", "JAVA_HOME=C:\\jdks\\jdk5" };
File directory = "C:\\MyDirectory";
ProcessTask task = new ProcessTask(command, envs, directory);
// ...
```
* 如果你只想改变工作目录而不想使用环境变量，你可以在envs位置传入null值
```
ProcessTask task = new ProcessTask(command, null, directory);
```
当evns为null的时候，程序会继承当前JVM环境下工作的所有环境变量。

环境变量和工作目录也可以通过调用`setEnvs(String[])`和`setDirectory(java.io,File)`方法来设置

程序的标准输出和标准错误输出管道可以通过`setStdoutFile(java.io.File)`和`setStderrFile(java.io.File)`方法重定向到指定文件
```
ProcessTask task = new ProcessTask(command, envs, directory);
task.setStdoutFile(new File("out.txt"));
task.setStderrFile(new File("err.txt"));
```
同样的标准输入管道可以从已存在的文件中读取，通过调用方法`setStdinFile(java.io.File)`
```
ProcessTask task = new ProcessTask(command, envs, directory);
task.setStdinFile(new File("in.txt"));
```

[回到索引](#index)
- - -
<span id="5如何从调度配置文件中调度程序"></span>
### 5、如何从调度配置文件中调度程序
cron4j调度器可以从调度配置文件中调度一系列的程序流程

你需要准备一个调度配置文件，这和UNIX中crontab的用法非常相似，并且把文件通过`scheduleFile(File)`方法注册到调度器里面。

调度配置文件也可以通过`deschedule(File)`方法来脱离调度。

已经调度过的调度配置文件可以调用`getScheduledFiles()`方法来检索到。

已经注册过的调度配置文件会每分钟都被解析一次，调度器会根据调度配置文件去运行所有调用‘scheduling pattern（调度模式）’来正确声明的、匹配当前系统时间的程序。

cron4j的调度配置文件的声明规则可以从“[Cron解析器](#14)”小节中了解到。

[回到索引](#index)
- - -
<span id="6创建自定义的任务-task"></span>
### 6、创建自定义的任务 Task

一个`java.lang.Runnable`对象是一个简单的Task，但是为了获得对整个任务的控制权你还需要继承`it.sauronsoftware,cron4j.Task`类（注意这是一个抽象类）。

有两种比较简单的创建形式：
（1）实现Runnable接口的时候：任务就是run方法所执行的语句。
（2）继承Task抽象类的时候：任务就是需要实现的`execute(TaskExecutionContext)`方法所执行的语句。

execute(TaskExecutionContext)方法提供了一个`it.sauronsoftware.cron4j.TaskExecutionContext`实例对象，这是在run方法中所没有的。

你可以用这个对象做这些事情来操作当前任务：

* status tracking 状态跟踪
> 任务可以和它的执行器进行通信，可以通过文本描述来向外通知它的internal state（内部状态）
>
> 如果你想要你的任务支持这个功能的话，你可以重载`supportsStatusTracking()`方法，这个方法仅需要实现一个true为返回值就可以表示开启该功能。
>
> 当你重载过这个方法之后，在`execute(TaskExecutionContext)`方法里面就可以调用`context.setStatusMessage(String)`方法，这会给该任务的执行器发一条状态消息。这个状态消息，通过执行器，可以被外部用户索引到（具体看“[执行器 Executors](#9执行器-executors)”小节）。
>

* completeness tracking 完成度跟踪
> 任务可以和它的执行器进行通信，可以通过数字值来向外通知它的completeness level（完成度），
>
> 如果你想要你的任务支持这个功能的话，你可以重载`supportsCompletenessTracking()`方法，这个方法仅需要实现一个true为返回值就可以表示开启该功能。
>
> 当你重载过这个方法之后，在`execute(TaskExecutionContext)`方法里面就可以调用`context.setCompleteness(double)`方法，这个方法需要传递一个0~1之间的double值，这会给该任务的执行器发一个完成度值。这个完成度值，通过执行器，可以被外部用户索引到（具体看“[执行器 Executors](#9执行器-executors)”小节）。

* paused 被暂停
> 任务可以根据情况而暂停。
>
> 如果你想要你的任务支持这个功能的话，你可以重写`canBePaused()`方法，这个方法仅需要实现一个true为返回值就可以表示开启该功能。
>
> 当你重载过这个方法之后，你需要定期地（原文此处为：you have to periodically call the...）调用`context.pauseIfRequested()`方法，这会暂停任务的执行，直到被外部用户恢复或者终止当前任务。

* stopped 被终止
> 任务可以根据情况而终止。
>
> 如果你想要你的任务支持这个功能的话，你可以重载`canBeStopped()`方法，这个方法仅需要实现一个true为返回值就可以表示开启该功能。
>
> 当你重载过这个方法之后，你需要定期地（...）调用`context.isStopped()`方法，当被外部用户命令终止的时候，这会返回一个true值（具体看“[执行器 Executors](#9执行器-executors)”小节）。这时候你有义务处理好这个任务在执行时所反馈出来的事件结果，好让它在正在运行的状态下平稳地（原文：gently）结束。

* 索引调度器
> 通过context对象，你可以通过`getScheduler()`索引到调度本身调度器。

* 索引执行器
> 通过context对象，你可以通过`getTaskExecutor()`索引到调度本身调度器。

一个自定义的任务可以被任务收集器（task collector）所立即调度、运行、或者返回。

<br>

> *译者文外补充：可以查看Task类的源码，不难发现，上述所要重载的方法在源码中也仅仅只是返回false值，也即默认是关闭这些功能的，我们只有重载为true才能开启和使用它们。*

[回到索引](#index)
- - -
<span id="7创建自定义的收集器-Collector"></span>
### 7、创建自定义的收集器 Collector

通过任务收集器提供的API，你可以在调度器里面创建和插入你自己的任务源（task source）。

cron4j调度器支持注册一个或多个`it.sauronsoftware.cron4j.TaskCollector`实例，你只需要调用`addTaskCollector(TaskCollector)`方法即可。

被注册的收集器可以被调度器对象调用`getTaskCollectors`方法索引到，之前的收集器可以调用`removeTaskCollector(TaskCollector)`方法从调度器中移除。

收集器可以在任意的时间被添加（注册）、查询（索引）、移除，即使是在调度器正在运行的状态下也可以。

每一个被注册过的收集器每隔一分钟都会被调度器去索引一次，调度器会调用收集器的`collector.getTasks()`方法。这个实现方法会返回一个`it.sauronsoftware,cron4j.TaskTable`实例，我们把这个实例称为任务表。

每一个任务表都包含了本收集器中所有的任务实例和该任务对应的调度模式实例。一旦该表被检索到，调度器就会检查被记录（原文使用reported）到的对象，然后执行所有使用‘scheduling pattern（调度模式）’来正确声明的、匹配当前系统时间的任务。

一个自定义的收集器可以配合外部任务源来约束调度器的行为，比如数据库、或者xml文件，这些同样支持在运行时更改和管理的源。

<br>

> *译者文外补充：在[下面](#1collector-exp)贴出译者实践演示代码，代码中演示了如何向一个调度器中注册、移除收集器，并且查看收集器的信息，同时在代码运行的过程中也演示了调度器每分钟索引收集器task的过程。*
>
>*读者可以自行研究代码，花上4分钟体会一下。读者也可以从`TaskController`类的源码开始阅读下去，特别是`TaskTable`类中，仅仅只有几个简单易懂的API，了解过后你会发现这套流程其实并不难走通。*

[回到索引](#index)
- - -
<span id="8创建自定义的监听器来监控你的调度器"></span>
### 8、创建自定义的监听器来监控你的调度器
cron4j提供了`it.sauronsoftware,cron4j.SchedulerListener`类，我们可以使用它的API来对调度器的事件进行监听。

调度监听器需要实现以下方法：
```
taskLaunching(TaskExecutor)
这个方法会在每个调度任务启动的时候被调度器调用

taskSucceeded(TaskExecutor)
这个方法会在每个任务成功地执行完毕的时候被调用

taskFailed(TaskExecutor, Throwable)
这个方法会在每个任务执行失败的时候被调用
```
你可以从“[执行器 Executors](#9执行器-executors)”小节中了解到更多的信息。

当你准备好一个调度监听器（SchedulerListener）的时候，你可以调用调度器的`addSchedulerListener(SchedulerListener)`方法将这个监听器注册到该调度器中。

你可以调用`removeSchedulerListener(SchedulerListener)`方法移除已经注册的监听器。

你可以调用`getSchedulerListeners()`方法获取到所有在本调度器注册的监听器。

调度监听器可以在任何时候被注册或者移除，即使是在调度器正在运行的时候。

[回到索引](#index)
- - -
<span id="9执行器-executors"></span>
### 9、执行器 Executors
每当调度器被开启并且运行的时候，你可以通过方法索引到它的执行器。

执行器非常像一个线程，它是被调度器用来执行任务的利器。

你可以调用`Scheduler.getExecutingTasks()`方法来获得当前正在运行的执行器。

你也可以通过调度监听器来获得执行器（见“[8、创建自定义的监听器来监控你的调度器](#8创建自定义的监听器来监控你的调度器)”小节）。

每一个执行器，代表着一个`it.sauronsoftware.cron4j.TaskExecutor`实例，执行不同的任务。

执行器中的任务可以被`TaskExecutor.getTask()`方法索引到。

执行器的状态可以通过`TaskExecutor.isAlive()`方法来检查：如果当前执行器正在运行则返回true。

如果执行器处于运行状态，那么一直到整个执行过程完毕之前，你都可以通过`join()`方法来暂停当前线程

* 关于status tracking 状态跟踪
> 你可以调用`TaskExecutor.supportsStatusTracking()`方法，如果它会返回一个true值，则表示当前正在执行的任务支持状态跟踪功能。这意味着任务可以和它的执行者进行沟通，当然只能传递字符串。当前的状态信息可以被执行器调用`TaskExecutor.getStatusMessage()`方法索引到。

* 关于completeness tracking 完成度跟踪
> 你可以调用`supportsCompletenessTracking()`方法来检查当前正在执行的任务是否支持完成度跟踪。如果支持，那么你可以调用`TaskExecutor.getCompleteness()`方法来索引任务完成度值，它会返回一个0（未开始）~1（已完成）之间的数值。

* 关于 paused 暂停
> 你可以调用`TaskExecutor.canBePaused()`方法来检查当前正在执行的任务是否支持运行时暂停执行的功能。如果支持，那么你可以你可以调用`TaskExecutor.paused()`方法来暂停当前任务的执行。你还可以调用`TaskExecutor.isPaused()`方法来检查当前任务是否处于暂停状态。被暂停的执行器可以通过`TaskExecutor.resume()`方法来恢复运行。

* 关于 stopped 终止
> 你可以调用`TaskExecutor.canBeStopped()`方法来检查当前正在执行的任务是否支持运行时终止执行的功能。如果支持，那么你可以你可以调用`TaskExecutor.stop()`方法来终止当前任务的执行。同样你可以调用`TaskExecutor.isStopped()`方法去检查当前执行器是否被终止。
>
> **注意：被终止过的执行器不能再恢复运行。**

* 其他API
> `TaskExecutor.getStartTime()`
>
>它会返回一个时间标记（time stamp）来告诉你执行器启动的时间，或者一个小于0的值来表示执行器还没开始启动。

> `TaskExecutor.getScheduler()`
>
> 它会返回当前执行器所属的调度器对象。

> `TaskExecutor.getGuid()`
>
> 它会返回当前执行器所唯一对应的纯文本的GUID值。

* 关于事件驱动

> 执行器同时也提供了它自己的事件驱动API，你可以通过`it.sauronsoftware.cron4j.TaskExecutorListener`类来访问它们。
>
> 你可以分别调用：
>
> `addTaskExecutorListener(TaskExecutorListener)`
>
> `removeTaskExecutorListener(TaskExecutorListener)`
>
> `getTaskExecutorListeners()`
>
> 方法来注册、移除、索引到执行监听器。
>
> 一个执行监听器需要实现以下方法：
> ```
> executionPausing(TaskExecutor)
> 该方法会在执行器被请求暂停正在运行的任务时调用。传入的参数代表着被请求暂停任务执行的执行器对象。
>
> executionResuming(TaskExecutor)
> 该方法会在执行器被请求恢复正在被暂停的任务时调用。传入的参数代表着被请求恢复任务执行的执行器对象。
>
> executionStopping(TaskExecutor)
> 该方法会在执行器被请求终止任务执行时调用。传入的参数代表着被请求的执行器对象。
>
> executionTerminated(TaskExecutor, Throwable)
> 该方法会在执行器将任务执行完毕的时候被调用。传入的第一个参数代表着该执行器对象，第二个参数是可选项，代表着迫使执行器终止执行任务的异常，如果任务正确的执行成功的话，该对象值为null。
>
> statusMessageChanged(TaskExecutor, String)
> 该方法会在每次运行时任务的状态信息发生改变的时候调用。传入的第一个参数代表着该执行器对象，第二个参数则是新发布的任务状态信息。
>
> completenessValueChanged(TaskExecutor, double)
> 该方法会在每次运行时任务的完成度值发生改变的时候调用。传入的第一个参数代表着该执行器对象，第二个参数则是取值范围为0~1之间的新发布的完成度值。
> ```

<br>

> *译者文外补充：正如官方文档所说的一样，执行器非常像一个线程，所以它提供的API也相对线程性细致和线程性复杂，使用的时候一定要考虑周全，同时还要分清楚调度器和执行器的关系，以及调度监控器和执行监控器的事件监听方法的调用时机。*

[回到索引](#index)
- - -
<span id="10手动启动任务"></span>
### 10、手动启动任务








- - -
# 部分实践演示代码
<span id="1collector-exp"></span>
### 1、Collector exp
```
public class CollectorExp {

    public static void main(String[] args) {
        Scheduler scheduler = new Scheduler();

        TaskCollector c1 = new TaskCollector() {
            @Override
            public TaskTable getTasks() {
                System.out.println("过了一分钟 调度器又来索引我啦");
                TaskTable taskTable = new TaskTable();
                taskTable.add(new SchedulingPattern("* * * * *"), new MyTask("one"));
                taskTable.add(new SchedulingPattern("*/2 * * * *"), new MyTask("two"));
                return taskTable;
            }
        };

        TaskCollector c2 = () ->{
            System.out.println("过了一分钟 调度器又来索引我啦");
            TaskTable taskTable = new TaskTable();
            taskTable.add(new SchedulingPattern("* * * * *"), new MyTask("three"));
            taskTable.add(new SchedulingPattern("*/2 * * * *"), new MyTask("four"));
            return taskTable;
        };

        scheduler.addTaskCollector(c1);
        scheduler.addTaskCollector(c2);

        showController(scheduler);

        scheduler.start();

        try {
            Thread.sleep(2000L * 60L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("两分钟到 移除c1");
        scheduler.removeTaskCollector(c1);
        showController(scheduler);

    }

    static void showController(Scheduler scheduler){
        TaskCollector[] taskCollectors = scheduler.getTaskCollectors();
        System.out.println("|----当前调度器中有"+taskCollectors.length+"个收集器");
        for (int i = 0 ; i < taskCollectors.length ; ++i){
            System.out.println("|----|----当前显示第"+(i+1)+"个收集器的信息");
            TaskCollector now = taskCollectors[i];
            TaskTable tasks = now.getTasks();
            System.out.println("|----|----|----当前收集器有"+tasks.size()+"个任务");
            for (int j = 0 ; j < tasks.size() ; ++j){
                System.out.println("|----|----|----|----当前显示第"+(j+1)+"个任务信息");
                System.out.println("|----|----|----|----Task:["+tasks.getTask(j)+"] and scp:["+tasks.getSchedulingPattern(j)+"]");
            }
        }
    }

}

class MyTask extends Task{

    private String num;

    MyTask(String num) {
        this.num = num;
    }

    @Override
    public void execute(TaskExecutionContext taskExecutionContext) throws RuntimeException {
        LocalTime now = LocalTime.now();
        System.out.println("This is Task "+num+" ! [ " + now.getHour() + " : " + now.getMinute() + " ]");

    }
}
```
[返回Collector小节](#7创建自定义的收集器-Collector)
- - -

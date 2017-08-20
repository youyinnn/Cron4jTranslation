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
- - -
<span id="index"></span>
## Index
> 1. [快速开始](#1)
> 1. [调度模式 scheduling pattern](#2)
> 1. [如何调度 schedule、重新调度 reschedule、脱离调度 deschedule一个任务](#3)
> 1. [如何调度系统进程](#4)
> 1. [如何从文件配置中调度任务](#5)
> 1. [建立你的任务 Task](#6)
> 1. [建立你的收集器 Collector](#7)
> 1. [建立你的调度器的监听器](#8)
> 1. [执行者Executors](#9)
> 1. [手动启动任务](#10)
> 1. [在指定时区下运行](#11)
> 1. [守护线程 Daemon threads](#12)
> 1. [预报器 Predictor](#13)
> 1. [Cron解析器](#14)

- - -
<span id="1"></span>
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
<span id="2"></span>
### 2、调度模式
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
> 每小时的过五分执行一次（1：05、2：05 etc）

> \* * * * *
> 每分钟执行一次

> \* 12 * * Mon
> 每周一的12时内的每分钟都执行一次

> \* 12 16 * Mon
> 每月的16号的12时内的每分钟都执行一次

每个子模式都可以包含一个或多个逗号来分隔模式值
> 59 11 * * 1,2,3,4,5
> 每周一、周二、周三、周四、周五的 11:59am 会执行一次

取值间隔也可以使用“-”号
> 59 11 * * 1-5
> 和上面的结果一样

斜杠也可以运用到子模式当中，用来识别子模式取值范围内的分步值。
它有两种运用方式：
* */c
* a-b/c
第一种会匹配到子模式范围0到最大值中的每个c增值 包含0值
第二种会匹配到范围a到b中的每个c增值 包含a值

> \*/5 * * * *
> 每小时内从0分开始每过5分钟就执行一次（0：00、0：05、0：10、...）

> 3-18/5 * * * *
> 每小时中从3分到18分中每过5分钟就执行一次（0：03、0：08、0：13、0：18、1：03、...）

> \*/15 9-17 * * *
> 每天的9时到17时中从0分开始每过一刻钟就执行一次（9：00、9：15、...、最后一次执行会是在17：45分）

上述所有规则都可以混合使用

> \* 12 10-16/2 * *
> 每月的10号到16号中每过两天中当天12时中的每分钟执行一次（也即10、12、14、16号中...）

> \* 12 1-15,17,20-25 * *
> 每月的1到15号、17号、20到25号当天中的12时中的每分钟执行一次

cron4j允许你使用“|”符号连接多个调度模式组成一个调度模式

> 0 5 * * *|8 10 * * *|22 17 * * *
> 每天的5：00、10：08、17：22执行一次

[回到索引](#index)
- - -
<span id="3"></span>
### 3、如何调度 schedule、重新调度 reschedule、脱离调度 deschedule一个任务

##### （1）调度
创建Task的最简单最常用的方法就是实现`java.lang.Runnable`接口，任务创建好的时候，它可以被`it.sauronsoftware.cron4j.Scheduler.schedule(String, Runnable)`方法安排进调度器中，如果调度模式有格式异常，将会抛出`it.sauronsoftware.cron4j.InvalidPatternException`异常。

创建Task的另一种方法就是继承抽象方法`it.sauronsoftware.cron4j.Task`，这种方式比上一种方式更加强大，它可以使开发者访问一些cron4j提供的特性。你可以在“[建立你的任务 Task](#6)”章节中了解到更多相关用法。Task的实例可以被`schedule(String, Task)`方法和`schedule(SchedulingPattern, Task)`方法安排进调度器中。

##### （2）重新调度/脱离调度
在调度器对象的调度方法`schedule`会返回一个ID值（String类型）用来识别和检索已经安排过的操作。

这个ID可以被用来之后做：
* 重新调度该任务（需要改变它的调度模式）
* 把某个任务脱离调度（把任务从调度器中移除）

可以使用这两个方法取重新调度该任务：
* `reschedule(String, String)`
* `reschedule(String, SchedulingPattern)`

可以使用这个方法让任务脱离调度：
* `deschedule(String)`

[回到索引](#index)
- - -
<span id="4"></span>

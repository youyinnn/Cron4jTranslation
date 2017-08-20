# Cron4j : A pure Java cron-like scheduler
- - -
创立时间：2017年8月20日13:35:01

官方文档地址：http://www.sauronsoftware.it/projects/cron4j/
- - -
Maven依赖：
```
<dependency>
    <groupId>it.sauronsoftware.cron4j</groupId>
    <artifactId>cron4j</artifactId>
    <version>2.2.5</version>
</dependency>
```
- - -
#### 前言：
> 这是一篇针对cron4j官方英文文档的翻译。
>
> 该翻译并不是来自官方的，仅仅是自学用的翻译，如有翻译错误，请在issue中指出。
>
> 本人并仅仅英语四级压线水平，所以文档并没有严格按照语法规范翻译，首先准从本人自己的理解（如有理解错误也请在issue中指出），再尽量翻译成通俗的语句，要求的是能根据本文快速掌握cron4j工具。一切以实践为标准，我在学习的时候也会先实践一番，再结合实践翻译到文档中。
>
> 虽然官方文档并不难看懂，但是本着高尚的自学精神，再加上暑假有点无聊，我尝试着翻译这篇Java工具包的技术文档。
>
> 在JavaWeb开发中，一些后台业务场景会有需要定时任务的需求，这些定时任务如果人工取执行的话就会显得非常蠢，所以就有了定时任务工具包/框架的出现。
>
> 其实流行的定时任务框架就像Quartz这样的，应该是运用到生产环境中比较好的选择，但是Quartz的官方文档结构有点蛋疼（也可能是我没细看），并不能深入浅出的让开发者循序渐进的掌握它。而在一些博客中对比也谈到Quartz比Cron4j臃肿一些，性能不知对比得怎么样，不过我猜应该是Quartz要好一点，毕竟持续到近两个月前Quartz还在继续维护当中，而Cron4j最近的发布时间是：28-Dec-2011（膜拜）。
>
> 我为什么选择cron4j，是因为最近在学JFinal3.2，里面插件扩展的章节介绍到了cron4j，我想一个最求设计极简和性能至上的框架，就算它用到了6年前的一款老工具，也应该是经过深思熟虑才会选择它的（毕竟业内Velocity也是一名老兵不是么），在学习cron4j的过程中也确实感受到了它的“pure”之意，所以我也愿意花一些时间来去翻译它的官方文档和学习它。
- - -
#### Overview部分：
cron4j是Java平台的一个调度器（也就是任务调度工具/框架），它非常像UNIX系统下的具有进程守护的定时任务工具cron。

有了cron4j，你可以在你规定好的时间内在Java应用程序中执行你指定的任务，而这只需要你制定一些简单的规则。

虽然Java平台已经内置了一个由`java.util.Timer`类实例化的调度器，但是cron4j走的是和前者不同的另一条路子。

你可以说`java.util.Timer`调度器是
> “从现在开始过5分钟后启动这个任务”

或者说
> “从现在开始过5分钟后执行这个任务，然后每10分钟重复执行它”。

这就是`java.util.Timer`。

而cron4j调度器会让你稍微多做一些复杂的事情，
比如：
> “再每个周一的12时执行这个任务”
> “每隔5分钟执行这个任务，但是周末期间可以不执行”
> “在8：00am到8：00pm之间的每个小时执行一次任务，而在8：00pm到8：00am之间的每5分钟执行一次任务”
> “除了7月和8月之外的月份内并且在一周内除了周日之外，每天都执行一次任务”

这些蜜汁操作，想要实现它们你只需要简单的写一小行代码就可以Duang出来。

把cron4j使用到你的项目里面其实非常简单，你只需要掌握一些常用API就足够了。启动定时任务的启动规则必须是一个字符串表达式，它被称为**scheduling pattern（调度模式）**，它的语法等同于UNIX系统中crontab所使用的语法一样。如果你了解过UNIX中crontab的操作，那么恭喜你，你已经掌握本工具的一大半了。如果你不会，don't worry：crontab的调度模式你只需要花上几分钟就能掌握（骗人！），再说了，后面还有documentation给你学习呢。
- - -
运行要求：

你可以在任何Java平台使用它。

License:
```
cron4j is Free Software and it is licensed under LGPL (you will find a copy of the license bundled into the downloadable software distribution).
```

Feedback

...

Make a donation

...

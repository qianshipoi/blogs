---
title: Use Quartz
published: 2022-06-13
description: ''
image: ''
tags: ['Dotnet']
category: 'Develop'
draft: false
---

### 安装 Quartz

```shell
Install-Package Quartz
```

### 常见定义

##### `StdSchedulerFactory` 调度程序工厂

>  用于创建调度程序
>
> ```c#
> StdSchedulerFactory factory = new StdSchedulerFactory();
> IScheduler scheduler = await factory.GetScheduler();
> ```

##### `IScheduler` 调度程序接口

>用于定义并执行任务
>
>```c#
>// ...
>await scheduler.ScheduleJob(job, trigger);	// 定义任务
>await scheduler.Start();	// 任务开始
>await Task.Delay(TimeSpan.FromSeconds(60));
>await scheduler.Shutdown();		// 任务关闭
>```

##### `JobBuilder` 用户构建任务 `IJobDetail` 实例

>```C#
>IJobDetail job = JobBuilder.Create<HelloJob>()
>    .WithIdentity("myJob", "group1") // name "myJob", group "group1"
>    .UsingJobData("params", "传递参数")
>    .Build();
>```
>
>`Create<T>()` 方法： 创建 `JobBuilder` 实例。`T` : `IJob` 实现类。
>
>`.WithIdentity("name","group") ` 方法： 设置任务唯一身份。
>
>`.Build()` 方法：构建 `IJobDetail` 实例。
>
>`.UsingJobData(key, value)` 方法： 设置`JobDataMap` 在工作类方法中能通过 `context.JobDetail.JobDataMap` 获取值。也可以在工作类中创建同名属性，`Quartz`会在创建工作类时根据类属性在 `JobDataMap`中查找并赋值。

##### `IJobDetail` 任务详细信息接口

##### `TriggerBuilder` 用于构建触发器`ITrigger`实例

>```csharp
>ITrigger trigger = TriggerBuilder.Create()
>    .WithIdentity("myTrigger", "group1")
>    .StartNow()
>    .WithSimpleSchedule(x => x
>        .WithIntervalInSeconds(40)
>        .RepeatForever())
>    .Build();
>```
>
>`Create()`方法： 创建`TriggerBuilder`实例。
>
>`.WithIdentity("name","group") ` 方法： 设置任务唯一身份。
>
>`.StartNow()` 方法： 立即执行。
>
>`.WithSimpleSchedule(Action<SimpleScheduleBuilder> action)` 方法：定义任务执行周期。
>
>`.Build()` 方法：构建触发器 `ITrigger` 实例。
>
>`SimpleScheduleBuilder`
>
>1. `WithIntervalInSeconds` 多少秒触发一次。
>2. `RepeatForever` 永远循环。

#####  `ITrigger` 触发器接口

##### `Trigger` 触发器

>常用特性
>
>1. `JobKey` ：唯一身份标识。
>2. `StartTimeUtc`：定义该触发器在什么时候首次生效，该值是`DateTimeOffset`对象。或者标记计划执行时间。
>3. `EndTimeUtc`：定义任务结束时间。
>4. `Priority`：优先级，由于工作线程数量有限，当同一时间执行多条任务时，可以设置优先级依次触发。值为整数，默认值为`5`。当检测到触发器作业需要恢复时，将按照原始触发器相同的优先级安排恢复。
>5. `Misfire Instuctions` ：触发器的另一个重要属性是其“失火指令”。如果持久触发器“未命中”它的触发时间，则会出现错误，因为在调度程序关闭时，或因为Quartz.net的线程池中没有可用的线程来执行作业。不同的触发类型对它们提供了不同的错误指令。默认情况下，它们使用“智能策略”指令 - 基于触发类型和配置具有动态行为。当调度程序启动时，它会搜索具有错误的任何持久触发器，然后基于其单独配置的错误指令更新它们中的每一个。当您在自己的项目中开始使用Quartz.net时，您应该熟悉在给定触发器类型上定义的错误指令，并在其API文档中解释。在每个触发器类型的教程教训中，将在特定于每个触发类型的教程课程中给出有关错误指令的更具体信息。
>6. `Calendars`：日历，可以通过设置日历达到任务的效果。

##### `HolidayCalendar` 假期日历

>```csharp
>HolidayCalendar holidayCalendar = new HolidayCalendar();
>holidayCalendar.AddExcludedDate(DateTime.Now);
>
>await scheduler.AddCalendar("myHoliday", holidayCalendar, false,false);
>
>ITrigger trigger = TriggerBuilder.Create()
>  .WithIdentity("trigger1", "group1")
>  .StartNow()
>  .ModifiedByCalendar("myHoliday")
>  .WithSimpleSchedule(x => x
>        .WithIntervalInSeconds(5)
>        .RepeatForever())
>  .Build();
>
>```
>
>`AddExcludedDate(DateTime Day)`方法：设置跳过哪天不执行。参数：时间（某天）。
>
>`AddCalendar("myHoliday", holidayCalendar, false, false)`方法：将假期规则添加到调度程序。参数：规则名称， 实现ICalendar的类型对象，是否设置 [replace]， 是否更新以存在的`trigger`。

##### `ISimpleTrigger` 时间触发器

>```csharp
>ISimpleTrigger simpleTrigger = (ISimpleTrigger)TriggerBuilder.Create()
>                .WithIdentity("simpleTrigger1", "group1")
>                .StartAt(DateTimeOffset.Now)        // 设置开始时间，如果未设置则为当前时间
>                .WithSimpleSchedule(x => x
>                    .WithIntervalInSeconds(2)          // 10s执行一次
>                    .WithRepeatCount(10))                   // 重复10次
>                .ForJob("job1", "group1")       // 设置任务唯一标识
>                .Build();
>```
>
>`.StartAt(DateTimeOffset time)` 方法：设置开始时间。
>
>`.StartAt(DateBuilder.FutureDate(5, IntervalUnit.Minute))`：设置在未来5分钟内执行一次。
>
>`.EndAt(DateBuilder.DateOf(13, 53, 10))`方法：设置终止时间。
>
>`DateBuilder.FutureDate(int, IntervalUnit)`方法：创建未来日期。参数：偏移量，时间单位。
>
>`DateBuilder.EvenHourDate(null)`方法：获取下一个偶数小时。
>
>`.WithRepeatCount(10)`方法：设置重复10次。
>
>`.ForJob("job1", "group1")`方法：绑定任务。参数 ：任务Key，任务所在组 。或者 `JobDetail` 对象。

#####  `CronTrigger` 表达式触发器

>```csharp
>ITrigger trigger2 = TriggerBuilder.Create()
>.WithIdentity("simpleTrigger1", "group1")
>.WithCronSchedule("0/5 * * * * ?")
>//.WithSchedule(CronScheduleBuilder.DailyAtHourAndMinute(10, 42))
>.Build();
>```
>
>`.WithCronSchedule("0/5 * * * * ?")`方法：使用表达式设置触发时间。
>
>`.WithSchedule(CronScheduleBuilder.DailyAtHourAndMinute(10, 42))`方法： 使用 `CronScheduleBuilder` 构建表达式。
>
>`Corn`表达式 由 `7` 个子表达式组成，使用`white-space`分割，例如：`0 0 12 ? * WED` 表示 `每周三中午12点`。
>
>>格式：
>>
>>| 字段         | 必填 | 值范围           | 可用符号    |
>>| ------------ | ---- | ---------------- | ----------- |
>>| Seconds      | True | 0-59             | , - * /     |
>>| Minutes      | True | 0-59             | , - * /     |
>>| Hours        | True | 0-23             | , - * /     |
>>| Day of month | True | 1-31             | , - * / L W |
>>| Month        | True | 1-12 or JAN-DEC  | , - * /     |
>>| Day of week  | True | 1-7 or SUN-SAT   | , - * / L # |
>>| Year         | True | empty, 1970-2099 | , - * /     |
>
>>1. Seconds [0 - 59]
>>2. Minutes [0 - 59]
>>3. Hours [0 - 23 ]
>>4. Day-of-Month [0 - 31]
>>5. Month [0 - 11]
>>6. Day-of-Week [1- 7] or [SUN, MON, TUE, WED, THU, FRI, SAT]
>>7. Year (可选)
>
>>符号
>
>>1. `*` 通配符，表示所有，例如 `0 0 * * * ?` 表示`每月每天每小时0分0秒执行一次`。
>>2. `/` 表示间隔多长时间执行，例如`0 2/20 * * * ?` 表示`每月每天每小时在2分0秒 22分0秒 42分零秒 各执行一次`。
>>3. `?` 只有 `Day-of-Month` 和 `Day-of-Week` 可以使用。表示无特定值。
>>4. `L` 只有 `Day-of-Month` 和 `Day-of-Week` 可以使用。表示月份或者星期的最后一天。如果在 `Day-of-Month` 和 `Day-of-Week` 中同时使用，例如`Day-of-Month` 为 `*`；`Day-of-Week`为 `6L`，则表示`一个月的最后一个星期五`。
>>5. `W` 离指定日期最近的工作日（周一至周五）。例如给`Day-of-Month`设置为 `15W`，表示`距离该月15日最近的工作日`。
>>6. `#` 用于指定该月的第几周。例如`Day-of-Week` 设置为 `6#3` 表示 `该月第三个星期五`。
>>7. `,` 表示多个参数满足其一。例如 `0 30 10-13 ? * WED,FRI` 表示 `每周三和周五的 10:30、11:30、12:30 和 13:30 触发`。
>>8. `-` 表示一个时间段。例如 `0 0/30 8-9 5,20 * ?` 表示 `每月 5 日和 20 日的上午 8 点和上午 10 点之间每半小时触发一次`。

##### `JobDataMap` 任务数据列表

>用于定义任务时向`IJob` 实现类传递参数

##### `IJob` 任务定义接口

>1. 实现类需实现 `Execute` 方法。参数为 `IJobExecutionContext`上下文。
>2. 实现类必须包含无参构造函数。因为在执行任务前会重新创建对象。
>3. 特性 `Attribute`
>   - `[DisallowConcurrentExecution]`  ：阻止任务并发。约束是基于`JobDetail`,而不是基于当前实例（使用同一个`IJob`实现类创建多个`IJobDetail`并不会限制并发）。
>   - `[PersistJobDataAfterExecution]`： 在`Execute()`执行完毕后告诉`Quartz`更新`JobDetail`内的`JobDataMap`数据。如果使用该特性，需配合 `[DisallowConcurrentExecution]`特性一起使用。否则可能会出现数据未更新。
>   - `[Durability]`：以后不会触发的工作会自动从调度程序删除。
>   - `[RequestsRecovery]`：在调度程序崩溃重启后执行，这时`JobExecutionContext.Recovering` 的返回结果为`True`。

##### `Job Listeners` 任务监听

>实现 `IJobListener` 接口或 `JobListenerSupport` 抽象类。
>
> **监听器内部不能抛出异常，不然会阻塞任务。**
>
>使用：
>
>```csharp
>scheduler.ListenerManager.AddJobListener(new MyJobListener(), KeyMatcher<JobKey>.KeyEquals(new JobKey("job1", "group1")));
>```
>
>成员：
>
>1. `Name` 监听器名称。
>2. `Task JobToBeExecuted(IJobExecutionContext context, CancellationToken cancellationToken = default)` 任务执行前触发。
>3. `Task JobExecutionVetoed(IJobExecutionContext context, JobExecutionException jobException, CancellationToken cancellationToken = default)` 任务结束后触发。
>4. `Task JobExecutionVetoed(IJobExecutionContext context, CancellationToken cancellationToken = default)` 执行被否决了。

##### `Trigger Listeners` 触发器监听

>实现 `ITriggerListener` 接口或 `TriggerListenerSupport` 抽象类。
>
>**监听器内部不能抛出异常，不然会阻塞任务。**
>
>使用：
>
>```csharp
>scheduler.ListenerManager.AddTriggerListener(new MyTriggerListener(), KeyMatcher<TriggerKey>.KeyEquals(new TriggerKey("simpleTrigger1", "group1")));
>```
>
>成员：
>
>1. `Name` 监听器名称。
>2. `Task TriggerFired(ITrigger trigger, IJobExecutionContext context, CancellationToken cancellationToken = default)` 开始执行。
>3. `Task<bool> VetoJobExecution(ITrigger trigger, IJobExecutionContext context, CancellationToken cancellationToken = default)` 是否否决执行。
>4. `Task TriggerComplete(ITrigger trigger, IJobExecutionContext context, SchedulerInstruction triggerInstructionCode, CancellationToken cancellationToken = default)` 执行完毕。
>5. `Task TriggerMisfired(ITrigger trigger, CancellationToken cancellationToken = default)` 任务失火。

##### `Scheduler Listeners` 调度程序监听

>实现 `ISchedulerListener` 接口或 `SchedulerListenerSupport` 抽象类。
>
>**监听器内部不能抛出异常，不然会阻塞任务。**
>
>使用：
>
>```csharp
>scheduler.ListenerManager.AddSchedulerListener(new MySchedulerListener());
>```
>
>成员：
>
>1. `SchedulerStarting()` 调度程序开启时触发。
>2. `SchedulerStarted()` 调度程序开启后触发。
>3. `SchedulerInStandbyMode()` 调度程序在待机状态时触发。
>4. `SchedulerShuttingdown()` 调度程关闭时触发。
>5. `SchedulerShutdown()` 调度程序关闭后时触发。
>6. `SchedulerError()` 调度程序异常时触发。
>7. `SchedulingDataCleared()` 调度程序数据清除后触发。
>8. `JobAdded()` 添加任务时触发。
>9. `JobDeleted()` 移除任务时触发。
>10. `JobInterrupted()` 任务中断时触发。
>11. `JobPaused()` 任务暂停时触发。
>12. `JobResumed()` 任务恢复时触发。
>13. `JobScheduled()` 调度程序预定任务时触发。
>14. `JobUnscheduled()` 调度程序取消预定时触发。
>15. `JobsPaused()` 组任务暂停时触发。
>16. `JobsResumed()` 组任务恢复时触发。
>17. `TriggerFinalized()` 触发器开始执行时触发。
>18. `TriggerPaused()` 触发器暂停时触发。
>19. `TriggerResumed()` 触发器恢复时触发。
>20. `TriggersPaused()` 组触发器暂停时触发。
>21. `TriggersResumed()` 组触发器恢复时触发。

##### `Job Stores` 调度程序数据中心

>###### `RAMJobStore`  默认 `JobStore`
>
>优点：简单，性能高。
>
>缺点：所有数据保存在 `RAM` 中，程序在结束或崩溃时，所有调度信息都将丢失。
>
>配置：`quartz.jobStore.type = Quartz.Simpl.RAMJobStore, Quartz`。
>
>
>
>`AdoJobStore` ADO.NET Job Store
>
>优点：数据保存在数据库中。
>
>缺点：对比 `RAMJobStore` 效率相对较低。
>
>配置：
>
>>要使用 `AdoJobStore`，您必须首先创建一组数据库表供 `Quartz.NET` 使用。您可以在 `Quartz.NET` 发行版的 `database/dbtables` 目录中找到创建表的 SQL 脚本。如果还没有适合您的数据库类型的脚本，只需查看现有脚本之一，并以您的数据库所需的任何方式对其进行修改。需要注意的一件事是，在这些脚本中，所有表都以前缀 `QRTZ_ ` 开头（例如表 `QRTZ_TRIGGERS ` 和 `QRTZ_JOB_DETAIL`）。这个前缀实际上可以是你想要的任何东西，只要你通知 `AdoJobStore ` 前缀是什么（在你的 `Quartz.NET` 属性中）。使用不同的前缀可能有助于在同一数据库中为多个调度程序实例创建多组表。
>>
>>目前，`AdoJobStore` 内部实现的唯一选择是 `JobStoreTX`，它自己创建事务。这与 `Java版本 ` 的 `Quartz `不同，其中还可以选择使用 `J2EE` 容器管理事务的 `JobStoreCMT`。
>>
>>最后一步是设置数据源，`AdoJobStore `可以从中连接到您的数据库。数据源在 `Quartz` 中定义。数据源信息包含连接字符串和 `ADO.NET` 委托信息。
>>
>>配置文件添加：`quartz.jobStore.type = Quartz.Impl.AdoJobStore.JobStoreTX, Quartz`
>>
>>接下来，您需要为 `JobStore ` 选择要使用的 `IDriverDelegate` 实现。 `DriverDelegate ` 负责执行特定数据库可能需要的任何 ` ADO.NET` 工作。`StdAdoDelegate `是一个使用**“普通”**`ADO.NET `代码（和 SQL 语句）来完成其工作的委托。如果没有专门为您的数据库制作的其他委托，尝试使用这种委托 - 特别代表通常有数据库的具体问题更好的性能或解决方法。其他委托可以在 `Quartz.Impl.AdoJobStore ` 命名空间或其子命名空间中找到。
>>
>>注意：
>>
>>>如果您使用默认的 StdAdoDelegate，Quartz.NET 将发出警告，因为当您有很多触发器可供选择时，它的性能很差。特定委托有特殊的 SQL 来限制结果集长度（SqlServerDelegate uses TOP n、PostgreSQLDelegate LIMIT n、OracleDelegate ROWCOUNT () <= n etc.）。
>>
>>选择委托后，将其类名设置为 AdoJobStore 使用的委托。
>>
>>配置文件添加：
>>
>>```ini
>># 配置 Quartz 以使用 DriverDelegate
>>quartz.jobStore.driverDelegateType = Quartz.Impl.AdoJobStore.StdAdoDelegate, Quartz
>># 配置 Quartz 使用表前缀
>>quartz.jobStore.tablePrefix = QRTZ_
>># 配置数据源
>>quartz.jobStore.dataSource = myDS
>># 配置连接字符串
>>quartz.dataSource.myDS.connectionString = Server=localhost;Database=quartz;Uid=quartznet;Pwd=quartznet
>># 配置数据库提供程序
>>quartz.dataSource.myDS.provider = MySql
>>```
>>
>>支持的程序集：
>>
>>- `SqlServer` \- SQL Server driver 。
>>  - 对于完整框架，这是默认的 System.Data.SqlClient （Quartz 3.1 除外）。
>>  - 从 Quartz 3.2 开始，.NET Core 默认为 Microsoft.Data.SqlClient。
>>- `SystemDataSqlClient` - 在 .NET Core 上单独提供（完整框架的默认值）。
>>- `MicrosoftDataSqlClient` - 在完整框架上单独提供（.NET Core 的默认设置）。
>>- `OracleODP` - Oracle's Oracle Driver。
>>- `OracleODPManaged` -  Oracle's managed driver for Oracle 11。
>>- `MySql` - MySQL Connector/.NET。
>>- `SQLite` -  SQLite ADO.NET Provider。
>>- `SQLite-Microsoft` - Microsoft SQLite ADO.NET Provider。
>>- `Firebird` - Firebird ADO.NET Provider。
>>- `Npgsql` - PostgreSQL Npgsql。
>
>配置 AdoJobStore 以使用字符串作为 JobDataMap 值（推荐，它大大降低了类型序列化问题的可能性。）：
>
>```ini
>quartz.jobStore.useProperties = true
>```
>
>选择序列化程序：
>
>```ini
># "json" is alias for "Quartz.Simpl.JsonObjectSerializer, Quartz.Serialization.Json"
>quartz.serializer.type = json
>```

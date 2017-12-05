# Quartz

## 一	什么是quartz?

### (一)	什么是Quartz

`Quartz`是一个具有着丰富的自有特性,开源的定时任务库,它可以被集成在任何的java应用中,从小型的独立应用到大型的电子商务应用都可以被使用.`Quartz`可以用来创建简单的或者复杂的任务调度用于执行即使,几百,甚至成千上万个任务.你可以通过声明标准java组件的方式,让它帮你做任何事情!

### (二)	能够解决什么问题

假如你的应用中有需要在某个特定的时间点执行的任务,或者说你的系统中有重复的维护工作,使用`Quartz`确实是一个不错的方案.

* 工作流的驱动管理(**指定时间点执行的任务**):假设创建了一个新的订单,我们需要准确的在两个小时以后执行一项任务,这项任务的内容是:检查订单的状态,如果该订单还没有接收到确认信息,那么将订单的状态修改为"尚未确认,等待介入",并且出发一次警告通知.

* 系统维护(**重复的工作**):在每个工作日(除节假日以外的所有工作日)的晚上11:30,将数据库的内容转储到XML文件中.

* 在应用中使用定时提醒的服务等.

## 二   使用步骤：

  在spring boot项目中使用为例：
### 必须步骤：
1. 在pom.xml中添加依赖：

   ```
     <dependency>
         <groupId>org.quartz-scheduler</groupId>
         <artifactId>quartz</artifactId>
         <version>2.3.0</version>
     </dependency>
   ```

2. 创建一个`QuartzJob`类，该类必须实现`Job`接口

  覆写`Job`接口中的`execute(JobExecutionContext context)`方法,该类执行具体的任务

  ```java
  package com.common.test.quartz.job;

  import org.quartz.Job;
  import org.quartz.JobExecutionContext;
  import org.quartz.JobExecutionException;

  import java.text.SimpleDateFormat;
  import java.util.Date;

  /**
   * @author yangle
   * @date 2017/11/21
   */
  public class QuartzJob implements Job {
      @Override
      public void execute(JobExecutionContext context) throws JobExecutionException {
          Date now = new Date();
          SimpleDateFormat sdf = new SimpleDateFormat("MM-dd HH:mm:ss");
          System.out.println(sdf.format(now));
      }
  }
  ```

  ​

3. 构建任务，主要的API
  - JobDetail  创建关于QuartzJob的具体实例

    ```java
    JobDetail jobDetail = JobBuilder.newJob(QuartzJob.class)
                    .withIdentity("first", "group-first")
                    .build();
    ```

    ​

  - Trigger  创建一个触发器实例

    ```java
    Trigger trigger = TriggerBuilder.newTrigger()
                    .startNow()
                    .withIdentity("trigger-one", "group-first")
                    .withSchedule(SimpleScheduleBuilder.repeatSecondlyForever(10))
                    .build();
    ```

    ​

4. 创建调度器实例 quartzScheduler

   ```java
    SchedulerFactory factory = new StdSchedulerFactory();
    Scheduler quartzScheduler = factory.getScheduler();
   ```

   或者

   ```java
    Scheduler quartzScheduler = StdSchedulerFactory.getDefaultScheduler();
   ```

5. 绑定任务，执行

   ```java
   quartzScheduler.scheduleJob(jobDetail, trigger);
   ```


### 其他常用的API
- JobExecutionContext
- JobDataMap
- TriggerDataMap
## 三   配置文件 `quartz.properties`

### (一)  配置文件的路径

*   默认的配置文件路径:jar包下org/quartz/quartz.properties


*   自定义的配置文件路径:项目根路径下

### (二) 常用的配置项

```
    #quartz实例的名称,同一集群环境下的instanceName必须一致
    org.quartz.scheduler.instanceName=quartz
    #区别不同的quartz实例
    org.quartz.scheduler.instanceId=AUTO
    #将需要执行的任务持久化存储至数据库
    org.quartz.jobStore.class=org.quartz.impl.jdbcjobstore.JobStoreTX
    #JDBC代理
    org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate
    #标识一套完整的数据库连接配置
    org.quartz.jobStore.dataSource=DB
    #假如使用所在应用的连接池,此处配置对应的jndi
    #org.quartz.dataSource.DB.jndiURL=${jndiName}
    #quartz任务相关表的前缀
    org.quartz.jobStore.tablePrefix=QRTZ_
    #quartz运行在集群环境下
    org.quartz.jobStore.isClustered=true
    #检测集群环境下的各quartz实例的时间间隔
    org.quartz.jobStore.clusterCheckinInterval=20000
```

## 四   quartz与spring boot的整合

在实际的应用中,我们通常需要使用spring中的service或者repository来协助我们实现一些任务功能,但是目前我们如果只是简单的使用`@Autowired` 或者其他相关注解注入实例的话,是会报错的,为解决这个问题,我们需要做一些定制化的事情.

1.  添加依赖

    ```
    <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-context-support</artifactId>
       <version>4.3.10.RELEASE</version>
    </dependency>
    ```

2.  重写一个新的jobFactory

    ```java
    @Component
    public class MyJobFactory extends AdaptableJobFactory{

        @Autowired
        private AutowireCapableBeanFactory capableBeanFactory;

        @Override
        protected Object createJobInstance(TriggerFiredBundle bundle) throws Exception {
            //调用父类的方法
            Object jobInstance = super.createJobInstance(bundle);
            //进行注入
            capableBeanFactory.autowireBean(jobInstance);
            return jobInstance;
        }
    }
    ```

3.  将新的jobFactory配置到quartz.properties中

    ```
    org.quartz.scheduler.jobFactory.class=${MyJobFactory的全路径名}
    ```



## 五  监听器

相关API

*   JobListener
*   TriggerListener
*   SchedulerListener
*   ListenerManager


quartz任务扫描频率



```
bss
{
    "success": true,
    "code": 100000,
    "msg": "处理成功!",
    "data": {
        "count": 1,
        "rows": [
            "192.168.102.190:7209"
        ]
    }
}
```
## 六 一些推荐做法

### （一）Production System Tips

#### skip update check

目前quartz包含有一个”更新检测“的特性，它可以在线检测是否可用的新版本供下载。这样的检查异步执行，并且不会影响quartz的运行和初始化，而且连接服务器失败的情况下，他是不会执行的。

你可以通过在Quartz的属性配置：“org.quartz.scheduler.skipUpdateCheck=true”或者系统属性“org.terracotta.quartz.skipUpdateCheck=true”（which you can set in your system environment or as
a -D on the java command line）来禁用掉这个特性，而且这也是推荐的做法。

### （二）JobDataMap Tips

#### 1. 在JobDataMap中仅存储基本类型的数据类型（包括对应的封装类和java.lang.String）

在JobDataMap存储基本数据类型及对应封装类或者String可以避免出现序列话的错误

#### 2. 使用 Merged JobDataMap

`Merged JobDataMap`可以通过JobExecutionContext的实例获取（getMergedJobDataMap（））方法，它合并了分别存储在JobDetail和Trigger上的JobDataMap,如果两个JobDataMap中存在相同的key值，则取最新的一次更新。

如果你处理的job绑定了多个触发器triggers，并且每个独立的触发器trigger需要给Job提供不同的数据，这样的情况下，将JobDataMap存储在对应的Trigger中可能效果更好。

综合上述情况，推荐的做法是：在Job的execute()方法中，应该使用JobExecutionContext.getMergedJobDataMap()方法获得JobDataMap,而不建议分别从jobDetail和Trigger中获取对应的JobDataMap。














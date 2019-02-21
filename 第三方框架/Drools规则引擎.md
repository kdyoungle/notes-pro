#   Drools规则引擎

## 一. 场景引入

我们经常会碰到类似于根据一个人的考试分数,对这个人的分数进行评级的一类场景,通常情况下我们会使用if{...}else if{...}....else{...},或者switch分支这样的结构来进行处理,一般情况下没毛病,但是很多时候,这样的判断条件不是一成不变的,我们经常甚至是频繁的去修改判断条件,相应的每一次更改都要去编译发布应用,通常还要经过测试的流程,浪费了时间,无法快速的相应需求的变更.而且频繁的修改代码,无形中增加了出错的几率!

仔细观察,会发现:在单单修改了规则的情况下,后面的业务逻辑代码并没有影响,这种情况我们是否可以将规则从代码中剥离出来,单独的进行管理?

规则引擎就是为满足这样的需求而产生的!

## 二. 简介

Drools是由JBOSS公司开源的一套基于java的规则引擎系统;它实现了将业务规则从应用程序代码中分离出来,规则引擎使用特定语法编写业务规则,规则引擎可以接受数据输入,解释业务规则,并根据业务规则作出相应的决策;

### 第一个Hello World实例

案例

| 序号(id) |   成绩(score)    | 评级(level) |
| :------: | :--------------: | :---------: |
|    1     | 90<= score <=100 |  excellent  |
|    2     |  70<= score <90  |    good     |
|    3     |  60<= score <70  |    pass     |
|    4     |    score <60     |   nopass    |

1.  创建maven项目(项目结构)


    注意点:

    *   kmodule.xml与.drl文件的路径:kmodule.xml文件放在resource/META-INF目录下,.drl文件统一放在resource目录下

2.  添加相关依赖

```
	<properties>
        <drools-version>7.6.0.Final</drools-version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.drools</groupId>
            <artifactId>drools-compiler</artifactId>
            <version>${drools-version}</version>
        </dependency>
    </dependencies>
```

3.  相关文件的详细内容

    *   ScoreRecord.java

        ```java
        package com.drools.checkscore.entity;

        public class ScoreRecord {

            /**
             * 分数
             */
            private Float score;
            /**
             * 该分数所处的等级
             */
            private String level;
            // 省略getter,setter及toString()方法
        }

        ```

    *   score.drl

        ```java
        // package 可定义为任意字符串,方便管理与路径一致
        package com.drools.checkscore
        import com.drools.checkscore.entity.ScoreRecord

        rule excellent
            when
                sr : ScoreRecord(score >= 90 && score <= 100)
            then
                sr.setLevel("excellent");
        //        sr.setScore(60.0f);
                update(sr)
            end
        rule good
            when
                sr : ScoreRecord(score >= 70 && score < 90)
            then sr.setLevel("good");
            end
        rule pass
            when
                sr : ScoreRecord(score >= 60 && score < 70)
            then sr.setLevel("pass");
            end
        rule nopass
            when
                sr : ScoreRecord(score < 60)
            then sr.setLevel("nopass");
            end
        ```

    *   kmodule.xml

        ```
        <?xml version="1.0" encoding="UTF-8"?>
        <kmodule xmlns="http://www.drools.org/xsd/kmodule">
            <kbase name="scoreRules" packages="com.drools.checkscore">
                <ksession name="ksession"/>
            </kbase>
        </kmodule>
        ```

    *   RunTest.java

        ```java
        package com.drools.checkscore.run;

        import com.drools.checkscore.entity.ScoreRecord;
        import org.kie.api.KieServices;
        import org.kie.api.runtime.KieContainer;
        import org.kie.api.runtime.KieSession;

        public class RunTest {
            public static void main(String[] args) {
                // 获取kieSession
                KieServices ks = KieServices.Factory.get();
                KieContainer kieContainer = ks.getKieClasspathContainer();
                KieSession kieSession = kieContainer.newKieSession("ksession");
                // 创建测试对象
                ScoreRecord scoreRecord = new ScoreRecord();
                scoreRecord.setScore(90.0f);
                // 调用规则引擎进行判断
                kieSession.insert(scoreRecord);
                int count = kieSession.fireAllRules();
                System.out.println("本次触发了" + count + "条规则!");
                System.out.println("调用规则后: " + scoreRecord);
            }
        }
        ```

    输出结果为: 

    ```
    本次触发了1条规则!
    调用规则后: ScoreRecord{score=90.0, level='excellent'}
    ```

### 优点

1.  实现了业务逻辑与业务规则的分离,业务规则集中管理;
2.  可以动态修改规则,动态发布,快速响应需求变更;
3.  可以降低开发成本,可以让业务人员参与到规则编写,尽可能地降低开发人员的接入;
4.  可代降低业务复杂度,降低开发成本.

### 缺点

1.  使得程序的架构变得复杂,对程序的架构提出了更高的需求;
2.  要学习规则脚本的语法;
3.  某些特殊的场景或者要求,可能会出现不可知的错误.

## 三. workbench的使用

drools提供了workbench平台,便于编写,统一管理规则项目

### workbench安装(以win7系统,tomcat8.5.20为例)

1.  官网下载war包

2.  解压war包后,readme.txt中有详细的安装方式(内容如下:)

    *   (1)下载war包,将war包重命名为"kie-drools-wb.war",放到tomcat的webapps目录下

    *   (2)在tomcat/bin目录下创建文件setenv.bat,内容如下

        ```
        CATALINA_OPTS="-Xmx512M 
                       -Djava.security.auth.login.config=%CATALINA_HOME%/webapps/kie-drools-wb/WEB-INF/classes/login.config 
                       -Dorg.jboss.logging.provider=jdk"
        ```

    *   (3)添加依赖至tomcat/lib目录下

        *   kie-tomcat-integration  (org.kie:kie-tomcat-integration)
        *   JACC  (javax.security.jacc:artifactId=javax.security.jacc-api)
        *   slf4j-api  (org.slf4j:artifactId=slf4j-api)

    *   (4)在tomcat/conf/server.xml文件中`host`节点内末尾添加如下内容

        ```
        <Valve className="org.kie.integration.tomcat.JACCValve" />
        ```

    *   (5)在tomcat/conf/tomcat-user.xml文件中`tomcat-users`节点内添加如下内容

        ```
          <role rolename="admin"/>
          <role rolename="analyst"/>
          <role rolename="HR"/>
          <role rolename="kie-server"/>
          <role rolename="PM"/>
          <role rolename="user"/>
          <user username="kieserver" password="kieserver1!" roles="admin,kie-server"/>
          <user username="admin" password="admin" roles="admin,analyst,PM,HR,kie-server"/>
        ```

    *   (6)安装文件完整内容如下:

        ```
        Installation notes
        ==================
        ```


        1. Define system properties

            create setenv.sh (or setenv.bat) file inside TOMCAT_HOME/bin and add following:

            CATALINA_OPTS="-Xmx512M \
            -Djava.security.auth.login.config=$CATALINA_HOME/webapps/kie-drools-wb/WEB-INF/classes/login.config \
            -Dorg.jboss.logging.provider=jdk"
    
            NOTE: On Debian based systems $CATALINA_HOME needs to be replaced with $CATALINA_BASE. ($CATALINA_HOME defaults to /usr/share/tomcat8 and $CATALINA_BASE defaults to /var/lib/tomcat8/)
            NOTE: this is an example for unix like systems for Windows $CATALINA_HOME needs to be replaced with windows env variable or absolute path
            NOTE: java.security.auth.login.config value includes name of the folder in which application is deployed by default it assumes kie-drools-wb so ensure that matches real installation.
            login.config file can be externalized as well meaning be placed outside of war file.


           *******************************************************************************

        2. Configure JEE security for kie-wb on tomcat (with default realm backed by tomcat-users.xml)

           2a. Copy "kie-tomcat-integration" JAR into TOMCAT_HOME/lib (org.kie:kie-tomcat-integration)
           2b. Copy "JACC" JAR into TOMCAT_HOME/lib (javax.security.jacc:artifactId=javax.security.jacc-api in JBoss Maven Repository)
           2c. Copy "slf4j-api" JAR into TOMCAT_HOME/lib (org.slf4j:artifactId=slf4j-api in JBoss Maven Repository)
           2d. Add valve configuration into TOMCAT_HOME/conf/server.xml inside Host element as last valve definition:
    
              <Valve className="org.kie.integration.tomcat.JACCValve" />
    
           2e. Edit TOMCAT_HOME/conf/tomcat-users.xml to include roles and users, make sure there will be 'analyst' or 'admin' roles defined as it's required to be authorized to use kie-wb
        ```

3. 输入网址: http://127.0.0.1:8080/kie-drools-wb ,登录名:admin,密码:admin,成功登陆则部署成功!

### 使用步骤:

略

### 优点

1.  提供了在线编写规则,编译,发布的功能
2.  使用git进行版本管理,可以将项目clone到本地进行编写,完成后push到远程进行发布
3.  以jar包的形式发布规则,使用方需要将该jar包以项目依赖的方式引入,即可使用

### 缺点

1.  git作为版本管理工具与公司使用的svn不匹配
2.  以jar包的形式引入依赖,在每次修改规则的时候同样需要被使用的系统修改jar的版本以与最新规则保持一致
3.  而且以jar的形式引入的话,完全没有必要使用workbench,完全可以本地开发打包然后发布至maven仓库
4.  使用并不友好,在线编辑有很多的限制条件,自测过程中出现了一些处理解决的错误

## 四. workserver

server为drools的一个组件，drools通过workbench 打包开发部署到server上,server为规则JAR包提供了一个宿主容器，这样其他服务就可以通过http的方式，调用对应规则，这样的好处在于，要调用drools可以不再引用JAR包，并可以通过workbench功能定时扫描，实时部署。其简略的部署图如下:

![server](http://img.blog.csdn.net/20171129145831250?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbjYzMTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 服务搭建



## 五. rules流程控制的属性

设置在rule与when之间

1.  no-loop
2.  lock-on-active
3.  ruleflow-group
4.  salience
5.  agenda-group
6.  auto-focus
7.  activation-group
8.  date-effective
9.  date-expire
10.  enabled
11.  duration(已废弃,被定时器取代)

## 与spring boot集成时遇到的问题:

1.  kieSession为单例模式注入,调用结束后无法将原有内容清空,可以考虑使用StatelessKieSession;

    目前的解决方法是:在规则验证完毕后,从working memory中删除insert的对象


## 计划的方案:

1.  本地开发,打包为jar,发布至公共maven仓库,由需要的额系统进行调用

    优点:

    -   不需要部署workbench与workserver应用,可以立即开展开发工作

    缺点:

    -   jar包修改后,依赖的系统需要重新部署,没有完全达到动态加载规则的目标

2.  workbench与workserver组合

    优点:

    *   实现了动态的加载规则
    *   实时部署,即时性高

    缺点:

    *   需要部署workbench与workserver环境
    *   workbench中编写规则并不友好,需要解决本地开发代码提交的问题
    *   以域名或者地址的方式进行访问
    *   部署规则时,流程略微复杂
    *   规则请求时,参数格式复杂,还需要添加head信息,与目前的使用习惯有出入
    *   需要确保workbench的稳定性,规则内容的存储均在对应应用的目录下,无法多个workbench共享同一套规则

3.  (考虑应用的方案)  在系统内部使用更新规则文件的方式来动态的修改规则:

    可行性:drools规则加载后是运行在内存中,只要可以修改内存中已有的规则就可以实现动态的加载的功能,drools提供了相关的api来实现这样的功能,同时风控系统也有页面的需求,可以通过页面控制规则文件的加载

    优点:

    *   实现了规则的动态加载和修改
    *   通过文件路径的分类来选择性的加载测试或者投入生产应用的规则
    *   系统内部集成规则引擎,响应迅速,不受网络传输速度的影响


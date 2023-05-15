# logback配置详解

## 前言

**SLF4J与其它日志组件调用关系图**

![img](https://pic3.zhimg.com/80/v2-f2240a20242483593a43b1586a485a8e_720w.webp)

SLF4J，即简单日志门面（Simple Logging Facade for Java），不是具体的日志解决方案，它只服务于各种各样的日志系统。

SLF4J最常用的日志实现框架是：log4j、logback。一般有slf4j+log4j、slf4j+log4j2、slf4j+logback三种日志组合。本文选取logback做具体介绍。

## 一、logback介绍

Logback是由log4j创始人设计的另一个开源日志组件,官方网站： [http://logback.qos.ch](https://link.zhihu.com/?target=http%3A//logback.qos.ch)。它当前分为以下三个模块：

- logback-core：其它两个模块的基础模块。
- logback-classic：它是log4j的一个改良版本，同时它完整实现了slf4j API使你可以很方便地更换成其它日志系统如log4j或JDK14 Logging。
- logback-access：访问模块与Servlet容器集成提供通过Http来访问日志的功能。

默认情况下，Spring Boot会用Logback来记录日志，并用INFO级别输出到控制台。

日志级别（log level）：用来控制日志信息的输出，从高到低分为共分为七个等级:

> A：off 最高等级，用于关闭所有日志记录。
> B：fatal 指出每个严重的错误事件将会导致应用程序的退出。
> C：error 指出虽然发生错误事件，但仍然不影响系统的继续运行。
> D：warm 表明会出现潜在的错误情形。
> E：info 一般和在粗粒度级别上，强调应用程序的运行全程。
> F：debug 一般用于细粒度级别上，对调试应用程序非常有帮助。
> G：all 最低等级，用于打开所有日志记录。

## 二、pom依赖与logback.xml配置

**1.pom依赖**

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>

<!--引入以上依赖，会自动引入以下jar
	logback-classic.x.x.x.jar
	logback-core.x.x.x.jar
	slf4j-api-x.x.x.jar
-->
```

> 注意spring-boot-starter-parent里已集成logback，可直接使用。

**2.logback.xml**

在工程resources目录下建立logback.xml

1.logback首先会试着查找logback.groovy文件;

2.当没有找到时，继续试着查找logback-test.xml文件;

3.当没有找到时，继续试着查找logback.xml文件;

4.如果仍然没有找到，则使用默认配置（打印到控制台）。

## 三、configuration 标签

configuration节点主要包含appdender、logger、root三个标签，如下图：

![img](https://pic2.zhimg.com/80/v2-3a63449003b15abe06a9cf28a1d6baf1_720w.webp)

```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration scan="true" scanPeriod="60 seconds" debug="false" packagingData="true">

	<!-- 用来设置上下文名称，每个logger都关联到logger上下文，默认上下文名称为default。但可以使用<contextName>设置成其他名字，
	用于区分不同应用程序的记录。一旦设置，不能修改。-->
    <contextName>myApplicationName</contextName>


	<!--用来定义变量值，它有两个属性name和value，通过<property>定义的值会被插入到logger上下文中，可以使用“${}”来使用变量。
	name: 变量的名称，value: 的值时变量定义的值-->
    <property name="LOG_HOME" value="${catalina.base}/logs/cloudTest/" />
    
    
	<!--获取时间戳字符串，他有两个属性key和datePattern     key: 标识此<timestamp> 的名字；datePattern: 设置将当前时间（解析配置
	文件的时间）转换为字符串的模式，遵循	java.txt.SimpleDateFormat的格式。这个属性很少使用	-->
    <timestamp key="keyValue" datePattern="yyyy-MM-dd" />    
    <contextName>${keyValue}</contextName>

	<!--其他配置略-->
</configuration>
```

- **configuration**

scan: 当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。

scanPeriod: 设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。

debug: 当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

- **contextName**
- **property**
- **timestamp**

## 四、logger、root 标签

- **logger** 是configuration的子节点，用来设置某一个包或者具体的某一个类的日志打印级别、以及指定appender。logger仅有一个name属性，一个可选的level和一个可选的additivity属性。

name: 用来指定受此logger约束的某一个包或者具体的某一个类。

level: 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，还有一个特殊值INHERITED或者同义词NULL，代表强制执行上级的级别。如果未设置此属性，那么当前logger将会继承上级的级别。

additivity: 是否向上级logger传递打印信息。默认是true。

logger可以包含零个或多个appender-ref元素，标识这个appender将会添加到这个logger。

- **root** 也是logger元素，但是它是根logger。只有一个level属性，因为已经被命名为"root".

level: 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，不能设置为INHERITED或者同义词NULL。默认是DEBUG。

root可以包含零个或多个appender-ref元素，标识这个appender将会添加到这个logger。

## **五、additivity 标签案例说明**

*java代码：*

```java
package com.z7.springcloud.service;
public class WebSocketServer {
    private static final Logger logger = LoggerFactory.getLogger(WebSocketServer.class);
    //直接写到name=STDOUT的日志中去
    //private Logger logger = LoggerFactory.getLogger("STDOUT");
    public static void main(String[] args) {
        logger.trace("======trace");
        logger.debug("======debug");
        logger.info("======info");
        logger.warn("======warn");
        logger.error("======error");
    }
}
```

*logback.xml配置文件*

### **1.只配置root**

```xml
 <configuration scan="true" scanPeriod="60 seconds" debug="false">

    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level [%logger{15}:%line] - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- 日志输出级别 -->
    <root level="info">
        <appender-ref ref="STDOUT" />
    </root>

</configuration>
```

其中appender的配置表示打印到控制台；

<root level="INFO">将root的打印级别设置为“INFO”，指定了名字为“STDOUT”的appender。当执行WebSocketServer类时，root将级别为“INFO”及大于“INFO”的日志信息交给已经配置好的名为“STDOUT”的appender处理，“STDOUT”appender将信息打印到控制台；

打印结果如下：

```java
2022-03-04 14:38:51.566 [main] INFO com.z7.springcloud.service.WebSocketServer - ======info
2022-03-04 14:38:51.566 [main] WARN com.z7.springcloud.service.WebSocketServer - ======warn
2022-03-04 14:38:51.566 [main] ERROR com.z7.springcloud.service.WebSocketServer - ======error
```

### **2.带有logger的配置，不指定级别，不指定appender**

```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false">

    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level [%logger{15}:%line] - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <logger name="com.z7.springcloud.service" ></logger>

    <!-- 日志输出级别 -->
    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>

</configuration>
```

<logger name="com.z7.springcloud.service"/>将com.z7.springcloud.service包下的所有类的日志打印，但是并没设置打印级别，所以继承他的上级的日志级别“DEBUG”；没有设置additivity，默认为true，将此logger的打印信息向上级传递；没有设置appender，此logger本身不打印任何信息。

<root level="DEBUG">将root的打印级别设置为“DEBUG”，指定了名字为“STDOUT”的appender。

当执行com.z7.springcloud.service.WebSocketServer类时，因为WebSocketServer在包com.z7.springcloud.service中，所以首先执行<logger name="com.z7.springcloud.service" />，将级别为“DEBUG”及大于“DEBUG”的日志信息传递给root，本身并不打印；

root接到下级传递的信息，交给已经配置好的名为“STDOUT”的appender处理，“STDOUT”appender将信息打印到控制台；

打印结果如下：

```java
2022-03-04 14:38:51.562 [main] DEBUG com.z7.springcloud.service.WebSocketServer - ======debug
2022-03-04 14:38:51.566 [main] INFO com.z7.springcloud.service.WebSocketServer - ======info
2022-03-04 14:38:51.566 [main] WARN com.z7.springcloud.service.WebSocketServer - ======warn
2022-03-04 14:38:51.566 [main] ERROR com.z7.springcloud.service.WebSocketServer - ======error
```

### **3.带有多个logger的配置，指定级别，指定appender**

```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false">

    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level [%logger{15}:%line] - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <logger name="com.z7.springcloud.service" ></logger>

    <logger name="com.z7.springcloud.service.WebSocketServer" level="info" additivity="false">
        <appender-ref ref="STDOUT" />
    </logger>

    <!-- 日志输出级别 -->
    <root level="error">
        <appender-ref ref="STDOUT" />
    </root>

</configuration>
```

<logger name="com.z7.springcloud.service" />将控制com.z7.springcloud.service包下的所有类的日志的打印，但是并没用设置打印级别，所以继承他的上级的日志级别“error”；没有设置additivity，默认为true，将此logger的打印信息向上级传递；没有设置appender，此logger本身不打印任何信息。

<logger name="com.z7.springcloud.service.WebSocketServer" level="INFO" additivity="false">控制com.z7.springcloud.service.WebSocketServer类的日志打印，打印级别为“INFO”；additivity属性为false，表示此logger的打印信息不再向上级传递，指定了名字为“STDOUT”的appender。

<root level="ERROR">将root的打印级别设置为“ERROR”，指定了名字为“STDOUT”的appender。

当执行com.z7.springcloud.service.WebSocketServer类时，先执行<logger name="com.z7.springcloud.service.WebSocketServer" level="INFO" additivity="false">，将级别为“INFO”及大于“INFO”的日志信息交给此logger指定的名为“STDOUT”的appender处理，在控制台中打出日志，不再向logger的上级 <logger name="com.z7.springcloud.service"/> 传递打印信息； 注意此时因为WebSocketServer位于com.z7.springcloud.service包下，所以 <logger name="com.z7.springcloud.service"/>是<logger name="com.z7.springcloud.service.WebSocketServer" level="INFO" additivity="false">的上级！

<logger name="com.z7.springcloud.service"/>未接到任何打印信息，当然也不会给它的上级root传递任何打印信息；

打印结果：

```java
2022-03-04 14:38:51.566 [main] INFO com.z7.springcloud.service.WebSocketServer - ======info
2022-03-04 14:38:23.474 [main] WARN com.z7.springcloud.service.WebSocketServer - ======warn
2022-03-04 14:38:51.562 [main] ERROR com.z7.springcloud.service.WebSocketServer - ======error
```

如果将<logger name="com.z7.springcloud.service.WebSocketServer" level="INFO" additivity="false">修改为 <logger name="com.z7.springcloud.service.WebSocketServer" level="INFO" additivity="true">那打印结果将是什么呢？

没错，日志打印了两次，想必大家都知道原因了，因为打印信息向上级传递，logger本身打印一次， <logger name="com.z7.springcloud.service"/>接到后本身因为没有设置appender不会打印但向上继续传递给root，root会再打印一次。

```java
2022-03-04 14:38:51.566 [main] INFO com.z7.springcloud.service.WebSocketServer - ======info
2022-03-04 14:38:51.566 [main] INFO com.z7.springcloud.service.WebSocketServer - ======info
2022-03-04 14:38:23.474 [main] WARN com.z7.springcloud.service.WebSocketServer - ======warn
2022-03-04 14:38:23.474 [main] WARN com.z7.springcloud.service.WebSocketServer - ======warn
2022-03-04 14:38:51.562 [main] ERROR com.z7.springcloud.service.WebSocketServer - ======error
2022-03-04 14:38:51.562 [main] ERROR com.z7.springcloud.service.WebSocketServer - ======error
```

此处可能令人困惑，`<root>`日志级别不是error么，怎么还会打印info和warn级别的日志？这里是因为日志信息向上传递后，日志级别会由下级的level来决定！所以即使info日志也会打印！！ 因此注意**一般实际使用中additivity常常设置为false**。

## 六、appender 具体属性

appender有两个必要属性name和class，name指定appender名称，class指定appender的全限定名。另一个属性encoder：负责两件事，一是把日志信息转换成字节数组，二是把字节数组写入到输出流。目前PatternLayoutEncoder 是唯一有用的且默认的encoder ，有一个节点，用来设置日志的输入格式，使用“%”加“转换符”方式。

例如：

```xml
  <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <pattern>%yellow(%d{yyyy-MM-dd HH:mm:ss}) %red([%thread]) %highlight(%-5level) %cyan(%logger{50}) - %magenta(%msg) %n</pattern>
      <charset>UTF-8</charset>
 </encoder>
```

> %d表示日期
> %thread表示线程名
> %-5level表示级别从左显示5个字符宽度
> %msg是日志消息
> %n是换行符
> 如果要输出“%”则必须用“\”对“%”进行转义

下面介绍几种常用的appender。

### **1、RollingFileAppender**

滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。这个是最常用的！

有以下子节点：

<file>：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。

<filter>: 表示过滤器，用法稍后讲解。

<append>：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。

<encoder>：对记录日志进行格式化。

<rollingPolicy>: 当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名。

<triggeringPolicy >: 告知 RollingFileAppender 何时激活滚动。

<prudent>：当为true时，不支持FixedWindowRollingPolicy。支持TimeBasedRollingPolicy，但是有两个限制，1.不支持也不允许文件压缩，2.不能设置file属性，必须留空

### **2、SizeAndTimeBasedRollingPolicy（根据时间和文件大小的滚动策略）**

最常用的滚动策略，根据时间再根据文件大小来滚动生成文件，例如

```xml
    <!--当前项目的目录下-->
    <property name="LOG_HOME" value="logs/cloud" />
    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">

        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!-- rollover daily -->
            <fileNamePattern>${LOG_HOME}/mylog-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!-- 每个文件最多5MB，保存60天的历史记录，但最多20GB -->
            <maxFileSize>5MB</maxFileSize>
            <maxHistory>60</maxHistory>
            <totalSizeCap>20GB</totalSizeCap>
        </rollingPolicy>

        <encoder>
            <pattern>%d{yyyy-MM-dd'T'HH:mm:ss.SSSXXX} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
```

结果：（此图maxFileSize设置为5KB的结果，值比较小会有稍许偏差）

![img](https://pic4.zhimg.com/80/v2-3614029eacfbca3772121b7e437f349b_720w.webp)

### **3、 TimeBasedRollingPolicy（根据时间的滚动策略）**

根据时间的滚动策略，既负责滚动也负责触发滚动。有以下子节点：

- **fileNamePattern** 必要节点，文件名必须包含“%d”转换符， “%d”可以包含一个 java.text.SimpleDateFormat指定的时间格式，如：%d{yyyy-MM},如果直接使用 %d，默认格式是 yyyy-MM-dd。 RollingFileAppender 的file子节点可有可无，通过设置file，可以为活动文件和归档文件指定不同位置，当前日志总是记录到file指定的文件（活动文件），活动文件的名字不会改变；如果没设置file，活动文件的名字会根据fileNamePattern 的值，每隔一段时间改变一次。“/”或者“\”会被当做目录分隔符。
- **maxHistory** 可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每个月滚动，且 <maxHistory>是6，则只保存最近6个月的文件，删除之前的旧文件。注意，删除旧文件时，那些为了归档而创建的目录也会被删除。

例：每天生成一个日志文件，保存30天的

```xml
     <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>d://log/business.log.%d.log</FileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>

        <encoder>
            <pattern>%d{yyyy-MM-dd'T'HH:mm:ss.SSSXXX} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <logger name="com.z7.springcloud.service" level="INFO" additivity="false">
        <appender-ref ref="file" />
    </logger>
```

### **4、FixedWindowRollingPolicy（固定窗口的滚动策略）**

根据固定窗口算法重命名文件的滚动策略。有以下子节点：

- **minIndex**: 窗口索引最小值
- **maxIndex**: 窗口索引最大值，当用户指定的窗口过大时，会自动将窗口设置为12。

fileNamePattern :必须包含“%i”例如，假设最小值和最大值分别为1和2，命名模式为 mylog%i.log,会产生归档文件mylog1.log和mylog2.log。还可以指定文件压缩选项，例如，mylog%i.log.gz 或者 没有log%i.log.zip。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>test.log</file>

    <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
      <fileNamePattern>tests.%i.log.zip</fileNamePattern>
      <minIndex>1</minIndex>
      <maxIndex>3</maxIndex>
    </rollingPolicy>

    <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
      <maxFileSize>5MB</maxFileSize>
    </triggeringPolicy>

    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
  </appender>
        
  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```

### **5、triggeringPolicy（触发策略）**

如果当前活动文件的大小超过指定大小会触发当前活动文件滚动。只有一个节点:`<maxFileSize>`:当前活动日志文件的大小，默认值是10MB。

```xml
  <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
        <MaxFileSize>25MB</MaxFileSize>
  </triggeringPolicy>
```



### **6、 ConsoleAppender**

把日志输出到控制台，有以下子节点：

`<encoder>`：对日志进行格式化,上面已介绍；

`<target>`：字符串 System.out 或者 System.err ，默认 System.out ；

例如：

```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false">

    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!--格式化输出：%d表示日期，%thread表示线程名，%highlight()高亮显示，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %highlight(%-5level) [%logger{15}:%line] - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- 日志输出级别 -->
    <root level="error">
        <appender-ref ref="STDOUT" />
    </root>

</configuration>
```

### **7、FileAppender**

把日志添加到文件，有以下子节点：

<file>：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。

<append>：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。

<encoder>：对记录事件进行格式化。

<prudent>：如果是 true，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是 false。

```xml
<configuration>  

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">  
    <file>testFile.log</file>  
    <append>true</append>  
    <encoder>  
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>  
    </encoder>  
  </appender>  
          
  <root level="DEBUG">  
    <appender-ref ref="FILE" />  
  </root>  
</configuration>  
```

## **七、filter**

过滤器，执行一个过滤器会有返回个[枚举](https://link.zhihu.com/?target=https%3A//so.csdn.net/so/search%3Fq%3D%E6%9E%9A%E4%B8%BE%26spm%3D1001.2101.3001.7020)值，即DENY，NEUTRAL，ACCEPT其中之一。

> 返回DENY，日志将立即被抛弃不再经过其他过滤器； 返回NEUTRAL，有序列表里的下个过滤器过接着处理日志； 返回ACCEPT，日志会被立即处理，不再经过剩余过滤器。

过滤器被添加到<Appender> 中，为<Appender> 添加一个或多个过滤器后，可以用任意条件对日志进行过滤。<Appender> 有多个过滤器时，按照配置顺序执行。

下面是几个常用的过滤器：

### **1、LevelFilter**

- 级别过滤器，根据日志级别进行过滤。如果日志级别等于配置级别，过滤器会根据onMath 和 onMismatch接收或拒绝日志。有以下子节点：

<level>:设置过滤级别

<onMatch>:用于配置符合过滤条件的操作

<onMismatch>:用于配置不符合过滤条件的操作

例如：将过滤器的日志级别配置为INFO，所有INFO级别的日志交给appender处理，非INFO级别的日志，被过滤掉。

```xml
<configuration>   
  <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">   
    <filter class="ch.qos.logback.classic.filter.LevelFilter">   
      <level>INFO</level>   
      <onMatch>ACCEPT</onMatch>   
      <onMismatch>DENY</onMismatch>   
    </filter>   

    <encoder>   
      <pattern>   
        %-4relative [%thread] %-5level %logger{30} - %msg%n   
      </pattern>   
    </encoder>   
  </appender>   

  <root level="DEBUG">   
    <appender-ref ref="CONSOLE" />   
  </root>   
</configuration>  
```

### **2、ThresholdFilter**

- 临界值过滤器，过滤掉低于指定临界值的日志。当日志级别等于或高于临界值时，过滤器返回NEUTRAL；当日志级别低于临界值时，日志会被拒绝。

例如：过滤掉所有低于INFO级别的日志。

```xml
<configuration>   
  <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">   
    <!-- 过滤掉 TRACE 和 DEBUG 级别的日志-->   
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">   
      <level>INFO</level>   
    </filter>   

    <encoder>   
      <pattern>   
        %-4relative [%thread] %-5level %logger{30} - %msg%n   
      </pattern>   
    </encoder>  
  </appender>   

  <root level="DEBUG">   
    <appender-ref ref="CONSOLE" />   
  </root>   
</configuration>  
```

### **3、EvaluatorFilter**

- 求值过滤器，评估、鉴别日志是否符合指定条件。需要额外的两个JAR包，commons-compiler.jar和janino.jar有以下子节点：

**<evaluator>**

鉴别器，常用的鉴别器是JaninoEventEvaluato，也是默认的鉴别器，它以任意的java布尔值表达式作为求值条件，求值条件在配置文件解释过成功被动态编译，布尔值表达式返回true就表示符合过滤条件。evaluator有个子标签<expression>，用于配置求值条件。

求值表达式作用于当前日志，logback向求值表达式暴露日志的各种字段：

![img](https://pic4.zhimg.com/80/v2-7c73ce3896a282bdda95b8fdd703e747_720w.webp)

<onMatch>:用于配置符合过滤条件的操作

<onMismatch>:用于配置不符合过滤条件的操作

例如：过滤掉所有日志消息中不包含“billing”字符串的日志。

```xml
<configuration>   
   <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">   
    <filter class="ch.qos.logback.core.filter.EvaluatorFilter">         
      <evaluator> <!-- 默认为 ch.qos.logback.classic.boolex.JaninoEventEvaluator -->   
        <expression>return message.contains("billing");</expression>   
      </evaluator>   
      <OnMatch>ACCEPT </OnMatch>  
      <OnMismatch>DENY</OnMismatch>  
    </filter>   
    <encoder>   
      <pattern>   
        %-4relative [%thread] %-5level %logger - %msg%n   
      </pattern>   
    </encoder>   
  </appender>   
   
  <root level="INFO">   
    <appender-ref ref="STDOUT" />   
  </root>   
</configuration>  
```

**<matcher> ：**

匹配器，尽管可以使用String类的matches()方法进行模式匹配，但会导致每次调用过滤器时都会创建一个新的Pattern对象，为了消除这种开销，可以预定义一个或多个matcher对象，定以后就可以在求值表达式中重复引用。<matcher>是<evaluator>的子标签。

<matcher>中包含两个子标签，一个是<name>，用于定义matcher的名字，求值表达式中使用这个名字来引用matcher；另一个是<regex>，用于配置匹配条件。

```xml
<configuration debug="true">   
   
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">   
    <filter class="ch.qos.logback.core.filter.EvaluatorFilter">   
      <evaluator>           
        <matcher>   
          <Name>odd</Name>   
          <!-- filter out odd numbered statements -->   
          <regex>statement [13579]</regex>   
        </matcher>   
           
        <expression>odd.matches(formattedMessage)</expression>   
      </evaluator>   
      <OnMismatch>NEUTRAL</OnMismatch>   
      <OnMatch>DENY</OnMatch>   
    </filter>   
    <encoder>   
      <pattern>%-4relative [%thread] %-5level %logger - %msg%n</pattern>   
    </encoder>   
  </appender>   
   
  <root level="DEBUG">   
    <appender-ref ref="STDOUT" />   
  </root>   
</configuration>  
```

## 八、完整配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--监测配置文件是否有修改的时间间隔:60秒-->
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!--log日志目录-->
    <property name="logging.path" value="logs/cloud"/>
    <contextName>z7-spring-cloud</contextName>

    <!-- 控制台日志 -->
    <appender name="consoleLog" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出（配色）：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%yellow(%d{yyyy-MM-dd HH:mm:ss}) %red([%thread]) %highlight(%-5level) %cyan(%logger{50}) - %magenta(%msg) %n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!--根据日志级别分离日志，分别输出到不同的文件-->
    <!--appender用来格式化日志输出节点，有俩个属性name和class，class用来指定哪种输出策略，常用就是控制台输出策略和文件输出策略。-->
    <appender name="fileInfoLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>DENY</onMatch>
            <onMismatch>ACCEPT</onMismatch>
        </filter>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>
                %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{50} - %msg%n
            </pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!--滚动策略-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--按时间保存日志 修改格式可以按小时、按天、月来保存-->
            <fileNamePattern>${logging.path}/cloud-test-6010.info.%d{yyyy-MM-dd}.log</fileNamePattern>
            <!--保存时长-->
            <MaxHistory>90</MaxHistory>
            <!--文件大小-->
            <totalSizeCap>500MB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <!--ERROR-->
    <appender name="fileErrorLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <encoder>
            <pattern>
                %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{50} - %msg%n
            </pattern>
        </encoder>
        <!--滚动策略-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--路径-->
            <fileNamePattern>${logging.path}/cloud-test-6010.error.%d{yyyy-MM-dd}.log</fileNamePattern>
            <MaxHistory>90</MaxHistory>
        </rollingPolicy>
    </appender>

    <!-- root子节点;name:需要打印日志的包;level：设置传递日志级别;additivity：是否向根目录传递，默认值true -->
    <logger name="com.z7.springcloud.service" level="INFO" additivity="false">
        <!-- 将日志记录到fileErrorLog中的文件中 -->
        <appender-ref ref="fileErrorLog"/>
        <!-- 将日志打印到控制台 -->
        <appender-ref ref="consoleLog"/>
    </logger>

    <root level="info">
        <!-- 打印到控制台 -->
        <appender-ref ref="consoleLog"/>
        <appender-ref ref="fileInfoLog"/>
        <appender-ref ref="fileErrorLog"/>
    </root>
</configuration>
```

## 九、总结

1、输出源选择

logback的配置，需要配置输出源appender，打日志的logger（子节点）和root（根节点），实际上，它输出日志是从子节点开始，子节点如果有输出源直接输入，如果无，判断配置的additivity，是否向上级传递，即是否向root传递，传递则采用root的输出源，否则不输出日志。

2、日志级别Level

日志记录器（Logger）的行为是分等级的： 分为OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL或者您定义的级别。root、logger默认级别是DEBUG。
Log4j建议只使用四个级别，优先级从高到低分别是 ERROR、WARN、INFO、DEBUG，优先级高的将被打印出来。（logback通用），通过定义级别，可以作为应用程序中相应级别的日志信息的开关。

3、案例
比如在程序中定义了INFO级别，则应用程序中所有DEBUG级别的日志信息将不被打印出来。（设置INFO级别，即：日志等级>=INFO 生效）
项目上生产环境的时候一定得把debug的日志级别重新调为warn或者更高，避免产生大量日志。
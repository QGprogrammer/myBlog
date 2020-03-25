#  log4j2配置

*疑问: 项目打成war包后由tomcat运行 那File日志存在哪儿呢?阿里云上接入的是什么日志?*

*日志怎么备份?*

#### 常见输出参数

```xml
%c{参数} 或 %logger{参数}  ##输出日志名称
%C{参数} 或 %class{参数    ##输出类型
%d{参数}{时区te{参数}{时区} ##输出时间
%F|%file                  ##输出文件名
highlight{pattern}{style} ##高亮显示
%l  ##输出错误的完整位置
%L  ##输出错误行号
%m 或 %msg 或 %message ##输出错误信息
%M 或 %method ##输出方法名
%n            ##输出换行符
%level{参数1}{参数2}{参数3} ##输出日志的级别
%t 或 %thread              ##创建logging事件的线程名
```



#### 日志级别优先级

**OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->**

* ***FATAL*** ： 级别比较高了。重大错误，这种级别你可以直接停止程序了，是不应该出现的错误么！不用那么紧张，其实就是一个程度的问题。
*  ***error***： 错误信息。用的也比较多。
* ***warn***： 有些信息不是错误信息，但是也要给程序员的一些提示，类似于eclipse中代码的验证不是有error 和warn（不算错误但是也请注意，比如以下depressed的方法）。
*  ***info***： 输出一下你感兴趣的或者重要的信息，这个用的最多了。
* ***debug***： 调试么，我一般就只用这个作为最低级别，trace压根不用。是在没办法就用eclipse或者idea的debug功能就好了么。

* ***TRACE***： 是追踪，就是程序推进以下，你就可以写个trace输出，所以trace应该会特别多，不过没关系，我们可以设置最低日志级别不让他输出。



#### 过滤器匹配机制

Filters决定日志事件能否被输出。过滤条件有三个值：***ACCEPT(接受), DENY(拒绝) or NEUTRAL(中立)***

>ACCEP和DENY比较好理解就是接受和拒绝的意思，在使用单个过滤器的时候，一般就是使用这两个值。但是在组合过滤器中，如果用接受ACCEPT的话，日志信息就会直接写入日志文件,后续的过滤器不再进行过滤。所以，在组合过滤器中，接受使用NEUTRAL（中立），被第一个过滤器接受的日志信息，会继续用后面的过滤器进行过滤，只有符合所有过滤器条件的日志信息，才会被最终写入日志文件。



#### Lookups

##### Date

>与其他lookup不同，它不是通过key去查找值，而是通过SimpleDateFormat验证格式是否有效，然后记录当前时间

```xml
<RollingFile name="Rolling-${map:type}" fileName="${filename}" filePattern="target/rolling1/test1-$${date:MM-dd-yyyy}.%i.log.gz">
  <PatternLayout>
    <pattern>%d %p %c{1.} [%t] %m%n</pattern>
  </PatternLayout>
  <SizeBasedTriggeringPolicy size="500" />
</RollingFile>
```

##### Context Map Lookup: 如记录loginId

```xml
<File name="Application" fileName="application.log">
  <PatternLayout>
    <pattern>%d %p %c{1.} [%t] $${ctx:loginId} %m%n</pattern>
  </PatternLayout>
</File>
```

##### Environment Lookup：记录系统环境变量

```xml
<File name="Application" fileName="application.log">
  <PatternLayout>
    <pattern>%d %p %c{1.} [%t] $${env:USER} %m%n</pattern>
  </PatternLayout>
</File>
```

##### System Properties Lookup

```xml
<Appenders>
  <File name="ApplicationLog" fileName="${sys:logPath}/app.log"/>
</Appenders>
```



#### Example



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--Configuration后面的status,这个用于设置log4j2自身内部的信息输出,可以不设置,当设置成trace时,你会看到log4j2内部各种详细输出-->
<!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身,设置间隔秒数-->
<configuration status="error" monitorInterval="3600">
    <Properties>
        <!-- 输出格式 -->
        <Property name="patternLayout">%d{DEFAULT} [%-5level] %class{36}-%method:%line -%n %msg%n%xEx</Property>
    </Properties>

    <!-- 附加器 输出器 需要被引用 -->
    <appenders>
        <!-- 控制台的日志输出，开发环境使用 -->
        <Console name="console" target="SYSTEM_OUT"><!-- 默认为SYSTEM_ERR -->
            <Filters>
                <!-- 控制在debug以上级别 -->
                <ThresholdFilter level="DEBUG" onMatch="ACCEPT" onMismatch="DENY"/>
                <!-- 不处理rewrite标记的日志 -->
                <!-- Marker可用new Log4jMarker(MarkerManager.getMarker("xx")) 创建 -->
                <MarkerFilter marker="rewrite" onMatch="DENY" onMismatch="ACCEPT"/>
            </Filters>
            <PatternLayout pattern="${patternLayout}"/>
        </Console>

        <!-- 滚动日志 -->
        <!-- fileName 输出到项目目录下logs文件夹中 -->
        <!-- filePattern 归档压缩的格式 -->
        <RollingFile name="wechatFile" fileName="logs/wechat_error.log"
        filePattern="logs/log/$${date:yyyy-MM}/wechat-error-%d{MM-dd-yyyy}-%i.log.gz">
            <Filters>
                <!-- wechat 标记的错误以上级别 -->
                <ThresholdFilter level="ERROR" onMatch="NEUTRAL" onMismatch="DENY"/>
                <MarkerFilter marker="wechat" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
            <PatternLayout pattern="${patternLayout}"/>
            <SizeBasedTriggeringPolicy size="10MB"/>
    <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20 -->
            <DefaultRolloverStrategy max="20"/>
        </RollingFile>

        <!-- 文件形式的日志输出（每次启动时覆盖写入文件），开发或测试使用 -->
        <!-- 使用web lookup可以指定文件存放到应用部署的根路径下。注意$表示初始化时一次性取值，$$表示动态决定值 -->
        <!-- append 决定重启项目时是追加还是新建日志文件 保存在打包后的文件中 -->
        <File name="logfile" fileName="${log4j:configParentLocation}/static/log/jiandan-log.html" append="false">
            <Filters>
                <ThresholdFilter level="ERROR" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
            <!-- 输出的是html -->
            <HTMLLayout title="my-log" charset="UTF-8"/>
        </File>
        <!-- 这个会打印出所有的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
        <RollingFile name="rollingFile" fileName="logs/error.log"
                     filePattern="logs/log/$${date:yyyy-MM}/error-%d{MM-dd-yyyy}-%i.log.gz">
            <Filters>
                <ThresholdFilter level="ERROR" onMatch="ACCEPT" onMismatch="DENY"/><!-- 控制在INFO以上级别 -->
            </Filters>
            <PatternLayout pattern="${patternLayout}"/>
            <SizeBasedTriggeringPolicy size="20MB"/>
            <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20 -->
            <DefaultRolloverStrategy max="20"/>
        </RollingFile>
    </appenders>

    <!-- 日志集合 -->
    <loggers>
        <!-- 根日志集合 -->
        <root level="TRACE">
            <AppenderRef ref="console"/>
            <AppenderRef ref="logfile"/>
            <AppenderRef ref="rollingFile"/>
        </root>
        
        <!-- wechat 引用了两个输出器 -->
        <Logger name="wechat" level="WARN">
            <AppenderRef ref="console"/>
            <AppenderRef ref="wechatFile"/>
        </Logger>

        <!-- log4j提供的用于输出MDC日志的Logger。 EventLogger.logEvent方法的输出内容将使用该日志方案  -->
        <Logger name="EventLogger" level="DEBUG" additivity="false">
            <AppenderRef ref="console"/>
            <AppenderRef ref="logfile"/>
        </Logger>

        <!-- additivity="false" 表示收集上来的日志不会再汇到root中 意味着不会再重复输出 -->
        <Logger name="org.springframework" level="INFO" additivity="false">
            <AppenderRef ref="console"/>
            <AppenderRef ref="logfile"/>
        </Logger>

        <!-- 如果希望查看Mybatis扫描了哪些bean，可以放开该日志 -->
        <Logger name="org.apache.ibatis.io.ResolverUtil" level="ERROR" additivity="false">
            <AppenderRef ref="console"/>
            <AppenderRef ref="logfile"/>
        </Logger>

        <Logger name="org.apache.commons.beanutils" level="ERROR" additivity="false">
            <AppenderRef ref="console"/>
            <AppenderRef ref="logfile"/>
        </Logger>

        <!-- 立方日志 -->
        <Logger name="com.nettymq.server" level="ERROR" additivity="false">
            <AppenderRef ref="console"/>
            <AppenderRef ref="logfile"/>
        </Logger>

        <!-- netty日志 -->
        <Logger name="io.netty" level="INFO" additivity="false">
            <AppenderRef ref="console"/>
            <AppenderRef ref="logfile"/>
        </Logger>

        <!-- lettuce日志 -->
        <Logger name="io.lettuce" level="INFO" additivity="false">
            <AppenderRef ref="console"/>
            <AppenderRef ref="logfile"/>
        </Logger>

        <!--log4j2 自带过滤日志-->
        <Logger name="org.apache.catalina.startup.DigesterFactory" level="error" />
        <Logger name="org.apache.catalina.util.LifecycleBase" level="error" />
        <Logger name="org.apache.coyote.http11.Http11NioProtocol" level="warn" />
        <logger name="org.apache.sshd.common.util.SecurityUtils" level="warn"/>
        <Logger name="org.apache.tomcat.util.net.NioSelectorPool" level="warn" />
        <Logger name="org.crsh.plugin" level="warn" />
        <logger name="org.crsh.ssh" level="warn"/>
        <Logger name="org.eclipse.jetty.util.component.AbstractLifeCycle" level="error" />
        <Logger name="org.hibernate.validator.internal.util.Version" level="warn" />
        <logger name="org.springframework.boot.actuate.autoconfigure.CrshAutoConfiguration" level="warn"/>
        <logger name="org.springframework.boot.actuate.endpoint.jmx" level="warn"/>
        <logger name="org.thymeleaf" level="warn"/>
    </loggers>
</configuration>
```
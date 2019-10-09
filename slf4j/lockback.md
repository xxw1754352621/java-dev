# [logback日志配置关键类](https://logback.qos.ch/manual/introduction.html)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--配置文件如果发生改变，将会被重新加载 -->
<!-- scanPeriod设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。-->
<!--debug当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。-->
<configuration scan="true" scanPeriod="10 seconds" debug="false">
    <!-- 日志根路径 -->
    <property name="basePath" value="/usr/local/logs/pds"/>
    <springProperty scope="context" name="appName" source="spring.application.name" />
     <!-- 是否异步打印日志 -->
    <springProperty scope="context" name="pdsLogAsync" source="pds.log.async" />

    
	<!--控制台输出 -->
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
		    <pattern>%date{yyyy-MM-dd HH:mm:ss.SSS} %highlight(%-5level) %yellow([${appName:-},%X{X-B3-TraceId:-},%X{X-B3-SpanId:-},%X{X-Span-Export:-}]) [%thread] %cyan(%logger{56}.%method:%L) -%msg%n</pattern>
		</encoder>
	</appender>

	<!-- 文件输出 -->
	<appender name="FILE.LOG.ASYNC.false" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${basePath}/${fpxLogPath:-${appName:-tmp}}/${fpxLogModule:-info}.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${basePath}/${fpxLogPath:-${appName:-tmp}}/${fpxLogModule:-info}.%d{yyMMdd}.%i.log.gz</fileNamePattern>
            <maxHistory>${fpxLogMaxHistory:-30}</maxHistory>
            <totalSizeCap>${fpxLogTotalSize:-1GB}</totalSizeCap>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>${fpxLogMaxSize:-50mb}</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        
		<!-- 日志输出格式 -->
		<encoder>
            <immediateFlush>${fpxLogImmediateFlush:-true}</immediateFlush>
			<pattern>%date{yyyy-MM-dd HH:mm:ss.SSS} %-5level[%thread]%logger{56}.%method:%L -%msg%n</pattern>
		</encoder>
	</appender>

     <!--文件异步日志-->
    <appender name="FILE.LOG.ASYNC.true" class="ch.qos.logback.classic.AsyncAppender">
        <discardingThreshold>0</discardingThreshold>
        <queueSize>1024</queueSize>
        <neverBlock>true</neverBlock>
        <appender-ref ref="FILE.LOG.ASYNC.false"/>
    </appender>
    
 
    <root level="ERROR">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE.LOG.ASYNC.${pdsLogAsync:-false}" />
    </root>

</included>
```

## 关键组件分析

### 1.RollingFileAppender实现类，日志输出配置，一般日志追加到文件尾

```properties
 * <code>RollingFileAppender</code> extends {@link FileAppender} to backup the
 * log files depending on {@link RollingPolicy} and {@link TriggeringPolicy}.

 * For more information about this appender, please refer to the online manual
 * at http://logback.qos.ch/manual/appenders.html#RollingFileAppender
```

### 2.TimeBasedRollingPolicy实现类，文件滚动设置，每天，每周滚动，或者自定义滚动规则

```properties
 * <code>TimeBasedRollingPolicy</code> is both easy to configure and quite
 * powerful. It allows the roll over to be made based on time. It is possible to
 * specify that the roll over occur once per day, per week or per month.
 * 
 * <p>For more information, please refer to the online manual at
 * http://logback.qos.ch/manual/appenders.html#TimeBasedRollingPolicy
```

### 3.TriggeringPolicy接口，文件滚动触发设置

```properties
 * A <code>TriggeringPolicy</code> controls the conditions under which roll-over
 * occurs. Such conditions include time of day, file size, an 
 * external event, the log request or a combination thereof.
```


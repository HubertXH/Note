###### Logback文档地址：
https://logback.qos.ch/manual/index.html

###### 示例：
```
<?xml version="1.0" encoding="gbk"?>
<configuration debug="true" scan="true" scanPeriod="30 seconds">
    <property name="logDir" value="/data/joblog/pimWebAdmin/"/>
    <property name="log.maxFileSize" value="3GB"/>
    <property name="log.maxHistory" value="150"/>
    <property name="log.queueSize" value="10240"/>
    <property name="log.pattern"
              value="%date{yyyy-MM-dd HH:mm:ss} [%thread] %-5level [%mdc{logId}] %logger{80} - %msg%n"/>

    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%date{yyyy-MM-dd HH:mm:ss} [%thread] %-5level [%mdc{logId}] %logger{80} - %msg%n
            </pattern>
        </layout>
    </appender>

    <appender name="infolog"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${logDir}/info/log.%d{yyyy-MM-dd.HH}.%i.log.gz
            </fileNamePattern>
            <maxHistory>${log.maxHistory}</maxHistory>
			<timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
				<maxFileSize>${log.maxFileSize}</maxFileSize>
			</timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
    </appender>
    <appender name="infolog_ASYNC" class="ch.qos.logback.classic.AsyncAppender">
        <includeCallerData>true</includeCallerData>
        <appender-ref ref="infolog"/>
        <neverBlock>true</neverBlock>
        <queueSize>${log.queueSize}</queueSize>
    </appender>

    <appender name="debuglog"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--        <Encoding>GBK</Encoding>-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>DEBUG</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${logDir}/debug/log.%d{yyyy-MM-dd.HH}.log.gz
            </fileNamePattern>
            <maxHistory>${log.maxHistory}</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
    </appender>
    <appender name="debuglog_ASYNC" class="ch.qos.logback.classic.AsyncAppender">
        <includeCallerData>true</includeCallerData>
        <appender-ref ref="debuglog"/>
        <neverBlock>true</neverBlock>
        <queueSize>${log.queueSize}</queueSize>
    </appender>

    <appender name="errorlog"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--        <Encoding>GBK</Encoding>-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${logDir}/error/log.%d{yyyy-MM-dd.HH}.log.gz
            </fileNamePattern>
            <maxHistory>${log.maxHistory}</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
    </appender>
    <appender name="errorlog_ASYNC" class="ch.qos.logback.classic.AsyncAppender">
        <includeCallerData>true</includeCallerData>
        <appender-ref ref="errorlog"/>
        <neverBlock>true</neverBlock>
        <queueSize>${log.queueSize}</queueSize>
    </appender>

    <appender name="externallog"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--        <Encoding>GBK</Encoding>-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${logDir}/external/log.%d{yyyy-MM-dd.HH}.%i.log.gz
            </fileNamePattern>
            <maxHistory>${log.maxHistory}</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>${log.maxFileSize}</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
    </appender>
    <appender name="externallog_ASYNC" class="ch.qos.logback.classic.AsyncAppender">
        <includeCallerData>true</includeCallerData>
        <appender-ref ref="externallog"/>
        <neverBlock>true</neverBlock>
        <queueSize>${log.queueSize}</queueSize>
    </appender>

    <root level="DEBUG">
        <appender-ref ref="infolog_ASYNC"/>
        <appender-ref ref="errorlog_ASYNC"/>
        <appender-ref ref="debuglog_ASYNC"/>
        <appender-ref ref="stdout"/>
    </root>

    <logger name="com.dangdang.pim.external" level="INFO">
        <appender-ref ref="externallog_ASYNC"/>
    </logger>

    <logger name="org.apache" level="ERROR"/>

    <logger name="io.lettuce.core" level="ERROR"/>
</configuration>

```

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<configuration scan="false" scanPeriod="60000" debug="false">

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger - %msg%n</pattern>
        </layout>
    </appender>

    <logger name="java"/>

     <root level="debug">
         <appender-ref ref="STDOUT" />
     </root>

</configuration>
```

![](/assets/logback.png)

![](/assets/logback.configuration.png)

appender的属性

```
1.name指定&lt;appender&gt;的名称



2.class指定&lt;appender&gt;的全限定名
```




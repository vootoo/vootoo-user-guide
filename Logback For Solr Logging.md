# 使用 Logback

## solr.xml
solr.xml 增加如下配置

```xml
<solr>
  <!-- ... -->
  <logging>
    <str name="class">org.vootoo.logging.logback.LogbackWatcher</str>
    <bool name="enabled">true</bool>
    <!-- default watcher param -->
    <watcher>
      <int name="size">50</int>
      <str name="threshold">WARN</str>
    </watcher>
  </logging>
  <!-- ... -->
</solr>
```

有了 LogbackWatcher 才可以 http://localhost:8983/solr/#/~logging 输出和控制。

## vootoo-logging-logback-XXX.jar

solr-5.1.0/server/lib/ext 目录下把 log4j 的相关 jar 移除。加入 logback 相关 jar。

原来 lib/ext 的 log4j

```
# tree ext
ext
├── jcl-over-slf4j-1.7.7.jar
├── jul-to-slf4j-1.7.7.jar
├── log4j-1.2.17.jar
├── slf4j-api-1.7.7.jar
└── slf4j-log4j12-1.7.7.jar
```

现在 lib/ext 的 logback

```
# tree ext
ext
├── jcl-over-slf4j-1.7.7.jar
├── jul-to-slf4j-1.7.7.jar
├── log4j-over-slf4j-1.7.7.jar
├── logback-classic-1.1.3.jar
├── logback-core-1.1.3.jar
└── slf4j-api-1.7.7.jar
```

在 solr.home 目录的 lib 下加入 vootoo-logging-logback-0.3.0.jar

## logback 配置

还要在 server/resources 上加入 logback.xml 如：

```xml
<configuration scan="true">

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{collection} %X{shard} %X{replica} %X{core}] %-5level %logger{36}\(%L\) - %msg%n</pattern>
        </encoder>
    </appender>

      <appender name="SOLR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/solr.log</file>
        <append>true</append>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
          <fileNamePattern>logs/solr.log.%d{yyyy-MM-dd}</fileNamePattern>
          <maxHistory>7</maxHistory>
        </rollingPolicy>

        <encoder>
          <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{collection} %X{shard} %X{replica} %X{core}] %-5level %logger{36}\(%L\) - %msg%n</pattern>
        </encoder>
      </appender>

    <logger level="WARN" name="org.apache.hadoop" />
    <logger level="WARN" name="org.apache.zookeeper" />

    <root level="INFO">
        <appender-ref ref="SOLR" />
    </root>
</configuration>
```


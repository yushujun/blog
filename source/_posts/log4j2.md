---
title: log4j2教程
e: 2018-08-05 17:49:32
tags:
---

# 1.介绍日志框架
日志框架的作用就是输出日志，区别于System.out.print日志框架的有点是异步、解耦，通过框架可以控制日志输出的级别如ERROR、INFO、WARN、DEBUG，也可以控制日志输出的目的地如控制台、本地文件、远程主机等，同时代码中使用System.out.print也不符合规范，日志是需要保留下来用于调试排查问题使用的。
# 2.什么是Log4j2?
Log4j(Logging for Java)是apache实现的一个开源日志组件，logback同样是由log4j的作者设计完成的，拥有更好的特性，用来取代log4j的一个日志框架，[log4j2](http://logging.apache.org/log4j/2.x/manual/configuration.html)是log4j 1.x和logback的改进版，采用了无锁、异步等技术使得日志的吞吐量、性能比log4j 1.x提高10倍，并解决了一些死锁的bug,并且配置更加简单灵活。我们一般不直接使用log4j2而是使用slf4j(Simple Logging Facade for Java),slf4j是对所有日志框架制定的一种规范、标准、接口，并不是一个框架的具体的实现，因为接口并不能独立使用，需要和具体的日志框架实现配合使用（如log4j、logback）。
# 3.log4j2 + slf4j
pom.xml
```xml
<dependencies>
    <!-- slf4j依赖 -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.10</version>
    </dependency>

    <!-- 桥接:告诉slf4j使用Log4j2 -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-slf4j-impl</artifactId>
        <version>2.2</version>
    </dependency>

    <!-- log4j2核心包 -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.7</version>
    </dependency>

    <!-- log4j2 api -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.7</version>
    </dependency>

    <!— 异步日志依赖框架 —>
    <dependency>
        <groupId>com.lmax</groupId>
        <artifactId>disruptor</artifactId>
        <version>3.0.0</version>
    </dependency>
</dependencies>
```
log4j2配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <properties>
        <property name="FILE_NAME">test_log</property>
        <property name="LOG_LEVEL">info</property>
        <property name=“TEST_PATTERN”>%msg%n</property>
        <property name=“PROD_PATTERN”>%d{yyyy/MM/dd HH:mm:ss.SSS} %t [%p] %c{1} (%F:%L) %l %msg%n</property>
    </properties>

    <Appenders>
        <Console name="LocalConsole" target="SYSTEM_OUT">
            <PatternLayout pattern=“${TEST_PATTERN}" />
        </Console>

        <Console name=“ProdConsole” target=“SYSTEM_OUT”>
            <PatternLayout pattern=“${PROD_PATTERN}” />
        </Console>

        <File name="FileAppender" fileName="out.log">
            <PatternLayout pattern=“{PROD_PATTERN}" />
        </File>

        <RollingRandomAccessFile name="RollingRandomAccessFile" fileName="${LOG_HOME}/${FILE_NAME}.log" filePattern="${LOG_HOME}/$${date:yyyy-MM}/${FILE_NAME}-%d{yyyy-MM-dd HH-mm}-%i.log">
            <PatternLayout pattern=“${PROD_PATTERN}"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1"/>
                <SizeBasedTriggeringPolicy size="10 MB"/>
            </Policies>
            <DefaultRolloverStrategy max="20"/>
        </RollingRandomAccessFile>
        <Async name="AsyncAppender">
            <AppenderRef ref="FileAppender" />
        </Async>
    </Appenders>

    <Loggers>
        <AsyncRoot level="info">
            <AppenderRef ref=“ProdConsole" />
            <AppenderRef ref="RollingRandomAccessFile" />
        </AsyncRoot>

          <!— additivity 不在根logger上打印 —>
        <AsyncLogger name="com.ysj.test" level=“${LOG_LEVEL}" additivity="false">
            <AppenderRef ref=“LocalConsole" />
            <AppenderRef ref="FileAppender" />
        </AsyncLogger>
    </Loggers>
</Configuration>
```
Log4j2Test.java
```java
Log4j2Test.java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Log4j2Test {
  private final static Logger logger = LoggerFactory.getLogger(Log4j2Test.class);

  public static void main(String[] args) {
    long beginTime = System.currentTimeMillis();

    for (int i = 0; i < 100000; i++) {
     logger.trace("trace level");
     logger.debug("debug level");
     logger.info("info level");
     logger.warn("warn level");
     logger.error("error level");
    }

    try {
      Thread.sleep(1000 * 6);
    } catch (InterruptedException e) {

    }
    logger.info("请求处理结束，耗时:{}毫秒", (System.currentTimeMillis() - beginTime));
    logger.info("请求处理结束，耗时: " + (System.currentTimeMillis() - beginTime) + "毫秒");
  }
}
```

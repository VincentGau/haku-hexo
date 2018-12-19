---
title: Spring Boot使用logback记录日志到数据库
tags:
- logback
- database
categories:
- Java
- Programming
---
Logback 是在Java社区中被广泛使用的日志框架，前身是Log4j。
Logback 由三个模块组成：logback-core， logback-classic， logback-access；其中logback-core 是另外两个模块的基础； 

## 添加Maven依赖
首先需要在项目中添加maven依赖，这里除了添加logback-core和 logback-classic，还需要slf4j-api，slf4j 的全称是`Simple Logging Façade for java`, 作为各种日志框架的抽象，具体的日志实现还是由logback-classic完成；
```xml
<!--logger begin-->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <version>1.2.3</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>
<!--logger end-->
```

{% blockquote %}
**_NOTE_**: 
slf4j-api 1.8以上版本使用了ServiceLoader机制，依赖static binder机制的早期版本不再被支持；注意保持版本间的兼容性，如果出现
`java.lang.ClassNotFoundException: org.slf4j.impl.StaticLoggerBinder`
错误，请调整依赖的版本；
{% endblockquote %}

## logback 配置文件
依赖添加之后，在`src\main\resource`目录下创建`logback.xml`配置文件；
```xml
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>
 
  <root level="debug">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

## logger创建与调用
logback.xml配置之后，创建logger对象并在需要的方法中调用logger.info() 等方法就会生成日志信息；
{% codeblock lang:java %}
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;


public class Example {
 
    private static final Logger logger 
      = LoggerFactory.getLogger(Example.class);
 
    public static void main(String[] args) {
        logger.info("Example log from {}", Example.class.getSimpleName());
    }
}
{% endcodeblock %}
到此，基本的logback配置已经完成，可以在合适的位置输出日志。

{% blockquote %}
**_NOTE_**: 如要使用spring扩展profile支持，logback配置文件需命名为logback-spring.xml，使用springProfile 节点定义dev， prod等环境；
{% endblockquote %}

## 记录日志到数据库
logback提供了各种Appender，如需要将日志写入数据库，可以使用logback提供的DBAppender，前提是需要创建三个数据库表，表结构固定，日志信息将被写入按约定创建好的数据表；建表SQL语句可以在`ch.qos.logback.classis.db`的 `script` 目录下找到；
{% asset_img db-script.png %} 

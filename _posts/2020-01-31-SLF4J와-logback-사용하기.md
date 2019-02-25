---
layout: post
title: "Spring 프로젝트에 SLF4J와 logback 적용하기"
date: 2019-01-31
excerpt: "logback에 대한 설명부터 프로젝트에 적용하기 까지"
tags: [BaseCamp, rookie 6기, SLF4J, logback]
comments: true
---

# SLF4J와 logback 사용하기


### logback이란?
 프로젝트를 위해 개발을 하다보면 중간중간에 로그를 남겨야할 필요를 느낄때가 있다. 디버깅을 위해서라던지, 제대로 동작했는지 확인을 하기 위해서 혹은 문제가 생겼을때 관련된 정보를 보기 위해서 말이다. 나 같은 초보 개발자 라면 일단 언어에서 제공하는 표준 출력(System.out.println 등..)을 사용해서 로그를 찍으려고 할것이다. 하지만 표준 출력함수를 통해 로그를 남기는 것은 상당한 오버헤드를 가지고 있다.
 그래서 예전에 java에서는 apache에서 개발한 Log4j라는 로깅 유틸리티를 이용해 로그를 남겼다. 하지만 지금은 Log4j도 오래된 것이 되었고, 주로 logbackack이나 log4j2를 사용한다고 한다. (참고 - [Readons to prefer logback over log4j](https://logback.qos.ch/reasonsToSwitch.html))



* * *

### SLF4J란?
 Simple Logging Facade for Java (SLF4J)는 이름에서 알수 있듯이, 자바에서 사용되는 수많은 로깅 라이브러리를 같은 방식으로 사용할 수 있도록  Facade Pattern을 활용하여 구현된 인터페이스다. 이번 포스팅에서도 slf4j를 적용하여 언제든지 로깅 라이브러리를 바꿀수 있게 된다.
 
 

* * *

### 프로젝트에 SLF4J와 logback 적용하는 방법
(Maven web project를 기준으로 설명합니다.)

1. Maven POM에 logback dependency를 추가한다.

```xml
(pom.xml)
    ...
    <dependency>
    	<groupId>ch.qos.logback</groupId>
    	<artifactId>logback-classic</artifactId>
    </dependency>
```


2. XML파일을 통해 로깅에 관련된 필요한 설정을 한다.(생략 가능) 

 
```xml
(src/main/resource/logback.xml)

   <?xml version="1.0" encoding="UTF-8"?>
    <configuration>
    
      <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
          <Pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</Pattern>
        </layout>
      </appender>
      
      <logger name="com.base22" level="TRACE"/>
      
    
      <root level="debug">
        <appender-ref ref="STDOUT" />
      </root>
    </configuration>
```

설정을 통해 로그에 찍히는 형식도 지정할 수 있고, 다른 파일을 통해 로그를 따로 저장할 수도 있다. level을 설정해서 log를 출력할 level 스코프를 지정할 수도 있다.

3. 원하는 곳에 logging 코드를 넣는다.

먼저 관련 클래스를 import한다.
```java
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
```
로거 팩토리를 통해 로거를 생성한다.

```java
    static final Logger logger = LoggerFactory.getLogger(MyClassName.class);
```

로거를 통해 로그를 남긴다.

```java
    logger.debug("debug!");
    logger.info("info!!");
    logger.warn("warn!!!");
    logger.error("error!!!!");
```

4. 실행.
실행을 통해 잘 동작하는지 확인한다.
---
layout: post
title: "Spring boot 앱에서 mysql timeout 문제"
date: 2019-02-22
excerpt: "mysql timeout 문제와 해결방법"
tags: [BaseCamp, rookie 6기, Spring, mysql]
comments: true
---



# Spring boot 앱에서 mysql timeout 문제

spring boot로 개발을 하던 도중 다음과 같은 에러를 보는 경우가 있다.

```java
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: 
The last packet successfully received from the server was 34,510,841 
milliseconds ago.  The last packet sent successfully to the server was 
34,510,841 milliseconds ago. is longer than the server configured value of 
'wait_timeout'. You should consider either expiring and/or testing 
connection validity before use in your application, increasing the server 
configured values for client timeouts, or using the Connector/J connection 
property 'autoReconnect=true' to avoid this problem.
```
## 원인

  찾아보니 원인은 이러했다. 
 Spring boot에서 DB Connection을 관리하기 위해 Connection pool 방식을 사용한다. 그래서 pool에 등록되어 있는 Connection이 재사용되는 방식이다. 그런데 mysql을 사용할 경우, 설정값에 따라 일정 시간동안 사용되지 않는 Connection을 mysql이 자동으로 끊어주고 있었다.  그래서 보통 다음날이 되어 아침에 연결을 시도해면 다음과같은 에러가 떴던 것이다. (이때 설정되는 mysql의 wait_timeout의 기본값은 28800(8시간) 이다.)
  에러를 보면 몇가지 해결책도 제시해 주고 있다. 그런데 mysql의 설정을 변경하는것을 Connection pool의 리소스가 증가하게 될것이다. 그래서 나는 Spring boot의 설정을 통해 일정 시간마다 DB 연결을 확인하는 방식으로 해결하는 방법을 사용하였다.

&nbsp;


## 해결 방법

### Connection pool 설정
Spring에서는 효율적인 DB 연결을 위해 object pool pattern을 활용해 DB Connection을 관리한다.(참고 https://en.wikipedia.org/wiki/Object_pool_pattern, https://en.wikipedia.org/wiki/Connection_pool)

Spring boot 공식 문서를 확인하면 Connection pool 설정 및 기본값을 확인 할 수 있다.
![springboot-ref-doc](/assets/img/springboot-ref-doc-29.1.2.png)

#### 먼저, pom.xml에 다음과 같이 추가해 준다.
```xml
  <dependencies>
  ...
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
  <dependencies>
```
(참고 : 나의 경우 mybatis를 사용하는데, mybatis-sping-boot-starter 설정에 이미 추가가 되어있어 추가해줄 필요가 없었다.)

#### 다음으로, application.properties에 다음과 같이 추가해 준다.
```
...
# 지정한 시간마다 evictor가 pool을 돌면서 connection이 유효한지 검사하고,
# 유효하지 않은 connection을 pool에서 제거한다.(기본값 -1, 이 경우 evictor가 동작하지 않는다.)
spring.datasource.tomcat.time-between-eviction-runs-millis=3600000
# Connection test를 할 때 던지는 쿼리 (mysql의 경우 'SELECT 1' 권장)
spring.datasource.tomcat.validation-query=SELECT 1
# Connection이 idle한 상태에서도 테스트 한다. (기본값 false)
spring.datasource.tomcat.test-while-idle=true
```

#### 여기서 주의할 점은 자신이 사용할 Connection Pool의 종류에 따라 설정을 해줘야 한다는 점이다. ( 나는 기본값인 tomcat JDBC connection pool로 설정해주었다.)

Spring boot의 기본(Default) Connection pool에 대해 알고 싶다면, 자신의 버전에 맞는 공식 레퍼런스를 참고하면 된다. 나의 경우 1.5.19버전으로 개발중이므로 아래 문서를 참고했다.
( Spring boot 1.5.19 레퍼런스 문서 - 29.1.2 Connection to a production database: https://docs.spring.io/spring-boot/docs/1.5.19.RELEASE/reference/htmlsingle/#boot-features-connect-to-production-database)



구글링 해본 결과에 의하면, 기본 ConnectionPool로 원래 Tomcat JDBC Connection Pool이  쓰이다가,  2.0.0대 이후부터 HikariCP 로 바뀌었다고 한다. 

추가적으로 Tomcat Connection Pool에 대한 설정을 바꾸고 싶다면 다음 문서를 참고하면 좋을 것 같다. 
(https://tomcat.apache.org/tomcat-8.0-doc/jdbc-pool.html#Common_Attributes)

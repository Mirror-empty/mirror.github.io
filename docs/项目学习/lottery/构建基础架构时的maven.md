# 构建基础架构时的maven

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.itedus.lottery</groupId>
    <artifactId>Lottery</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>

    <modules>
        <module>lottery-application</module>
        <module>lottery-domain</module>
        <module>lottery-infrastructure</module>
        <module>lottery-interfaces</module>
        <module>lottery-rpc</module>
        <module>lottery-common</module>
    </modules>

    <properties>
        <!-- Base -->
        <jdk.version>1.8</jdk.version>
        <sourceEncoding>UTF-8</sourceEncoding>
        <!-- Spring -->
        <spring.version>4.3.24.RELEASE</spring.version>
        <servlet-api.version>2.5</servlet-api.version>
        <spring.redis.version>1.8.4.RELEASE</spring.redis.version>
        <!-- DB：mysql、mybatis-->
        <mysql.version>8.0.22</mysql.version>
        <mybatis.version>3.3.0</mybatis.version>
        <mybatis_spring.version>1.2.3</mybatis_spring.version>
        <!-- JSON -->
        <fastjson.version>1.2.60</fastjson.version>
        <jackson.version>2.5.4</jackson.version>
        <!-- Junit -->
        <junit.version>4.12</junit.version>
        <!-- Common -->
        <commons-dbcp2.version>2.6.0</commons-dbcp2.version>
        <commons-lang3.version>3.8.1</commons-lang3.version>
        <!-- 日志 -->
        <slf4j.version>1.7.7</slf4j.version>
        <logback.version>1.0.9</logback.version>
        <!-- 其他服务 -->
        <dubbo.version>2.6.6</dubbo.version>
        <zookeeper.version>3.4.14</zookeeper.version>
        <netty.version>4.1.36.Final</netty.version>
        <redis.version>2.9.0</redis.version>
        <scheduler.version>2.3.2</scheduler.version>
    </properties>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!-- 移除嵌入式tomcat插件 -->
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.22</version>
        </dependency>
        <!-- test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- 添加servlet依赖模块 -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <scope>provided</scope>
            <version>4.0.0</version>
        </dependency>
        <!-- 添加jstl标签库依赖模块 -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <!--添加tomcat依赖模块.-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </dependency>
        <!-- 使用jsp引擎，springboot内置tomcat没有此依赖 -->
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
            <version>9.0.2</version>
        </dependency>
        <!-- fastjson -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.58</version>
        </dependency>
        <dependency>
            <groupId>dom4j</groupId>
            <artifactId>dom4j</artifactId>
            <version>1.6.1</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.8</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.thoughtworks.xstream/xstream -->
        <dependency>
            <groupId>com.thoughtworks.xstream</groupId>
            <artifactId>xstream</artifactId>
            <version>1.4.10</version>
        </dependency>

    </dependencies>

    <build>
        <finalName>Lottery</finalName>
        <plugins>
            <!-- 编译plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>${jdk.version}</source>
                    <target>${jdk.version}</target>
                    <compilerVersion>1.8</compilerVersion>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

```
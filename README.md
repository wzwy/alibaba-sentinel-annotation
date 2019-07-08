# alibaba-sentinel-annotation

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

	<!--springboot最终打的jar包-->
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.wz</groupId>
	<artifactId>alibaba-sentinel-annotation</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>alibaba-sentinel-annotation</name>
	<description>Spring Boot</description>

	<!--springboot父jar包-->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.5.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<!--jar版本-->
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<springcloud.version>Finchley.SR1</springcloud.version>
		<springcloud.nacos.version>0.2.2.RELEASE</springcloud.nacos.version>
	</properties>

	<dependencies>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<!--springmvc-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<!--测试-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<!--自动编译-->
		<!--<dependency>-->
			<!--<groupId>org.springframework.boot</groupId>-->
			<!--<artifactId>spring-boot-devtools</artifactId>-->
			<!--<optional>true</optional>-->
		<!--</dependency>-->

		<!--sentinel-->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
		</dependency>

		<!--lombok-->
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<version>1.18.2</version>
			<optional>true</optional>
		</dependency>

	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${springcloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
			<!--spring cloud alibaba的版本，由于spring cloud alibaba还未纳入spring cloud的主版本管理中，所以需要自己加入-->
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-alibaba-dependencies</artifactId>
				<version>${springcloud.nacos.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<!--maven打包-->
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>


</project>


#表示项目的端口，默认端口8080
server:
  port: 8001
spring:
  application:
    name: alibaba-sentinel-annotation #表示服务名称
  cloud:
    sentinel:
      transport:
        dashboard: 127.0.0.1:8080 #表示Sentinel限流和熔断的server地址
management:
  endpoints:
    web:
      exposure:
        include: '*' #表示健康检查开放全部
        
        package com.wz.springboot;

import com.alibaba.csp.sentinel.annotation.aspectj.SentinelResourceAspect;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;;
@SpringBootApplication//springboot主启动类
public class SpringbootApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringbootApplication.class, args);
	}

	//开启配置访问资源文件注解式限流
	@Bean
	public SentinelResourceAspect sentinelResourceAspect(){
	    return new SentinelResourceAspect();
    }

}


package com.wz.springboot;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Slf4j
@Service
public class SentinelAnnotationService {

    //配置访问资源文件注解式限流，value名称、blockHandler是如果限流指定执行的方法
    @SentinelResource(value = "doSomeThing",blockHandler = "exceptionHandler")
    public void doSomeThing(String str){
        log.info(str);
    }
    //限流返回此方法
    public void exceptionHandler(String str, BlockException ex){
        log.error("blockHandler:"+str,ex);
    }

    //配置访问资源文件注解式熔断，value名称，fallback是如果熔断执行的方法
    @SentinelResource(value = "doSomeThingException",fallback = "fallbackHandler")
    public void doSomeThingException(String str){
        log.info(str);
        throw new RuntimeException("发生异常");
    }

    public void fallbackHandler(String str){
        log.error("fallbackHandler:"+str);
    }

}


package com.wz.springboot;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Date;

@RestController
public class SentinelAnnotationController {

    @Autowired
    private SentinelAnnotationService sentinelAnnotationService;

    @GetMapping("/hello")
    public String hello() {
        sentinelAnnotationService.doSomeThing("hello " + new Date());
        return "didispace.com";
    }

    @GetMapping("/helloException")
    public String helloException() {
        sentinelAnnotationService.doSomeThingException("helloException " + new Date());
        return "didispace.com";
    }

}



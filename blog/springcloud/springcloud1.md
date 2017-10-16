# EUREKA的学习使用
## 搭建服务注册中心
* 首先创建一个springboot工程,命名为cloud_eureka_server
* pom文件导入相关依赖

		<parent>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-parent</artifactId>
			<version>1.5.2.RELEASE</version>
		</parent>
	
		<properties>
			<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
			<java.version>1.8</java.version>
		</properties>
	
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-starter-eureka-server</artifactId>
			</dependency>
	
		</dependencies>	
	
		<dependencyManagement>
			<dependencies>
				<dependency>
					<groupId>org.springframework.cloud</groupId>
					<artifactId>spring-cloud-dependencies</artifactId>
					<version>Dalston.SR3</version>
					<type>pom</type>
					<scope>import</scope>
				</dependency>
			</dependencies>
		</dependencyManagement>


* 配置文件application.yml
		server: 
		  port: 8081
		eureka: 
		  client: 
		    registerWithEureka: false #是否注册到其他节点
		    fetchRegistry: false  #是否同步其他节点数据
		    serviceUrl:
		      defaultZone: http://localhost:8081/eureka/
		  instance: 
		    hostname: localhost
	通过配置registerWithEureka和fetchRegistry来表明这是一个eureka server


* 应用程序入口RegisterApp

		@SpringBootApplication
		@EnableEurekaServer//只需要通过添加这个注解让springboot帮你自动启动为注册中心
		public class RegisterApp {
		
			public static void main(String[] args) {
				SpringApplication.run(RegisterApp.class, args);
			}
		
		}


* 启动后访问  http://localhost:8081/  页面如下
![](/img/0000.png)
Instances currently registered with Eureka中没有内容说明还没有服务注册


## 注册服务提供者
* 创建一个叫cloud-eureka工程,命名为cloud-service-hi
* pom文件导入相关依赖
		<parent>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-parent</artifactId>
			<version>1.5.2.RELEASE</version>
		</parent>
	
		<properties>
			<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
			<java.version>1.8</java.version>
		</properties>
	
		<dependencies>

			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-starter-eureka</artifactId>
			</dependency>
	
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-web</artifactId>
			</dependency>
	
		</dependencies>
	
		<dependencyManagement>
			<dependencies>
				<dependency>
					<groupId>org.springframework.cloud</groupId>
					<artifactId>spring-cloud-dependencies</artifactId>
					<version>Dalston.SR3</version>
					<type>pom</type>
					<scope>import</scope>
				</dependency>
			</dependencies>
		</dependencyManagement>



* application.yml

		eureka:
		  client:
		    serviceUrl:
		      defaultZone: http://localhost:8081/eureka/ #指定注册中心地址
		server:
		  port: 8181
		spring:
		  application:
		    name: service-hi #注册的服务名

* 应用程序入口ServiceHiApp

		@SpringBootApplication
		@EnableEurekaClient
		@RestController
		public class ServiceHiApp {
		
			@Value("${server.port}")
			private String port;
			
			public static void main(String[] args) {
				SpringApplication.run(ServiceHiApp.class, args);
			}
		
			@GetMapping("/hi")
			public String sayHi(){
				return "Hi Spring Cloud </br> 端口号: " + port;
			}
		}

* 启动后刷新注册中心
![](/img/0001.png)
			红字表示注册中心进入了保护模式,此时注册中心会保留服务的注册信息,即便服务挂了,信息也不会丢失.
		尝试关闭service-hi服务,发现注册信息仍然保留在注册中心.	
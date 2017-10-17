>首先推荐一篇博客
>http://blog.csdn.net/forezp/article/details/70148833
>这是我学习springcloud参考的一个博客
>也是个人觉得写的比较清楚和全面的

#Eureka的学习使用(二)

##服务消费者

###准备工作

- 将cloud-service-hi通过不同的端口注册到注册中心,形成一个小集群
- 访问 http://localhost:8081/ 如图所示
![](/img/0011.png)

###ribbon+restTemplate实现方式

- 创建一个工程,命名为cloud-service-ribbon
- 导入pom依赖如下
		<parent>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-parent</artifactId>
			   <version>1.5.7.RELEASE</version>
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
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-starter-ribbon</artifactId>
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

- 配置文件application.yml如下
		eureka: 
		  client: 
		    serviceUrl: 
		      defaultZone: http://localhost:8081/eureka/  #注册中心地址
		server: 
		  port: 8281   #端口
		spring: 
		  application: 
		    name: service-ribbon  #注册的服务名

- 在入口中添加@EnableDiscoveryClient向服务中心注册

		@SpringBootApplication
		@EnableDiscoveryClient
		public class RibbonApp {
		
			public static void main(String[] args) {
				SpringApplication.run(RibbonApp.class, args);
			}
		
			@Bean
			@LoadBalanced//这个注解表示这个restRemplate开启负载均衡的功能。
			RestTemplate restTemplate() {
				return new RestTemplate();
			}
		
		}

- 写一个测试类HiService
 
		@Service
		public class HiService {
		
			@Autowired
			RestTemplate restTemplate;
			
			public String hiService(){
				//这里使用服务名替代url地址因为ribbon会自动根据服务名生成具体的url
				//后面的String.class为responseType,即服务的返回值.本例中为String类型
				return restTemplate.getForObject("http://SERVICE-HI/hi", String.class);
			}
		}

- 写一个controller调用HiService的方法

		@RestController
		public class HelloController {
		
			@Autowired
			HiService hiService;
			
			@RequestMapping("/hi")
			public String hiService(){
				return hiService.hiService();
			}	
		}


- 启动工程访问 http://localhost:8281/hi  , 浏览器交替显示
	
	![](/img/0012.png)
	![](/img/0013.png)


###feign实现方式

- Feign简介
		
>Feign是一个声明式的伪Http客户端，它使得写Http客户端变得更简单。使用Feign，只需要创建一个接口并注解。
它具有可插拔的注解特性，可使用Feign 注解和JAX-RS注解。Feign支持可插拔的编码器和解码器。Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果。
		
>	简而言之：
	- Feign 采用的是基于接口的注解
	- Feign 整合了ribbon

- 创建一个工程命名为cloud-service-feign
- 导入pom依赖如下
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
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-starter-feign</artifactId>
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

- 配置文件application.yml如下
		eureka:
		  client:
		    serviceUrl:
		      defaultZone: http://localhost:8081/eureka/
		server:
		  port: 8282
		spring:
		  application:
		    name: service-feign
		feign: 
		  hystrix: 
		    enabled: true

- 在入口中添加@EnableFeignClients 开去Feign功能
 
		@SpringBootApplication
		@EnableDiscoveryClient
		@EnableFeignClients
		public class FeignApp {

			public static void main(String[] args) {
				SpringApplication.run(FeignApp.class, args);
			}
		}

- 定义一个接口,通过@FeignClient("服务名")来指定调用哪个服务
		@FeignClient("service-hi")
		public interface ServiceHi {
		
			@GetMapping("/hi")
			String HiService();
		}

- 写一个controller
		@RestController
		public class FeignController {
		
			@Autowired
			ServiceHi serviceHi;
			
			@GetMapping("/hi")
			public String HiService(){
				return  serviceHi.HiService();
			}	
		}

-启动工程多次访问 http://localhost:8282/hi  , 浏览器交替显示
	![](/img/0012.png)
	![](/img/0013.png)

###至此Ribbon和Feign两种服务调用方式都实现了(今晚加鸡腿 =.=)
- Feign源码解析：http://blog.csdn.net/forezp/article/details/73480304
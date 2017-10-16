>首先推荐一篇博客
>http://blog.csdn.net/forezp/article/details/70148833
>这是我学习springcloud参考的一个博客
>也是个人觉得写的比较清楚和全面的

#配置中心的高可用
##继续对之前的工程进行改造
###cloud-config-server:
 - 新增pom依赖，使该工程可以被注册为服务
 
   		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>

 - application.yml文件中添加如下配置以指定注册中心位置：
 
		 eureka: 
		  client: 
		    serviceUrl: 
		      defaultZone: http://localhost:8081/eureka/

 - 程序入口添加 @EnableDiscoveryClient 注解，用来将服务注册到注册中心
		
		@SpringBootApplication
		@EnableConfigServer
		@EnableDiscoveryClient
		public class ConfigServerApp {
		
		    public static void main(String[] args) {
		        new SpringApplicationBuilder(ConfigServerApp.class).web(true).run(args);    
		    }
		
		}
 - 通过更改启动参数--server.port启动为两个服务
 
 ![](/img/0029.png)
 
 ![](/img/0030.png)

  - 打开 http://localhost:8081/ 可以看到成功注册服务
  
   ![](/img/0031.png)

###cloud-config-client:
- 和cloud-config-server一样，新增pom依赖

   		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>

- 配置文件 bootstrap.yml 如下：

		spring:
		  application:
		    name: config
		  cloud:
		    config:
		      #uri: http://localhost:8888/
		      profile: default
		      label: master
		      discovery: 
		        enabled: true
		        serviceId: config-server
		      
		server:
		  port: 8880
		  
		eureka: 
		  client: 
		    serviceUrl: 
		      defaultZone: http://localhost:8081/eureka/
		
- 入口上添加 @EnableDiscoveryClient 注解

		@SpringBootApplication
		@EnableDiscoveryClient
		public class ConfigClientApp {
		
		    public static void main(String[] args) {
		        new SpringApplicationBuilder(ConfigClientApp.class).web(true).run(args);
		    }
		}

- 在之前的Controller上添加 @RefreshScope 注解 (该注解应该添加在加载变量的类上)

		@RestController
		@RefreshScope
		public class ConfigController {
		
		    @Value("${info.title}")
		    private String title;
		
		    @GetMapping("/title")
		    public String getTitle(){
		        return title;
		    }
		}

- 启动服务
	
	- 访问： http://localhost:8081/ 发现服务注册成功

   		![](/img/0033.png)

	- 访问： http://localhost:8880/title 页面显示如下： 

   		![](/img/0032.png)
	
##配置刷新
- 在cloud-config-client中添加如下依赖

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>

- 重启cloud-config-client 继续访问  http://localhost:8880/title 页面显示如下

   	![](/img/0032.png)

- 修改仓库中的配置文件application.properties

   	![](/img/0034.png)

- 通过postman发送POST请求到 http://localhost:8880/refresh  返回值为：

	![](/img/0035.png)

- 继续访问 http://localhost:8880/title 页面显示如下 ，说明更新成功 

	![](/img/0036.png)
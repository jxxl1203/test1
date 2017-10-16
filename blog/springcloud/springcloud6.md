>首先推荐一篇博客
>http://blog.csdn.net/forezp/article/details/70148833
>这是我学习springcloud参考的一个博客
>也是个人觉得写的比较清楚和全面的

#配置中心的学习使用

##准备一个仓库
https://github.com/jxxl1203/config-repo-demo/

- 上传配置文件

- config.yml
	
		info: 
		  title: hahaha
		  type: default
		test: lala

- config-dev.yml
			
		info: 
		  title: springcloud-dev
		  type: dev
		test: lala

##构建配置中心
- 创建一个工程cloud-config-server

- 导入pom依赖

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
				<artifactId>spring-cloud-config-server</artifactId>
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


- 在程序入口上添加 @EnableConfigServer 开启Spring Cloud Config的服务端功能
			
		@SpringBootApplication
		@EnableConfigServer
		public class ConfigServerApp {
		
			public static void main(String[] args) {
				new SpringApplicationBuilder(ConfigServerApp.class).web(true).run(args);	
			}
		
		}

- 配置文件如下application.yml

		spring: 
		  application: 
		    name: config-server
		  cloud:
		    config:
		      server:
		        git:
		          uri: https://github.com/jxxl1203/config-repo-demo #仓库地址
		          searchPaths: repo #添加需要扫面的下属路径
		          username: #用户名
		          password: #密码
		server:
		  port: 8888

- 启动工程,访问: http://localhost:8888/config/master 页面显示如下: 
![](/img/0023.png)

	说明此时已经可以从远程仓库读取配置文件信息

**访问配置信息的URL与配置文件的映射关系如下：**

- {application}/{profile}[/{label}]
- {application}-{profile}.yml
- {label}/{application}-{profile}.yml
- {application}-{profile}.properties
- {label}/{application}-{profile}.properties

##构建客户端

- 创建一个工程cloud-config-client 

- 导入pom依赖

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
				<artifactId>spring-cloud-starter-config</artifactId>
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
					<version>Camden.SR2</version>
					<type>pom</type>
					<scope>import</scope>
				</dependency>
			</dependencies>
	
		</dependencyManagement>


- 创建程序入口

		@SpringBootApplication
		public class ConfigClientApp {
		
			public static void main(String[] args) {
				new SpringApplicationBuilder(ConfigClientApp.class).web(true).run(args);
			}
		}

- 配置文件如下: 

		spring:
		  application:
		    name: config
		  cloud:
		    config:
		      uri: http://localhost:8888/
		      profile: default
		      label: master
		server:
		  port: 8880

- 新建一个controller 

		@RestController
		public class ConfigController {
			
			@Value("${info.title}")
			private String title;
		
			@GetMapping("/title")
			public String getTitle(){
				return title;
			}
		}
			
	

- 启动工程 , 访问: http://localhost:8880/title 页面显示如下: 
![](/img/0024.png)
  
- 修改配置文件参数 spring.cloud.config.profile=dev : 
		spring:
		  application:
		    name: config
		  cloud:
		    config:
		      uri: http://localhost:8888/
		      profile: dev
		      label: master
		server:
		  port: 8880

- 启动工程 , 再次访问: http://localhost:8880/title 页面显示如下: 
![](/img/0025.png)
	说明读取到了config-dev.yml中的内容

**上述配置参数与Git中存储的配置文件中各个部分的对应关系如下：**

- spring.application.name：对应配置文件规则中的{application}部分
- spring.cloud.config.profile：对应配置文件规则中的{profile}部分
- spring.cloud.config.label：对应配置文件规则中的{label}部分
- spring.cloud.config.uri：配置中心config-server的地址

##遇到的问题以及解决思路
- **访问路径问题**
	- 我在repo目录下上传了application.properties 和 application-dev.properties
	- application.properties
	
			info.title=properties
			info.type=default
			info.test=中文

	- application-dev.properties
	
			demo.english=Englist
			demo.chinese=中文

- 先访问: http://localhost:8888/config.yml 报错了0.0
![](/img/0026.png)
	贼熟悉的页面.......

- 接着访问 http://localhost:8888/config-dev.yml 成功显示: 
![](/img/0027.png)
	(中文编码问题稍后会讲)

- 后来仔细看了人家写的博客  :	{application}-{profile}.yml
	发现{profile}用的是大括号....
于是尝试访问 http://localhost:8888/config-default.yml 发现这个问题其实需要看眼科...
![](/img/0028.png)
其实还碰到了很多需要看眼科的问题就不一一列举了....


- **读取中文乱码问题**

	- 新建一个controller,并把之前的ConfigController注释掉
	- ConfigController
		
			//@RestController
			public class ConfigController {
			
	- ApplicationController
	
			@RestController
			public class ApplicationController {
				
				@Value("${info.test}")
				private String test;
				
				@GetMapping("/test")
				public String getTest() {
					System.out.println(test);
					return test;
				}
			}


- 此时, 访问 http://localhost:8880/test 出现了中文乱码问题
	首先把获取到的数据打印出来,发现已经乱码了
	原因可能是框架在传输的时候产生了编码问题
	由于水平有限,并不十分了解该如何解决所以采用了最原始的办法...
	修改后的代码如下:

		@RestController
		public class ApplicationController {
			@Value("${info.test}")
			private String test;
			
			private boolean test_flag;
			
			@GetMapping("/test")
			public String getTest(HttpServletResponse response) throws Exception{
				synchronized (this){
					if (!test_flag){
						test = new String(test.getBytes("iso-8859-1"),"utf-8");
						test_flag = true;
					}
				}
				response.setCharacterEncoding("utf-8");
				System.out.println(test);
				return test;
			}
			
		}

如果你有更好的解决办法

 </br></br></br></br></br>
</br></br></br></br></br>
  


- 求求你救救我吧
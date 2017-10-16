#springboot学习笔记(二)

##Thymeleaf模版的使用

- pom文件导入依赖
	
		<parent>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-parent</artifactId>
			<version>1.5.7.RELEASE</version>
		</parent>
	
		<dependencies>
	
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter</artifactId>
			</dependency>
	
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-web</artifactId>
			</dependency>
			
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-thymeleaf</artifactId>
			</dependency>
		</dependencies>

- 构建一个程序入口ThymeleafApp
		
		@SpringBootApplication
		public class ThymeleafApp {
		
			public static void main(String[] args) {
				new SpringApplicationBuilder(ThymeleafApp.class).build().run(args);
			}
		
		}

- application.yml 文件如下

		server:
		  port: 8082
		spring:
		  thymeleaf:
		    cache: false # Enable template caching.
		    check-template-location: true # Check that the templates location exists.
		    content-type: text/html # Content-Type value.
		    enabled: true # Enable MVC Thymeleaf view resolution.
		    encoding: UTF-8 # Template encoding.
		    excluded-view-names: # Comma-separated list of view names that should be excluded from resolution.
		    mode: HTML5 # Template mode to be applied to templates. See also StandardTemplateModeHandlers.
		    prefix: classpath:/templates/ # Prefix that gets prepended to view names when building a URL.
		    suffix: .html # Suffix that gets appended to view names when building a URL.
		    template-resolver-order: # Order of the template resolver in the chain. spring.thymeleaf.view-names= # Comma-separated list of view names that can be resolved.

- 模版文件保存目录: src/main/resources/templates
- index.html

		<!DOCTYPE html>
		<html>
		<head lang="en">
		    <meta charset="UTF-8" />
		    <title></title>
		</head>
		<body>
		<h1 th:text="${name}">Hello World</h1> 
		</body>
		</html>

- 构建一个Controller

		@Controller
		public class ThymeleafController {
		
			@GetMapping("/")
			public String index(Model model){
				model.addAttribute("name", "Thymeleaf");
				return "index";
			}
			
		}

- 启动程序访问 http://localhost:8082/ 页面显示:
![](/img/0002.png)
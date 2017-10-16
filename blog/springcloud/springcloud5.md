>首先推荐一篇博客
>http://blog.csdn.net/forezp/article/details/70148833
>这是我学习springcloud参考的一个博客
>也是个人觉得写的比较清楚和全面的

#Zuul的学习使用

##Zuul简介

Zuul的主要功能是路由转发和过滤器。路由功能是微服务的一部分，比如／api/user转发到到user服务，/api/shop转发到到shop服务。zuul默认和Ribbon结合实现了负载均衡的功能。

##创建一个工程cloud-service-zuul
- 导入pom文件

		<parent>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-parent</artifactId>
			<version>1.5.2.RELEASE</version>
		</parent>
		<properties>
			<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
			<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
			<java.version>1.8</java.version>
		</properties>
	
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-starter-eureka</artifactId>
			</dependency>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-starter-zuul</artifactId>
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

- 再程序入口中添加@EnableZuulProxy , 开启zuul功能

		@SpringBootApplication
		@EnableEurekaClient
		@EnableZuulProxy
		public class ZuulApp {
		
			public static void main(String[] args) {
				SpringApplication.run(ZuulApp.class, args);
			}
		
		}

- 配置文件如下

		eureka: 
		  client: 
		    serviceUrl: 
		      defaultZone: http://localhost:8081/eureka/
		server: 
		  port: 8280
		spring: 
		  application: 
		    name: service-zuul
		zuul: 
		  routes: 
		    api-a: 
		      path: /api-a/**
		      serviceId: SERVICE-RIBBON
		    api-b: 
		      path: /api-b/**
		      serviceId: SERVICE-FEIGN
		    api-c:
		      path: /api-c/**
		      url: http://localhost:8182/ 
		#这里用url或者serviceId都可以正确访问

- 启动所有工程, 打开浏览器分别访问
  - http://localhost:8769/api-a/hi 
  - http://localhost:8769/api-b/hi 
  - http://localhost:8769/api-c/hi 
- 浏览器正常显示: 
![](/img/0020.png)
- 说明zuul起到了路由的作用

## 服务过滤

- zuul还提供了用于安全过滤的方法,继续改造工程

		@Component
		public class MyFilter extends ZuulFilter {
			
			/**
			 * filterType：返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型，具体如下： 
			 * pre：路由之前
			 * routing：路由之时 
			 * post： 路由之后 
			 * error：发送错误调用
			 */
		
			@Override
			public String filterType() {
				return "pre";
			}

			@Override
			public Object run() {
				RequestContext ctx = RequestContext.getCurrentContext();
		        HttpServletRequest request = ctx.getRequest();
		        Object accessToken = request.getParameter("token");
		        if(accessToken == null) {
		            ctx.setSendZuulResponse(false);
		            ctx.setResponseStatusCode(401);
		            try {
		                ctx.getResponse().getWriter().write("token is empty");
		            }catch (Exception e){}
		
		            return null;
		        }
		        return null;
			}
		
			@Override
			public boolean shouldFilter() {
				return true;
			}
		
			@Override
			public int filterOrder() {
				return 0;
			}
		}

- 启动项目 
	- 访问 http://localhost:8280/api-a/hi 显示
	![](/img/0021.png)
	
	- 访问  
	![](/img/0022.png)
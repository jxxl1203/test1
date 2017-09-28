# Hystrix的学习使用

## 断路器简介
- 当访问服务出现故障时,断路器会调用FallBack方法返回一个固定值,以避免长时间等待造成的阻塞
- 当对特定的服务的调用的不可用达到一个阀值（Hystric 是5秒20次） 断路器将会被打开。

## Ribbon中实现Hystrix
- 在原有的cloud-service-ribbon上进行修改
  - 首先在pom文件中添加依赖
  
  		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-hystrix</artifactId>
		</dependency>

  - 在入口中添加@EnableHystrix启动Hystrix
  
		@SpringBootApplication
		@EnableDiscoveryClient
		@EnableHystrix
		public class RibbonApp {
		
			public static void main(String[] args) {
				SpringApplication.run(RibbonApp.class, args);
			}
		
			@Bean
			@LoadBalanced
			RestTemplate restTemplate() {
				return new RestTemplate();
			}
		
		}

  - 在测试类的方法上添加@HystrixCommand

		@Service
		public class HiService {
		
			@Autowired
			RestTemplate restTemplate;
			
			@HystrixCommand(fallbackMethod = "hiError")
			public String hiService(){
				return restTemplate.getForObject("http://service-Hi/hi", String.class);
			}
			
			public String hiError(){  //参数要与调用方法一直
				return "SERVICE-HI ERROR !   ";
			}
			
		}

- 启动工程,访问 http://localhost:8281/hi ,浏览器显示
![](/img/0014.png)

- 接着关闭service-hi服务,再次访问 http://localhost:8281/hi ,浏览器显示
![](/img/0015.png)

	- 说明断路器起作用了

## Feign中实现Hystrix
- 由于feign中集成了Hystrix,所以我们可以直接使用,只需要在配置文件中打开Hystrix

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
		    enabled: true #默认是关闭的
	- 至于为什么默认是关闭的请看https://github.com/spring-cloud/spring-cloud-netflix/issues/1277
	- 大概是为了防止在不需要使用Hystrix时却因为使用了feign而导致调用了Hystrix

- 写一个类继承之前的ServiceHi接口,这样框架就会在连接异常时自动调用里面的方法

		@Component
		public class ServiceHiHystric implements ServiceHi{
		
			@Override
			public String HiService() {
				return "SERVICE-HI ERROR";
			}
		
		}

- 在ServiceHi中对@FeignClient进行更改,添加fallback参数
 
		@FeignClient(value = "service-hi",fallback = ServiceHiHystric.class)
		public interface ServiceHi {
		
			@GetMapping("/hi")
			String HiService();
			
		}

- 启动工程,访问 http://localhost:8282/hi ,浏览器显示
![](/img/0016.png)

- 接着关闭service-hi服务,再次访问 http://localhost:8282/hi ,浏览器显示
![](/img/0017.png)

### ok 至此Riboon和Feign实现Hystrix都搞定了
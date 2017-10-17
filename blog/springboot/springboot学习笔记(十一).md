#springboot学习笔记(十一)
##使用log4j记录日志

- 导入 pom 依赖
	1. springboot1.4以上版本中需引用log4j2,而不是log4j
	2. spring-boot-starter 中包含了 spring-boot-starter-logging ，该依赖内容就是Spring Boot默认的日志框架Logback，所以我们在引入log4j2之前，需要先排除该包的依赖，再引入log4j2的依赖，就像下面这样：
	
			<parent>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-parent</artifactId>
				<version>1.5.7.RELEASE</version>
			</parent>
		
			<dependencies>
		
				<dependency>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter</artifactId>
					<exclusions>
						<exclusion>
							<groupId>org.springframework.boot</groupId>
							<artifactId>spring-boot-starter-logging</artifactId>
						</exclusion>
					</exclusions>
				</dependency>
		
				<dependency>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-web</artifactId>
				</dependency>
		
				<dependency>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-log4j2</artifactId>
				</dependency>
		
				<dependency>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-aop</artifactId>
				</dependency>
			</dependencies>
	

- 构建一个程序入口

		@SpringBootApplication
		public class Log4jApp {
		
			public static void main(String[] args) {
				new SpringApplicationBuilder(Log4jApp.class).build().run(args);
			}
		}


- application.yml 文件如下

		spring:
		  aop: 
		    auto: true
		    proxy-target-class: false
		server: 
		  port: 8091

- log4j2.xml配置文件如下(log4j2中不再支持.properties配置文件)

		<?xml version="1.0" encoding="UTF-8"?>
		<!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
		<!--Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出 -->
		<!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数 -->
		<configuration status="WARN" monitorInterval="30">
			<!--先定义所有的appender -->
			<appenders>
				<!--这个输出控制台的配置 -->
				<console name="Console" target="SYSTEM_OUT">
					<!--输出日志的格式 -->
					<PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n" />
				</console>
				<!--文件会打印出所有信息，这个log每次运行程序会自动清空，由append属性决定，这个也挺有用的，适合临时测试用 -->
				<File name="log" fileName="log/test.log" append="true">
					<PatternLayout
						pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n" />
				</File>
				<!-- 这个会打印出所有的info及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档 -->
				<RollingFile name="RollingFileInfo" fileName="${sys:user.home}/logs/info.log"
					filePattern="${sys:user.home}/logs/$${date:yyyy-MM}/info-%d{yyyy-MM-dd}-%i.log">
					<!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch） -->
					<ThresholdFilter level="info" onMatch="ACCEPT"
						onMismatch="DENY" />
					<PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n" />
					<Policies>
						<TimeBasedTriggeringPolicy />
						<SizeBasedTriggeringPolicy size="100 MB" />
					</Policies>
				</RollingFile>
				<RollingFile name="RollingFileWarn" fileName="${sys:user.home}/logs/warn.log"
					filePattern="${sys:user.home}/logs/$${date:yyyy-MM}/warn-%d{yyyy-MM-dd}-%i.log">
					<ThresholdFilter level="warn" onMatch="ACCEPT"
						onMismatch="DENY" />
					<PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n" />
					<Policies>
						<TimeBasedTriggeringPolicy />
						<SizeBasedTriggeringPolicy size="100 MB" />
					</Policies>
					<!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20 -->
					<DefaultRolloverStrategy max="20" />
				</RollingFile>
				<RollingFile name="RollingFileError" fileName="${sys:user.home}/logs/error.log"
					filePattern="${sys:user.home}/logs/$${date:yyyy-MM}/error-%d{yyyy-MM-dd}-%i.log">
					<ThresholdFilter level="error" onMatch="ACCEPT"
						onMismatch="DENY" />
					<PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n" />
					<Policies>
						<TimeBasedTriggeringPolicy />
						<SizeBasedTriggeringPolicy size="100 MB" />
					</Policies>
				</RollingFile>
			</appenders>
			<!--然后定义logger，只有定义了logger并引入的appender，appender才会生效 -->
			<loggers>
				<!--过滤掉spring和mybatis的一些无用的DEBUG信息 -->
				<logger name="org.springframework" level="INFO"></logger>
				<logger name="org.mybatis" level="INFO"></logger>
				<root level="all">
					<appender-ref ref="Console" />
					<appender-ref ref="RollingFileInfo" />
					<appender-ref ref="RollingFileWarn" />
					<appender-ref ref="RollingFileError" />
				</root>
			</loggers>
		</configuration>


- 编写一个简单的Controller

		@RestController
		public class HelloController {
		
			@GetMapping("/hello")
			public String hello(String name){
				return "Hello " + name;
			}
			
		}

- 编写一个AOP日志管理切面

		@Aspect
		@Component
		public class WebLogAspect {
		
			private Logger logger = LogManager.getLogger(WebLogAspect.class);
		
			@Pointcut("execution(public * pers.ray.springboot.controller..*.*(..))")
			public void webLog() {
			}
		
			@Before("webLog()")
			public void doBefore(JoinPoint joinPoint) throws Throwable {
				// 接收到请求，记录请求内容
				ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
				HttpServletRequest request = attributes.getRequest();
				// 记录下请求内容
				logger.info("URL : " + request.getRequestURL().toString());
				logger.info("HTTP_METHOD : " + request.getMethod());
				logger.info("IP : " + request.getRemoteAddr());
				logger.info("CLASS_METHOD : " + joinPoint.getSignature().getDeclaringTypeName() + "."
						+ joinPoint.getSignature().getName());
				logger.info("ARGS : " + Arrays.toString(joinPoint.getArgs()));
			}
		
		}


- 启动程序访问 http://localhost:8091/hello?name=Ray 控制台打印如下日志: 
![](/img/0009.png)

打开文件 C:\Users\Administrator\logs\info.log
可以看到日志文件正常生成了
![](/img/0010.png)


		个人学习springboot博客地址:
		http://blog.csdn.net/forezp/article/details/70341818
		http://blog.didispace.com/Spring-Boot%E5%9F%BA%E7%A1%80%E6%95%99%E7%A8%8B/
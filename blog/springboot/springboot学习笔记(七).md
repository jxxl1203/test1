#springboot学习笔记(七)
##整合Redis

- 导入 pom 依赖
	
		<parent>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-parent</artifactId>
			<version>1.5.7.RELEASE</version>
		</parent>
		<dependencies>
	
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-web</artifactId>
			</dependency>
			
					<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-test</artifactId>
			</dependency>
	
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-data-redis</artifactId>
			</dependency>
	
		</dependencies>

- 构建一个程序入口


		@SpringBootApplication
		@RestController
		public class RedisApp {
		
			public static Logger logger = LoggerFactory.getLogger(RedisApp.class);
		
			@Autowired
			RedisDao redisDao;
		
			@GetMapping("")
			public void getKey() {
				redisDao.setKey("id", "1");
				redisDao.setKey("name", "zhangsan");
		
				logger.info(redisDao.getValue("id"));
				logger.info(redisDao.getValue("name"));
				
			}
		
			public static void main(String[] args) {
				new SpringApplicationBuilder(RedisApp.class).build().run(args);
			}
		
		}


- application.yml 文件如下

		server: 
		  port: 8087
		spring:
		  redis:
		    host: 192.168.100.201
		    port: 6379
		    #password: 
		    database: 1
		    pool: 
		      max-active: 8
		      max-wait: -1
		      max-idle: 500
		      min-idle: 0
		    timeout: 0   
		    


- 构建一个数据库访问层

		@Repository
		public class RedisDao {
		
			@Autowired
			private StringRedisTemplate template;
			
			public void setKey(String key, String value){
				ValueOperations<String, String> ops = template.opsForValue();
				ops.set(key, value, 1, TimeUnit.MINUTES);;
			}
			
			public String getValue(String key){
				ValueOperations<String, String> ops = template.opsForValue();
				return ops.get(key);
			}
			
		}


- 启动程序访问: http://localhost:8087/  控制台打印日志如下
![](/img/0003.png)

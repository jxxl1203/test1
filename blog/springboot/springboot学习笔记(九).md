#springboot学习笔记(九)
##开启缓存spring cache

- 导入 pom 依赖

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
				<artifactId>spring-boot-starter-test</artifactId>
			</dependency>
	
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-cache</artifactId>
			</dependency>
	
		</dependencies>

- 构建一个程序入口

		@SpringBootApplication
		public class CacheApp {
		
			public static void main(String[] args) {
				SpringApplication.run(CacheApp.class, args);
			}
		}


- application.yml 文件如下

		server: 
		  port: 8089


- 构建一个数据访问层

		@Repository
		public class UserDaoImpl implements UserDao{
		
			@Override
			public User getById(Integer id) {
				System.out.println("get user " + id + " in repo");
				dosomething();
				return new User(id);
			}
		
			private void dosomething(){
				
				try {
					Thread.sleep(2000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			
		}


- 编写测试类 
		
		@Component
		public class AppRunner implements CommandLineRunner {
		
		    private static final Logger logger = LoggerFactory.getLogger(AppRunner.class);
		
		    private final UserDao userDaoImpl;
		
		    public AppRunner(UserDao userDaoImpl) {
		        this.userDaoImpl = userDaoImpl;
		    }
		
		    @Override
		    public void run(String... args) throws Exception {
		        logger.info("Fetching users...");
		        logger.info("" + userDaoImpl.getById(1));
		        logger.info("" + userDaoImpl.getById(2));
		        logger.info("" + userDaoImpl.getById(1));
		        logger.info("" + userDaoImpl.getById(2));
		        logger.info("" + userDaoImpl.getById(1));
		        logger.info("" + userDaoImpl.getById(1));
		        System.out.println();
		    }
		
		}

- 启动程序,控制台打印如下: 
![](/img/0005.png)
可以看见每一次都执行了数据访问层的方法,此时并没有开启缓存技术

###开启缓存技术

- 在程序的入口中加入@ EnableCaching开启缓存技术：

		@SpringBootApplication
		@EnableCaching
		public class CacheApp {
		
			public static void main(String[] args) {
				SpringApplication.run(CacheApp.class, args);
			}
		}

- 在需要缓存的地方加入@Cacheable注解，比如在 getById() 方法上加入@Cacheable(“users”)，这个方法就开启了缓存策略，当缓存有这个数据的时候，会直接返回数据，不会等待去查询数据库。

		@Repository
		public class UserDaoImpl implements UserDao{
		
			@Override
			@Cacheable("users")
			public User getById(Integer id) {
				System.out.println("get user " + id + " in repo");
				dosomething();
				return new User(id);
			}
		
			private void dosomething(){
				
				try {
					Thread.sleep(2000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			
		}

- 此时启动程序,控制台打印如下: 
![](/img/0006.png)
只有前面两个数据被打印了,说明此时缓存起作用了
#springboot学习笔记(五)
##JPA的使用

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
				<artifactId>spring-boot-starter-web</artifactId>
			</dependency>
	
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-data-jpa
	            </artifactId>
			</dependency>
	
			<dependency>
				<groupId>mysql</groupId>
				<artifactId>mysql-connector-java</artifactId>
				<scope>runtime</scope>
			</dependency>
	
		</dependencies>


- 构建一个程序入口

		@SpringBootApplication
		public class JPAApp {
		
			public static void main(String[] args) {
				new SpringApplicationBuilder(JPAApp.class).build().run(args);
			}
		
		}


- application.yml 文件如下

		server: 
		  port: 8085
		spring:
		  datasource: 
		    url: jdbc:mysql://localhost:3306/springboot
		    username: root
		    password: root
		    driver-class-name: com.mysql.jdbc.Driver
		  jpa: 
		    hibernate: 
		      ddl-auto: update  
		    show-sql: true

- User类直接Copy上一个工程的

- 构建一个Controller

		@RestController
		@RequestMapping("user")
		public class UserController {
		
			@Autowired
			UserDao userDao;
			
			@GetMapping("")
			public List<User> getAllUser(){
				return userDao.findAll();
			}
			
			@GetMapping("/id/{id}")
			public User getById(@PathVariable Integer id){
				User user = userDao.getOne(id);
				System.out.println(user);
				return user;
			}
			
			@GetMapping("/name/{name}")
			public User getByname(@PathVariable String name){
				return userDao.getByName(name);
			}
			
			@PostMapping("/search")
			public List<User> searchByNameAndAge(User user){
				System.out.println(user);
				List<User> list = userDao.getLikeNameAndAge(user.getName(), user.getAge());
				return list;
			}
			
		}

- 构建UserDao

		@Repository
		public interface UserDao extends JpaRepository<User, Integer>{
			
			public User getByName(String name);
			
			@Query("from User u where u.name like %:name% and (u.age = :age or :age is null)")
			public List<User> getLikeNameAndAge(@Param("name") String name, @Param("age") Integer age);
			
		}


- 测试过程省略......
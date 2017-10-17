#springboot学习笔记(六)
## 整合Mybatis

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
				<groupId>org.mybatis.spring.boot</groupId>
				<artifactId>mybatis-spring-boot-starter</artifactId>
				<version>1.3.1</version>
			</dependency>
	
			<dependency>
				<groupId>mysql</groupId>
				<artifactId>mysql-connector-java</artifactId>
				<scope>runtime</scope>
			</dependency>
			<dependency>
				<groupId>com.alibaba</groupId>
				<artifactId>druid</artifactId>
				<version>1.0.29</version>
			</dependency>
		</dependencies>

- 构建一个程序入口

		@SpringBootApplication
		public class MybatisApp {
		
			public static void main(String[] args) {
				new SpringApplicationBuilder(MybatisApp.class).build().run(args);
			}
		
		}


- application.yml 文件如下

		server: 
		  port: 8086
		spring:
		  datasource: 
		    url: jdbc:mysql://localhost:3306/springboot
		    username: root
		    password: root
		    driver-class-name: com.mysql.jdbc.Driver

- User类省略

- 构建一个Controller

		@RestController
		@RequestMapping("user")
		public class UserController {
		
			@Autowired
			UserMapper userMapper;
			
			@GetMapping("")
			public List<User> getAllUser(){
				return userMapper.getAllUser();
			}
			
			@GetMapping("/id/{id}")
			public User getById(@PathVariable Integer id){
				return userMapper.getById(id);
			}
			
			@PostMapping("")
			public String add(User user){
				userMapper.add(user);
				return "success add";
			}
		}

- 构建UserMapper接口 

		@Mapper
		public interface UserMapper {
		
			@Insert("insert into user values(null,#{name},#{age},#{sex})")
			int add(User user);
			
			@Update("update user set name = #{name}, age = #{age}, sex = #{sex} where id = #{id}")
			int update(@Param("name") String name,@Param("age") Integer age,@Param("sex") String sex,@Param("id") Integer id);
			
			@Delete("delete from user where id = #{id}")
			int delete(Integer id);
			
			@Select("select * from user where id = #{id}")
			User getById(Integer id);
			
			@Select("select * from user")
			List<User> getAllUser();
		}





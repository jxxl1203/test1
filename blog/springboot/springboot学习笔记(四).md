#springboot学习笔记(四)
##JdbcTemplate的使用

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
				<artifactId>spring-boot-starter-jdbc</artifactId>
			</dependency>
	
			<dependency>
				<groupId>mysql</groupId>
				<artifactId>mysql-connector-java</artifactId>
			</dependency>
	
		</dependencies>

- 构建一个程序入口

		@SpringBootApplication
		public class JdbcTemplateApp {
		
			public static void main(String[] args) {
				new SpringApplicationBuilder(JdbcTemplateApp.class).build().run(args);
			}
		}
		

- application.yml 文件如下

		server:
		  port: 8084
		spring:
		  datasource: 
		    url: jdbc:mysql://localhost:3306/springboot
		    username: root
		    password: root
		    driver-class-name: com.mysql.jdbc.Driver  

- 构建一个实体类User

		public class User implements Serializable{
			
		    private Integer id;
			private String name;
			private Integer age;
			private String sex;
			
			
			public Integer getId() {
				return id;
			}
			public void setId(Integer id) {
				this.id = id;
			}
			public String getName() {
				return name;
			}
			public void setName(String name) {
				this.name = name;
			}
			public Integer getAge() {
				return age;
			}
			public void setAge(Integer age) {
				this.age = age;
			}
			public String getSex() {
				return sex;
			}
			public void setSex(String sex) {
				this.sex = sex;
			}
			public String toString() {
				return "User [id=" + id + ", name=" + name + ", age=" + age + "]";
			}
		}


- 构建一个UserController

		@RestController
		@RequestMapping("user")
		public class UserController {
		
			@Autowired
			UserService userservice;
			
			@PostMapping("")
			public void add(User user){
				userservice.add(user);
			}
			
			@DeleteMapping("/id/{id}")
			public void deleteById(@PathVariable Integer id){
				userservice.deleteById(id);
			}
			
			@DeleteMapping("/name/{name}")
			public void deleteByName(@PathVariable String name){
				userservice.deleteByName(name);
			}
			
			@GetMapping("/count")
			public Integer getCount(){
				return userservice.getCount();
			}
			
			@GetMapping("")
			public List<User> getAllUsers(){
				return userservice.getAllUsers();
			}
			
			@GetMapping("/id/{id}")
			public User getById(@PathVariable Integer id){
				return userservice.getById(id);
			}
			
			@GetMapping("/name/{name}")
			public List<User> getByName(@PathVariable String name){
				return userservice.getByName(name);
			}
		}

- 服务层

	- UserService
			public interface UserService {
			
				public void add(User user);
				
				public void deleteById(Integer id);
				
				public void deleteByName(String name);
				
				public Integer getCount();
				
				public List<User> getAllUsers();
				
				public User getById(Integer id);
				
				public List<User> getByName(String name);
			}

	- UserServiceImpl
	
			@Service
			public class UserServiceImpl implements UserService {
			
				@Autowired
				private JdbcTemplate template;
				
				public void add(User user) {
					template.update("insert into USER values(null,?,?,?)", user.getName(),user.getAge(),user.getSex());
				}
			
				public void deleteById(Integer id) {
					template.update("delete from USER where id = ?", id);
				}
			
				public void deleteByName(String name) {
					template.update("delete from USER where name = ?", name);
				}
			
				public Integer getCount() {
					return template.queryForObject("select count(1) from USER", Integer.class);
				}
			
				public List<User> getAllUsers() {
					return template.query("select * from USER", new UserRowMapper());
				}
			
				public User getById(Integer id) {
					List<User> list = template.query("select * id,name,age,sex from USER where id = ?",new Object[]{id}, new UserRowMapper());
					return list.isEmpty() ? null : list.get(0);
				}
			
				@Override
				public List<User> getByName(String name) {
					List<User> list = template.query("select * id,name,age,sex from USER where name = ?",new Object[]{name}, new UserRowMapper());
					return list;
				}
			
			}

- 创建一个RowMapper

		public class UserRowMapper implements RowMapper<User> {
		
			public User mapRow(ResultSet rs, int i) throws SQLException {
				User user = new User();
				user.setId(rs.getInt(1));
				user.setName(rs.getString(2));
				user.setAge(rs.getInt(3));
				user.setSex(rs.getString(4));
				return user;
			}
		
		}
		


- 测试过程省略......
#springboot学习笔记(三)
##构建一个较为复杂的Restful API


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
			
		</dependencies>


- 构建一个程序入口

		@SpringBootApplication
		public class RestfulApp {
		
			public static void main(String[] args) {
				new SpringApplicationBuilder(RestfulApp.class).build().run(args);
			}
		}


- application.yml 文件如下

		server:
		  port: 8083

- 构建实体类User

		public class User implements Serializable{
			
		    private Integer id;
			private String name;
			private Integer age;
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
			@Override
			public String toString() {
				return "User [id=" + id + ", name=" + name + ", age=" + age + "]";
			}
			
		}


- 构建一个Controller

		@RestController()
		@RequestMapping("/users")
		public class UserController {
		
			private Map<Integer,User> map = new HashMap<>();
			
			@GetMapping("")
			public List<User> getUserList(){
				List<User> list = new ArrayList<>(map.values());
				return list;
			}
			
			@PostMapping("")
			public String insertUser(User user){
				map.put(user.getId(), user);
				return "insert success !";
			}
			
			@GetMapping("/{id}")
			public User getUserById(@PathVariable Integer id){
				return map.get(id);
			}
			
			@PutMapping("/{id}")
			public String updateUserById(@PathVariable Integer id,User user){
				map.put(id, user);
				return "update success !";
			}
			@DeleteMapping("/{id}")
			public String deleteUserById(@PathVariable Integer id){
				map.remove(id);
				return "delete success !";
			}
		}		


- 通过postman测试过程就省略了...
#springboot学习笔记(八)
##集成Swagger2

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
				<groupId>io.springfox</groupId>
				<artifactId>springfox-swagger2</artifactId>
				<version>2.6.1</version>
			</dependency>
	
			<dependency>
				<groupId>io.springfox</groupId>
				<artifactId>springfox-swagger-ui</artifactId>
				<version>2.6.1</version>
			</dependency>
			
		</dependencies>

- 构建一个程序入口
		
		@SpringBootApplication
		public class SwaggerApp {
		
			public static void main(String[] args) {
				new SpringApplicationBuilder(SwaggerApp.class).build().run(args);
			}
		}

- 编写配置类 

@Configuration
@EnableSwagger2
public class Swagger2 {
	@Bean
	public Docket createRestApi() {
		return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo()).select()
				.apis(RequestHandlerSelectors.basePackage("pers.ray.springboot.controller")).paths(PathSelectors.any()).build();
	}

	private ApiInfo apiInfo() {
		return new ApiInfoBuilder().title("Spring Boot中使用Swagger2构建RESTful APIs")
				.description("学习使用Swagger2构建RESTful APIs")
				.termsOfServiceUrl("http://blog.csdn.net/xl8117602").version("1.0").build();
	}
}

通过@Configuration注解，表明它是一个配置类，@EnableSwagger2开启swagger2。apiINfo()配置一些基本的信息。apis()指定扫描的包会生成文档。


- application.yml 文件如下

		server: 
		  port: 8088

- 构建一个Controller

@RestController
@RequestMapping("/user")
public class UserController {

	private Map<Integer, User> map = new ConcurrentHashMap<>();
	
	@PostMapping("")
	@ApiOperation(value="添加用户", notes="添加用户")
	public String add(User user) {
		map.put(user.getId(), user);
		return "Success Add !";
	}
	
	@ApiOperation(value="获取用户列表", notes="")
	@GetMapping("")
	public List<User> getAll(){
		List<User> list = new ArrayList<>(map.values());
		return list;
	}
	
	
	@ApiOperation(value="获取用户详细信息", notes="根据url的id来获取用户详细信息")
	@ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Integer")
	@GetMapping("/id/{id}")
	public User getById(@PathVariable Integer id){
		return map.get(id);

	}

	@ApiOperation(value="更新用户详细信息", notes="根据url的id来指定更新对象，并根据传过来的user信息来更新用户详细信息")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long"),
            @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
    })
	@PutMapping("/id/{id}")
	public String update(@PathVariable Integer id, User user){
		User u = map.get(id);
		
		u.setName(user.getName());
		u.setAge(user.getAge());
		
		map.put(id, u);
		return "Success Update !";
	}
	
	@ApiOperation(value="删除用户", notes="根据url的id删除用户")
	@DeleteMapping("/id/{id}")
	public String delete(@PathVariable Integer id){
		map.remove(id);
		return "Success Delete !";
	}
	
	
}


- 启动程序,访问: http://localhost:8088/swagger-ui.html#/ 就能看到swagger-ui: 
![](/img/0004.png)
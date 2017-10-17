#springboot学习笔记(十二)
##使用RestTemplate消费服务

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
				<groupId>com.fasterxml.jackson.core</groupId>
				<artifactId>jackson-databind</artifactId>
			</dependency>
	
		</dependencies>
	

- 构建一个程序入口

		@SpringBootApplication
		public class RestTemplateApp {
		
			Logger log = LoggerFactory.getLogger(RestTemplateApp.class);
			
			public static void main(String[] args) {
				new SpringApplicationBuilder(RestTemplateApp.class).build().run(args);
			}
		
			@Bean
			public RestTemplate setRestTemplate(RestTemplateBuilder builder){
					return builder.build();
			}
		
			@Bean
			public CommandLineRunner run(RestTemplate restTemplate) {
				return args -> {
					String quote = restTemplate.getForObject("http://gturnquist-quoters.cfapps.io/api/random", String.class);
					log.info(quote.toString());
				};
			}
		
		}

- application.yml 文件如下

		server: 
		  port: 8092

- 构建一个服务发布者

		@RestController
		@RequestMapping("/user")
		public class ProducterController {
		
			@GetMapping("")
			public User getUser(){
				return new User(1);
			}
			
			@GetMapping("/list")
			public List<User> getUserList(){
				List<User> list = new ArrayList<>();
				for(int i = 0; i < 10; i++){
					list.add(new User(i));
				}
				return list;
			}
		}

- 构建一个服务消费者者

		@RestController
		public class CustomerController {
			
			@Autowired
			private RestTemplate restTemplate;
			
			@GetMapping("/getUser")
			public User getUser() throws Exception{
				String json = restTemplate.getForObject("http://localhost:8080/user", String.class);
				System.out.println(json);
				ObjectMapper mapper = new ObjectMapper();
				User user = mapper.readValue(json, User.class);
				return user;
			}
			@GetMapping("/getList")
			public List<User> getUserList() throws Exception{
				String json = restTemplate.getForObject("http://localhost:8080/user/list", String.class);
				ObjectMapper mapper = new ObjectMapper();
				/*JavaType javaType = mapper.getTypeFactory().constructParametricType(List.class, User.class);
				List<User> list = mapper.readValue(json, javaType);*/
				List<User> list = mapper.readValue(json, List.class);
				return list;
			}
		}

- 启动程序访问 http://localhost:8092/getUser 页面显示如下: 
![](/img/0011.png)



	个人学习springboot博客地址:
	http://blog.csdn.net/forezp/article/details/70341818
	http://blog.didispace.com/Spring-Boot%E5%9F%BA%E7%A1%80%E6%95%99%E7%A8%8B/
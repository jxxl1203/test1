#springboot学习笔记(一)

##实现一个简单的HelloWorld API

- pom文件导入依赖

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
				<artifactId>spring-boot-starter-test</artifactId>
				<scope>test</scope>
			</dependency>
			
		</dependencies>


- 构建一个程序入口 HelloApp 

		@SpringBootApplication
		public class HelloApp {
		
			public static void main(String[] args){
				new SpringApplicationBuilder(HelloApp.class).web(true).run(args);
			}
			
		}

- 构建一个HelloController

		@RestController
		public class HelloController {
		
			@GetMapping("/hello")
			public String hello(){
				return "Hello World";
			}
			
		}

- 如果需要指定配置新建一个application.yml文件即可,比如我们通过server.port 指定端口号

		server:
		  port: 8081
		  

- 启动HelloApp 访问 http://localhost:8081/hello 页面成功显示 
![](/img/0001.png)

- 接下来进行Test类的编写 springboot1.4之后只需要通过@RunWith 和 @SpringBootTest 两个注解启动测试类即可
		
		@RunWith(SpringRunner.class)
		@SpringBootTest(classes = HelloAppTest.class)
		public class HelloAppTest {
		
			private MockMvc mvc;
			
			@Before
			public void setUp(){
				mvc = MockMvcBuilders.standaloneSetup(new HelloController()).build();
			}
			
			@Test
			public void testHello() throws Exception {
					mvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
							.andExpect(status().isOk())
							.andExpect(content().string(equalTo("Hello World")));
			}
			
		}

启动Junit测试 成功通过
#springboot学习笔记(十)
##整合RabbitMQ

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
				<artifactId>spring-boot-starter-amqp</artifactId>
			</dependency>

			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-test</artifactId>
			</dependency>
	
		</dependencies>

- 构建一个程序入口
		
		@SpringBootApplication
		public class RabbitMQApp {
		
			public static void main(String[] args) {
				new SpringApplicationBuilder(RabbitMQApp.class).build().run(args);
			}
		
		}


- application.yml 文件如下

		spring:
		  application: 
		    name: rabbitmq-demo
		  rabbitmq: 
		    host: 192.168.100.201
		    port: 5672
		    username: root
		    password: root
		server:
		  port: 8090

- 构建一个消息发送者: 

		@Component
		public class Sender {
		
			@Autowired
			private AmqpTemplate rabbitTemplate;
			
			public void send(String message){
				this.rabbitTemplate.convertAndSend("msg",message);
				System.out.println("send a message : " + message + " date: " + new Date());
			}
		}


- 构建一个消息接收者: 

		@Component
		@RabbitListener(queues = "msg")
		public class Receiver {
		
		    @RabbitHandler
			public void process(String msg){
				System.out.println("Receiver a message : " + msg + " date: " + new Date());
			}
			
		}



- 创建一个测试方法: 

		@RunWith(SpringRunner.class)
		@SpringBootTest
		public class RabbitTest {
		
			@Autowired
			private Sender sender;
			
			@Test
			public void testRabbit(){
				sender.send("Hello RabbitMQ !");
			}
			
		}

-先启动程序入口,再启动测试类
测试类控制台打印:
![](/img/0007.png)

切回RabbitMQApp控制台发现打印如下数据:
![](/img/0008.png)


>首先推荐一篇博客
>http://blog.csdn.net/forezp/article/details/70148833
>这是我学习springcloud参考的一个博客
>也是个人觉得写的比较清楚和全面的

#消息总线的学习
##RabbitMQ

- 对之前的cloud-config-client进行修改

- 导入pom依赖 

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-bus-amqp</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>

- 添加RabbitMQ配置信息

		spring:
		  rabbitmq :
		    host: localhost
		    port: 5672
		    #username: 
		    #password: 
		      
- 至此，代码修改部分完成了。我们以8880和8881端口启动cloud-config-client
- 打开注册中心监控，可以看到配置服务和客户端都注册成功
![](/img/0037.png)

- 访问 http://localhost:8880/title 能够正确读取配置文件信息

- 更改配置文件信息，并通过POST请求发送到 http://localhost:8880/bus/refresh 

- 此时再次访问 http://localhost:8880/title 和 http://localhost:8881/title 均能显示更新后的配置文件

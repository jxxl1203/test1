>首先推荐一篇博客
>http://blog.csdn.net/forezp/article/details/70148833
>这是我学习springcloud参考的一个博客
>也是个人觉得写的比较清楚和全面的

#Eureka的学习使用(三)
##实现注册中心的高可用

* 基于之前的单节点注册中心,对配置文件进行修改
  * 配置文件修改
  
  	* application-node1.yml :
	 
			 server: 
			    port: 8081
			  eureka: 
			  	client: 
			      serviceUrl:
			        defaultZone: http://node2:8082/eureka/ 
			  instance: 
			    hostname: node1

 	 * application-node2.yml
	  		server: 
			  port: 8082
			eureka: 
			  client: 
			    serviceUrl:
			      defaultZone: http://node1:8081/eureka/
			  instance: 
			    hostname: node2

 	 * 对application.yml进行修改
  
			spring:
		  	  application:
		        name: eureka-server
		    eureka: 
		      client: 
		        register-with-eureka: true #打开这个,实现注册中心相互注册
		        fetch-registry: true #打开这个,实现注册中心的异步同步


          



  * 修改 hosts 文件:
	* Windows文件位置: C:\Windows\System32\drivers\etc\hosts
	* linux文件位置: /etc/hosts
	
  - 在hosts中添加这两个映射关系,与上面的hostname相对应
  
			127.0.0.1	node1
			127.0.0.1	node2

  * ***在这里碰到了一个坑***
	
			没有添加 hosts 映射  且两个节点的 hostname 都用的 localhost 
			此时发生了一些不可描述的错误,最后会详细讲解
  
  * 通过不同的启动参数启动两个注册中心
    * 在Run Configurations里面添加启动参数
   
	![](/img/0002.png)
 
	![](/img/0003.png)
  
  * 启动后访问
  	* http://localhost:8081/ 
  	![](/img/0006.png)  
	
	* http://localhost:8082/ 
  	![](/img/0007.png)

  * 此时两个注册中心成功组成了一个集群
</br>
***
</br>
# **坑来了**
### 没有更改 hosts 文件及 hostname配置时 

  * 分别访问:  http://localhost:8081/  和  http://localhost:8082/ 出现相同情况,此时两个节点相对独立,并没有实现HA
  
	![](/img/0004.png) 
    * 此时的配置文件如下
		application-node1.yml :     
			server: 
			  port: 8081
			eureka: 
			  client: 
			    serviceUrl:
			      defaultZone: http://localhost:8082/eureka/
			  instance: 
			    hostname: localhost

		application-node2.yml :
			server: 
			  port: 8082
			eureka: 
			  client: 
			    serviceUrl:
			      defaultZone: http://localhost:8081/eureka/
			  instance: 
			    hostname: localhost
    
  * 简单做个小测试就能证实这个结论
	* 我们把服务注册在8081节点
		cloud-service-hi 工程配置文件如下

			eureka:
			  client:
			    serviceUrl:
			      defaultZone: http://localhost:8081/eureka/
			server:
			  port: 8181
			spring:
			  application:
			    name: service-hi

    * 然后把消费者注册在8082
	cloud-service-ribbon 工程配置文件如下

			eureka:
			  client: 
			    serviceUrl: 
			      defaultZone: http://localhost:8082/eureka/
			server: 
			  port: 8281
			spring: 
			  application: 
			    name: service-ribbon

    * 此时无法从注册中心8082获取服务,提示无可用服务,说明两个注册中心确实是独立的,并没有实现HA

	    ![](/img/0008.png)

### 后来我又想到了会不会是hostname相同引起的冲突,于是修改了hostname配置

* 此时的配置文件如下
	* application-node1.yml :     
			server: 
			  port: 8081
			eureka: 
			  client: 
			    serviceUrl:
			      defaultZone: http://localhost:8082/eureka/
			  instance: 
			    hostname: node1

	* application-node2.yml :
			server: 
			  port: 8082
			eureka: 
			  client: 
			    serviceUrl:
			      defaultZone: http://localhost:8081/eureka/
			  instance: 
			    hostname: node2
* 启动后访问 http://localhost:8081/ 
  ![](/img/0009.png)
* 于是接着按上面的方法进行测试, 发现能够从8082找到8081中注册的服务
  ![](/img/0010.png)
 
* 接着进行HA测试
  * cloud-service-hi 工程配置文件如下
	
		eureka:
		  client:
		    serviceUrl:
		      defaultZone: http://localhost:8082/eureka/,http://localhost:8081/eureka/
		server:
		  port: 8181
		spring:
		  application:
		    name: service-hi

  * cloud-service-hi 工程配置文件如下

				eureka: 
				  client: 
				    serviceUrl: 
				      defaultZone: http://localhost:8081/eureka/,http://localhost:8082/eureka/
				server: 
				  port: 8281
				spring: 
				  application: 
				    name: service-ribbon
  * 此时只启动node1节点,能进行正常的注册和发布服务
   
### *结论* 
	试验结果表明这个坑很可能就是因为 hostname 冲突导致的,与绑定hosts无关...
	但是并不能肯定,毕竟才疏学浅,希望哪位大牛能给我解惑
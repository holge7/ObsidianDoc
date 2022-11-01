Until now, to call to another service we used to #WebClient with the url harcoded

```
.uri("http://localhost:8082/api/inventory/service", ...)
```
## Doc

This way has a problem, the problem is that in locally it can work OK, but normally when we have got a enviroment running in the cloud everything is dynamic including the ips so we cant have these ips harcoded because we dont know what they might be

![[Pasted image 20221101151208.png]]

To solve this problem comes Service Discovery pattern!!, it will take care of the ip table with all services

![[Pasted image 20221101151536.png]]

How this communication will be happend?? easy:
1. Service A call to Discovery server asking what is the ip of Service B
2. Discovery server return this ip of B to A
3. Now, A know where is the ip of B and can call to it

![[Pasted image 20221101151900.png]]

For curiosity, what happeds if Discovery service is not aviable??, ok, what happens is that in the firts call to discovery server it return a copy of it table of service-ip and if this is down for some reason, the client have a copy of it

![[Pasted image 20221101152245.png]]

## Prog

IMPORTANT NOTE: DS cant be a spring-boot project must be a spring project
err: Request execution error. endpoint=DefaultEndpoint{ serviceUrl=http://localhost:8761/eureka/}

The firts step in implementing a Discovery Server (DS) is to create a new module project for it, after it, we will use of Spring Cloud Netflix the library #Eureka 

Add in pom of discovery-server

```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
	<version>3.1.4</version>
</dependency>
```

And in the pom of parent project

```
    <dependencyManagement>
	    <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
	    </dependencies>
	</dependencyManagement>
```

After it, we can go to DiscoveryServerAplication.java and add `@EnableEurekaServer`
```
@SpringBootApplication
@EnableEurekaServer
public class DicoveryServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(DicoveryServerApplication.class, args);
	}

}

```

For other hand, we need add a couple of properties in .properties of discovery-server
```
eureka.instance.hostname=localhost
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
server.port=8761
```

We set 
- register-with-eureka=false, because we dont want that eureka register it own service as a service to be use with eureka
- fetch-registry=false, because we dont want it to bring a copy of its own records since it is the one that has it stored in memory

Next step is defing eureka client in each service

1. Add eureka-client dependency 
```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

2. Add `@EnableEurekaClient`
```
@SpringBootApplication
@EnableEurekaClient
public class OrderServiceApplication {
```

3. Add eureka default zone (.properties)
`eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka`
To the client could find eureka server

4.  Now we can restart all owns services (but firts we need start DS)

Finally, if all went OK, we should be this log in each client service 
![[Pasted image 20221101170948.png]]

We can check all these services in eureka dashboard http://localhost:8761/

![[Pasted image 20221101171241.png]]

we can see that Eureka server identified 3 instances of applications, but the name is UNKNOWN becouse we dont have defined any name for owns services, we only need set in each .properties of clients: `spring.application.name=name-of-the-service`

![[Pasted image 20221101171613.png]]

To test, we can declare multiples instances of one service and in a random port, to do it:

1. Random port: set server port in 0 (.properties)
`server.port=0`

![[Pasted image 20221101172242.png]]


2. Run the same project in pararell
(in eclipse example)
**Run** > **Run configurations...** > Duplicate instance
![[Pasted image 20221101174113.png]]
![[Pasted image 20221101174243.png]]

![[Pasted image 20221101174337.png]]

NOTA: en eureka dashboard deberian aparecer 2 instancias de este servico pero no aparencen y no se porque :/

Now look OK, but if we try to do a request to inventory-service in this case:
```
.uri("http://inventory-service/api/inventory/service",
		uriBuilder -> uriBuilder.queryParam("skuCodes", skuCodes).build()
	)
```

we will recive this error:
`java.net.UnknownHostException: Failed to resolve 'inventory-service' after 2 queries `

This happens becouse the client have multiples instances of one service and is confused becouse it dont know wich should to call, to fix it, we can implement that the service A will try to call to first instance of B, if this is not aviable, try to call to 2º instance of B, etc. This is named #Client_Side_Loadbalancing and to enable it, we need set a anotation in the @Bean of WebClient

```
@Bean
@LoadBalanced
public WebClient.Builder webClientBuilder() {
	return WebClient.builder();
}
```

Now would be work correctly :)

To try, you can shut down DS and see how now each service have a local copy of the table   
service-ip of DS
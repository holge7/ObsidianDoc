So far we have multiples services running in randoms ports, that for own backend is ok, we used #Client_Side_Loadbalancing #Eureka #WebClient  to solve it, but really we need one ip to call it since client side, becouse they dont know what ips we have right now.

The solution for this problem is create a API Gateway, this api know all ports where are running owns services

![[Pasted image 20221101211605.png]]

To do own implimentation of this API Gateway we will need to use Spring Cloud Gateway.

"This project provides a library for building an API Gateway on top of Spring WebFlux. Spring Cloud Gateway aims to provide a simple, yet effective way to route to APIs and provide cross cutting concerns to them such as: security, monitoring/metrics, and resiliency"

The firts step is create a new module project for it and add the dependencies for gateway and eureka-client (this last if we are working with it) in pom file:
```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

Enable EurekaClient
```
@SpringBootApplication
@EnableEurekaClient
public class ApiGatewayApplication {

	public static void main(String[] args) {
		SpringApplication.run(ApiGatewayApplication.class, args);
	}

}
```

Configure .properties
```
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka
spring.application.name=api-gateway

logging.level.root=INFO
logging.level.org.springframework.cloud.gateway.route.RouteDefinitionRouteLocator= INFO
logging.level.org.springframework.cloud.gateway= TRACE
```

Now, we want defind routes in .properties
```
## Product Service Route
spring.cloud.gateway.routes[0].id=product-service
spring.cloud.gateway.routes[0].uri=lb://product-service
spring.cloud.gateway.routes[0].predicates[0]=Path=/api/products

## Order Service Route
spring.cloud.gateway.routes[1].id=order-service
spring.cloud.gateway.routes[1].uri=lb://order-service
spring.cloud.gateway.routes[1].predicates[0]=Path=/api/order

## Inventory Service Route
spring.cloud.gateway.routes[2].id=inventory-service
spring.cloud.gateway.routes[2].uri=lb://inventory-service
spring.cloud.gateway.routes[2].predicates[0]=Path=/api/inventory
```

Up the project and now, you can try some call from postma, example
http://localhost:8081/api/order

And... buhala, this is working and this is so easy :))))))
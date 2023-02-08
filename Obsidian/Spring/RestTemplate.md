## Documentacion

RestTemplate es una clase en Spring que proporciona una forma fácil y sencilla de realizar solicitudes HTTP y obtener respuestas. Es un cliente HTTP que se utiliza para consumir servicios REST.

## Prerequisitos

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## Tecnologias

## Configuración

Para tener un @Bean de #RestTemplate tendremos que definir en una clase de configuración, algo como:

```java
@Configuration
public class Config {
	
	@Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
	
}
```

Ya podremos usar lo básico, a partir de ahí podremos añadir cada vez más configuraciones a nuestro cliente HTTP como timeouts, etc. Usando un restTemplate Builder:

```java
@Bean 
public RestTemplate restTemplate(RestTemplateBuilder builder) { 
	return builder
		.setConnectTimeout(Duration.ofMillis(3000))
		.setReadTimeout(Duration.ofMillis(3000)) .build(); 
}
```


## Coding

Aquí un ejemplos de petición usando restTemplate:

##### GET
```java
@GetMapping("users")
public ResponseEntity<List<User>> getAll() { ... }

@GetMapping("users/{id}")
public ResponseEntity<User> getById(@PathVariable long id) { ... }
```


*~ API Response como JSON ~*

Podemos usar getForEntity() para obtener toda la respuesta (data, cabeceras, etc.)
```java
ResponseEntity<String> responseEntity = restTemplate.getForEntity("/users/{id}", String.class, Map.of("id", "1")); 

User user = gson.fromJson(responseEntity.getBody(), User.class);
```

Usando getForObject() obtenemos directamente la data
```java
String userJson = restTemplate.getForObject("/users/{id}", String.class, Map.of("id", "1"));
```


*~ API Response como POJO ~*

Podemos hacer una peticion y que nos retorne directamente la clase pojo sin tener que parsearla de un JSON:

```java
User[] usersArray = restTemplate.getForObject("/users", User[].class); 

User user = restTemplate.getForObject("/users/{id}", User.class, Map.of("id", "1"));
```


Y tambien con el ResponseEntity:

```java
ResponseEntity<User[]> responseEntity = restTemplate.getForEntity("/users", User[].class);

ResponseEntity<User> responseEntityUser = restTemplate.getForEntity("/users/{id}", User.class, Map.of("id", "1"));
```
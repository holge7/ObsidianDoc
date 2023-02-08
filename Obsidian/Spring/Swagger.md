Para crear la documentacion con swagger en spring, debemos usar las siguientes dependencias:


```xml
	   <dependency>
	      <groupId>org.springdoc</groupId>
	      <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
	      <version>2.0.2</version>
	   </dependency>
```

Y aniadir al .properties:
```
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.api-docs.path=/v3/api-docs
```

Despues de esto, ya podremos acceder a nuestra documentacion con las rutas:

- host:port/swagger-ui.html
- host:port/v3/api-docs



Para empezar a customizar

el header:
```java
@Configuration
public class SwaggerConfig {
    
	@Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("Sample REST API for centalized documentation using Spring Boot and spring-fox swagger" +
                                " 3")
                        .version("2.8.5")
                        .description("app description")
                        .termsOfService("http://swagger.io/terms/")
                        .license(new License().name("Apache 2.0").url("http://springdoc.org")));

    }

}

```

Y para los endpoints se puede usar algo como esto:
```java 
public interface UserController {
	
//	ApiResponse

	
	@Operation(
			summary = "Login with `email` and `password`", 
			description = "Returns userDTO and JWT for the user auth", 
			responses = {
					@ApiResponse(
							responseCode = "200", 
							description = "Returns `userDTO` and `JWT` for the user auth",
							content = @Content(schema = @Schema(implementation = JwtResponse.class))),
					@ApiResponse(
							responseCode = "400", 
							description = "Invalid request", 
							content = @Content(schema = @Schema(implementation = commons.utils.ApiResponse.class))),
					@ApiResponse(
							responseCode = "500", 
							description = "Internal server error", 
							content = @Content(schema = @Schema(implementation = commons.utils.ApiResponse.class)))
					}
			)
	public ResponseEntity<commons.utils.ApiResponse> login(@RequestBody LoginRequest user);
```


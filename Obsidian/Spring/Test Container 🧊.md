#Test #Docker #Junit

Si queremos ejecutar los test en una bd de prueba, una buena idea es usar test containers, estos aprovechand docker para levantar una bd alli y ejecutar los tests en ella


### Prerequisitos
- Docker
- JUnit > 4

### Instalacion
.pom: https://www.testcontainers.org/
```java
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>mysql</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
	<groupId>org.testcontainers</groupId>
	<artifactId>junit-jupiter</artifactId>
	<version>1.17.5</version>
	<scope>test</scope>
</dependency>


<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>testcontainers-bom</artifactId>
            <version>1.17.5</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

```

### Configuracion
Unicamente debemos crear una nueva clase para nuestros test y anotarla con @Testcontainers entre otros:
```java
@SpringBootTest
@Testcontainers
@AutoConfigureMockMvc
public class AuthControllerTests {
	....
}
```

y dentro de esta clase, crear un metodo con la anotación @DynamicPropertySource, esta para que podamos inyectar en el .properties la url donde vamos a dockerizar los servicios que queramos, en este caso una bd, se muestra a continuación.

### Uso
Para este ejemplo vamos a dockerizar una bd mysql sobre la que lanzaremos nuestros tests, para ello nos hace falta:

Crear el container con @Container y especificar los datos de la bd, en este caso:
```java
@Container
private static final MySQLContainer mysqlContainer = new MySQLContainer("mysql:latest")
	.withDatabaseName("auth-service")
	.withUsername("root")
	.withPassword("root");
```

Despues de esto, como se dijo en el apartado de configuración, debemos colocar estas propiedades en el .properties de manera dinamica, usando @DynamicPropertySource, ejemplo:
```java
@DynamicPropertySource
static void setProperties(DynamicPropertyRegistry dynamicPropertyRegistry) {
	dynamicPropertyRegistry.add("spring.datasource.url", mysqlContainer::getJdbcUrl);
}
```

Y listo, ya podemos ejecutar nuestros tests dockerizados :)
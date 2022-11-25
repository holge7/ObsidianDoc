### Documentacion

Los tests unitarios consisten en testear pequeñas partes de nuestra aplicacion, como puede ser una funcion, estas partes son llamadas units y se tienen que examinar de forma individual e independient para su correcto funcionamiento (normalmente estos tests se aplican durant el proceso de desarrollo).

Alguien importante (Michael Feathers in 2005) dijo que:
Un test no es un unit test si:

- Se comunica con la bd
- Se comunica a traves de la red
- Toca el sistema de archivos
- No se puede correr al mismo tiempo que otros unit tests
- Tienes que hacer cosas especiales como editar config files para correrlos

## Prerequisitos
- Docker
- JUnit > 4

## Tecnologias
#Junit #TestContainers #Mockito

## Instalacion

## Coding

Aqui hay un ejemplo de validar el unit para login
```java
@SpringBootTest
class ApiAuthApplicationTests {
	
	@Autowired
	UserRepository userRepository;
	
	@Autowired
	UserService userService;

	@Test
	void login_user() {
		
		String email = "jorge@gmail.com";
		String password = "password";
		Set<Rol> roles = new HashSet<>();
		roles.add(new Rol(ERol.ROLE_USER));
		
		// Save user
		User user = new User(email, "Jorge", password, roles);
		User userSave = userRepository.save(user);
		
		// Loggin user
		LoginRequest loginRequest = new LoginRequest(email, password);
		ResponseEntity<ApiResponse> response = userService.login(loginRequest);
		
		assertThat(response.status(HttpStatus.OK));
		
	}

}
```

Pero en este tests hay varias cosas mal:
- La anotación @SpringBootTest, esta etiqueta carga todos los contextos de nuestra aplicacion para los tests.
- Estamos accediendo a la capa de bd para registrar un usuario

Entonces que debemos hacer para aislar completamente estos tests? La repuesta es #Mockear los datos, cada vez que usemos un @MockBean, Spring crara un nuevo contexto de la aplicacion en los tests (Crear un nuevo contexto, aumenta el tiempo de ejecucion de las pruebas).

Bueno y con la etiqueta de @SpringBootTest que hacemos?, crearemos servicios #Unit-Testeable
esto quiere decir que en nuestros servicios y clases que vayamos a testear debemos evitar usar inyeccion de dependencias por medio de @Autowired y pasar a usarla por inyeccion del constructor de forma que sea posible pasar mocks a nuestro servicio
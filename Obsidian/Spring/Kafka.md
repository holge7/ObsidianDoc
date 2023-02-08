## Tecnologias
#ColaMensajería #Kafka #ApacheKafka

## Documentacion

Apache #kafka es un sistema distribuido de colas de mensajes diseñado para procesar grandes volumenes de datos en tiempo real.

Kafka funciona como una plataforma de publicación-suscripción, donde los productores publican mensajes en tópicos y los suscriptores se suscriben y los procesan.

Los componentes principales de kafka son:

- Publisher/Producer: Entidades que publican los mensajes con la información.
- Subscriber/Consumer: Entidades subscritas a un topico y que se mantienen escuchando hasta que llega un nuevo mensaje para procesarlo.
- Broker: Maquina individual donde corre kafka.
- Topic: Canal donde se publican los mensajes.
- Paritions: Son subdivisiones de un Topic.


![[kafka.svg]]





## Prerequisitos

```xml
<dependency>
	<groupId>org.springframework.kafka</groupId>
	<artifactId>spring-kafka</artifactId>
</dependency>
```


## Configuración

Para configurar kafka desde el .properties:

```properties
# Kafka Properties
spring.kafka.consumer.properties.spring.json.trusted.packages= *

# Consumer
spring.kafka.consumer.bootstrap-servers= localhost:9092
spring.kafka.consumer.group-id= group-id
spring.kafka.consumer.auto-offset-reset= earliest
spring.kafka.consumer.key-deserializer= org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer= org.springframework.kafka.support.serializer.JsonDeserializer

# Producer
spring.kafka.producer.bootstrap-servers=localhost:9092
spring.kafka.producer.key-serializer= org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer= org.springframework.kafka.support.serializer.JsonSerializer

```

- Permite confiar en todas los packages para hacer el parseo de json a X clase directamente
	spring.kafka.consumer.properties.spring.json.trusted.package

- Indicamos donde esta el servidor kafka
	bootstrap-servers

- Indica el groupID
	group-id

- Configuración que controla como el consumidor maneja el inicio de lectura de un mensaje en un tópico (valores: `earliest`, `latest` y `none`)

	spring.kafka.consumer.auto-offset-reset

	El valor `earliest` significa que el consumidor comenzará a leer mensajes desde el inicio de los registros disponibles en el tópico. (Cuando se quiere procesar todos los mensajes antiguos disponibles en el tópico, independientemente de si ya se han leído o no)

	El valor `latest` significa que el consumidor comenzará a leer mensajes desde el final de los registros disponibles en el tópico. (Esto es útil cuando se quiere procesar solo los mensajes nuevos que se publican en el tópico)

	El valor `none` significa que el consumidor no se iniciará si no se encuentra una posición válida de desplazamiento para el tópico y el grupo de consumidores asociado. (Esto puede ser útil si se quiere tener control manual completo sobre la posición de desplazamiento del consumidor)

- Con que clase se van a descerializar los mensajes
	org.apache.kafka.common.serialization.StringDeserializer
	
- Con que clase se van a serializar los mensajes
	value-deserializer


## Coding

Aquí un ejemplo de una comunicación entre los MS: 

Crear un bean:
```java
@Configuration
public class Kafka {
	
	@Bean
    public NewTopic topic() {
        return TopicBuilder.name("login")
                .partitions(10)
                .replicas(1)
                .build();
    }
}
```

 - Login: recibe peticiones de login y genera JWT
 - User: valida que el email/password existan y retorna los datos del usuario

MS Login (Controller)
```java
	@PostMapping("login")
	public CompletableFuture<ResponseEntity<String>> login(
			@RequestBody String email,
			@RequestBody String password) {
		return loginService.login(email, password);
	}
```

MS Login (Service)

```java

// Variable para almacenar las CompletableFuture
private Map<String, CompletableFuture<ResponseEntity<String>>> futures = new HashMap<>();

// (Publisher)
// Encargado de crear un CompletableFuture y almacenarlo en la variable  de clase,     // además envía el mensaje por el tópico de kafka 
public CompletableFuture<ResponseEntity<String>> login(String email, String password) {
	HashMap<String, String> user = new HashMap<>();
	user.put("email", email);
	user.put("password", password);
	
	CompletableFuture<ResponseEntity<String>> future = new CompletableFuture<>();
	futures.put("1", future);
	
	log.debug("Sending login request to user service...");
	kafkaTemplate.send("login-request-topic", user);

	return future;
}

// (Subscriptor)
// Este metodo esta subscrito al tópico "login-response-topic", entonces cuando se     // publique algún mensaje por ahí, se procesara 
@KafkaListener(topics = "login-response-topic", groupId = "group_id")
public void loginListen(ConsumerRecord<String, ApiResponse> cr) {
	ApiResponse apiResponse = cr.value();

	if (apiResponse.isError()) {
		log.error(apiResponse.getMsg());
		throw new CustomGenericalException(apiResponse.getMsg(), HttpStatus.INTERNAL_SERVER_ERROR);
	}

	// We need parse the data becouse comes as LinkedHashMap
	UserDTO userDTO = gson.fromJson(gson.toJson(apiResponse.getData()), UserDTO.class);

	String tokenString = jwtUtils.generateJwtToken(userDTO);
	
	ResponseEntity<String> responseString = 
			ResponseEntity.ok(tokenString);
	
	CompletableFuture<ResponseEntity<String>> future = futures.get("1");
	future.complete(responseString);
}	
```


MS User (Consumer)

```java
// Atributo de clase
private KafkaTemplate<String, Object> kafkaTemplate;

// Esta suscrito a ese topic y tambien actua como publisher del otro topic
@KafkaListener(topics = "login-request-topic", groupId = "group_id")
public void consume(HashMap<String, String> msg) {
	UserDTO userDTO = userService.login(msg.get("email"), msg.get("password"));
	ApiResponse response = new ApiResponse(false, "", userDTO);
	kafkaTemplate.send("login-response-topic", response);
}
    
```
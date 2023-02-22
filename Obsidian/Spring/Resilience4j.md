## Documentacion

#Resilience4j es una biblioteca de tolerancia a fallos para Java, Se utiliza para mejorar la resiliencia y robustez de aplicaciones Java que están expuestas a errores, interrupciones y congestiónes en el sistema. 

En resumen, Resilience4j nos otorga un patron #Circuit_breaker listo para solo configurarlo a nuestro gusto.

Resilience4j es muy amplio y otorga numerosos cores:

- resilience4j-circuitbreaker
- resilience4j-ratelimiter
- resilience4j-bulkhead
- resilience4j-retry
- resilience4j-cache
- resilience4j-timelimiter



## Prerequisitos

Al menos 2 servicios intercomunicados de forma síncrona.

## Tecnologias

#Resilience4j #circuit_breaker 


## Instalación

```xml
<dependency>
	<groupId>io.github.resilience4j</groupId>
	<artifactId>resilience4j-spring-boot3</artifactId>
	<version>2.0.2</version>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```


## Configuración

##### #Actuator
```
# RESILIENCE http://localhost:8080/actuator/health
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
management.health.circuitbreakers.enabled=true
```


##### #Circuitbreaker

```
resilience4j.circuitbreaker.instances.serviceA.registerHealthIndicator=true
resilience4j.circuitbreaker.instances.serviceA.event-consumer-buffer-size=10 
resilience4j.circuitbreaker.instances.serviceA.sliding-window-type=COUNT_BASED
resilience4j.circuitbreaker.instances.serviceA.sliding-window-size=5
resilience4j.circuitbreaker.instances.serviceA.failure-rate-threshold=50
resilience4j.circuitbreaker.instances.serviceA.wait-duration-in-open-state=5s
resilience4j.circuitbreaker.instances.serviceA.permitted-number-of-calls-in-half-open-state=3
resilience4j.circuitbreaker.instances.serviceA.permittedNumberOfCallsInHalfOpenState=3
resilience4j.circuitbreaker.instances.serviceA.automatic-transition-from-open-to-half-open-enabled=true

resilience4j.circuitbreakertimelimiter.instances.serviceA.timeout-duration=3s
```

1. `resilience4j.circuitbreaker.instances.serviceA.registerHealthIndicator=true`
Esta línea habilita el indicador de salud para el Circuit Breaker con nombre `serviceA`. Esto permite que la aplicación recopile y muestre información sobre el estado y el rendimiento del Circuit Breaker en la aplicación.

2. `resilience4j.circuitbreaker.instances.serviceA.event-consumer-buffer-size=10`
Esta línea de configuración se refiere al tamaño del búfer del consumidor de eventos para el circuito de "serviceA". El búfer de eventos es una estructura de datos en la memoria que almacena los eventos generados por el circuito "serviceA" hasta que puedan ser procesados y utilizados para mantener una visión actualizada del estado del circuito. Especificar un tamaño de búfer de 10 significa que solo se mantendrán los últimos 10 eventos generados por el circuito "serviceA", y los eventos más antiguos se descartarán para liberar memoria.

3. `resilience4j.circuitbreaker.instances.serviceA.sliding-window-type=COUNT_BASED`
Esta línea establece el tipo de ventana deslizante para el Circuit Breaker con nombre `serviceA`. `COUNT_BASED` significa que la ventana deslizante se basa en un número de llamadas a la operación, no en el tiempo transcurrido.

4. `resilience4j.circuitbreaker.instances.serviceA.sliding-window-size=5`
Esta línea establece el tamaño de la ventana deslizante para el Circuit Breaker con nombre `serviceA`. La ventana deslizante se utiliza para contar las llamadas fallidas a la operación y determinar el porcentaje de llamadas fallidas.

5. `resilience4j.circuitbreaker.instances.serviceA.failure-rate-threshold=50`
Esta línea establece el umbral de tasa de fallos para el Circuit Breaker con nombre `serviceA`. Si la tasa de fallos de llamadas a la operación supera este umbral, el Circuit Breaker cambiará a un estado abierto y detendrá temporalmente las llamadas a la operación.

6. `resilience4j.circuitbreaker.instances.serviceA.wait-duration-in-open-state=5s`
Esta línea establece la duración de espera en el estado abierto para el Circuit Breaker con nombre `serviceA`. Durante este tiempo, el Circuit Breaker detendrá temporalmente las llamadas a la operación.

7. `resilience4j.circuitbreaker.instances.serviceA.permitted-number-of-calls-in-half-open-state=3` 
Esta línea establece el número permitido de llamadas en el estado semabierto para el Circuit Breaker con nombre 

8. `resilience4j.circuitbreakertimelimiter.instances.serviceA.timeout-duration=3s`
Esta línea de configuración se refiere al tiempo de espera máximo para el circuito de "serviceA" cuando se utiliza el módulo de tiempo de espera de Resilience4j. El tiempo de espera es el tiempo máximo que una solicitud puede estar en espera de una respuesta antes de que se considere que ha fallado

## Coding

Aquí un ejemplo de como anotaríamos un endpoint con un circuit breaker que redirije el flujo del fallback a otra función:
```java
@GetMapping("failure")
@CircuitBreaker(name = BACKEND_B, fallbackMethod = "fallback")
@Bulkhead(name = BACKEND_B)
@Retry(name = BACKEND_B)
public String failure() {
	return restTemplate.getForObject(PATH_STRING+"failure", String.class);
}

public String fallback(HttpServerErrorException ex) {
	log.warn("FallBack Method!!!!");
	return "Hello from fallback method :)";
}
```

## Documentacion

Zipkin es una herramienta de trazabilidad distribuida que ayuda a monitorear y depurar aplicaciones de microservicios.

## Prerequisitos
Servidor de zipkin

## Tecnologias
#Zipkin

## Instalacion

```xml
<dependency>
	<groupId>io.micrometer</groupId>
	<artifactId>micrometer-observation</artifactId>
</dependency>
<dependency>
	<groupId>io.micrometer</groupId>
	<artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>
<dependency>
	<groupId>io.zipkin.reporter2</groupId>
	<artifactId>zipkin-reporter-brave</artifactId>
</dependency>
<dependency>
	<groupId>io.micrometer</groupId>
	<artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```properties
# Zipkin
spring.zipkin.base-url=http://localhost:9411
management.tracing.sampling.probability=1.0
logging.pattern.level=%5p [${spring.application.name:},%X{traceId:-},%X{spanId:-}]
```

## Coding
No hace falta :)
## Documentacion

Bien, ahora nos suponemos que queremos implementar autenticacion en una arquitectura de microservicios, como lo hariamos?, pues en este ejemplo lo voy a hacer con JWT y siguiendo el siguiente diagrama de flujo en el que se detalla como realizaremos las peticiones y que servicio es el encargado de cada cosa

![[aut-microservice-gateway-jwt.svg]]

El esquema cuenta de 3 microservicios:
- Gateway: encargado de redirigir el trafico y validar los jwt.
- Auth-service: acciones de loggeo, registro, generacion de jwt, etc.
- Resource: api con recursos privados y publicos a los que queremos acceder.

## Prerequisitos
- Auth service
- Discovery service
- Gateway
- Some service with resources

## Tecnologias
#JWT #Auth-service #gateway

## Instalacion

Necesitaremos tener en nuestro pom del gatwey:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
<!-- https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt -->
<dependency>
	<groupId>io.jsonwebtoken</groupId>
	<artifactId>jjwt</artifactId>
	<version>0.9.1</version>
</dependency>
```


## Coding
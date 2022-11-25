## Documentacion

Bien, ahora nos suponemos que queremos implementar autenticacion en una arquitectura de microservicios, como lo hariamos?, pues en este ejemplo lo voy a hacer con JWT y siguiendo el siguiente diagrama de flujo en el que se detalla como realizaremos las peticiones y que servicio es el encargado de cada cosa

![[aut-microservice-gateway-jwt.svg]]

El esquema cuenta de 3 microservicios:
- Gateway: encargado de redirigir el trafico y validar los jwt.
- Auth-service: acciones de loggeo, registro, generacion de jwt, etc.
- Resource: api con recursos privados y publicos a los que queremos acceder.

## Prerequisitos

## Tecnologias
#JWT #Auth-service #gateway

## Instalacion

## Coding
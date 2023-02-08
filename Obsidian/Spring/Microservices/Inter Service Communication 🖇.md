## Documentacion

La Comunicación entre Microservicios debe ser eficaz y robusta, existen 2 tipos de comunicación, sincrona y asincrona, dependiendo de nuestros requisitos deberemos usar una u otra.

#### ¿Cuándo usar sincronía?

- Es necesario una respuesta inmediata y confiable de otro MS.
- Complejidad de operación baja y tiempo de respuesta crítico.
- En sistemas de transacciónes donde la consistencia de los datos es fundamental.

Por ejemplo: en el caso de que tengamos un comercio electrónico y queramos comprobar nuestro stock para validar una compra, aquí el MS de facturación tiene que llamar y esperar al MS de stock para conocer la disponibilidad del producto

#### ¿Cuándo usar asincronía?

- El tiempo de respuesta no es muy crítico y la operación compleja.
- Para mejorar el rendimiento/escalabilidad del sistema al delegar tareas a otros MS de manera no bloqueante.

Por ejemplo:  en el ámbito bancario, la solicitud de préstamo debe procesarse y necesita la aprobación en múltiples niveles. Entonces en este caso cuando un usuario presente una solicitud de préstamo, el servicio de solicitud de préstamo proporcionará un número de referencia de inmediato. Una vez realizada todas las aprobaciones, persistirán los detalles de la solicitud de préstamo en la base de datos. Entonces, en este escenario, podemos usar la comunicación asíncrona.


En general, la comunicación asincrónica es más adecuada para tareas que pueden ser ejecutadas de manera independiente y en segundo plano, mientras que la comunicación sincrónica es adecuada para tareas que requieren una respuesta inmediata.

## Prerequisitos

Al menos 2 microservicios para realizar intercomunicaciónes.

## Tecnologias

##### Síncronas:
- [[RestTemplate]]

#RestAPI #HTTP #Loadbalancing #FeignClient 

##### Asíncronas:
- [[Kafka]]

#ColaMensajería #Kafka #RabbitMQ


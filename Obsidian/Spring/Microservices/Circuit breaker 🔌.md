## Documentacion

En una arquitectura de microservicios es común llamar de forma sincrona desde un servicio A a un servicio B, C, D, etc. Esto siempre conlleva un riesgo, y es que puede que al servicio que estamos llamando se encuentre en mantenimiento, caido, tenga demasiada carga y las peticiones tarden demasiado tiempo en responderse... Esto estaría bloqueando un hilo de nuestro servicio A por un tiempo demasiado valioso, pero entonces como podemos solucionar esto?

Podemos aplicar un patrón de #circuit_breaker que permite manejar la falla en un microservicios en un sistema distribuido a la hora de realizar llamadas sincronas a servicios externos. 

Un circuit breaker para gestionar los errores lo hace mediante estados, generalmente tiene 3 estados diferentes: "Colsed", "Open" y "Half-Open".

- Closed: El circuito permite que las solicitudes pase a través de él y sean procesadas como normalmente.
- Open: El circuito detiene automáticamente la ejecución de las solicitudes y no permite que pasen a través de él.
- Half-Open: En este estado, el circuito permite que algunas solicitudes sean procesadas de forma limitada, con el fin de determinar si el servicio externo ha recuperado su funcionamiento normal o no. Si las solicitudes se procesan correctamente, el circuito pasa a "Close", de lo contrario el circuito vuelve a su estado "Open".

![[circuit breaker.svg]]



## IMPLEMENTACIÓN

Existen principalmente 2 formas de implementar un circuit breaker en una arquitectura de microservicios:

- Implementación en cada servicio.
- Implementación en el gateway.

Implementarlo en un sitio o en otro dependera de nuestros requisitos, si queremos un mayor control y visibilidad sobre cada microservicio individual, podemos implementarlo en cada servicio, permitiendo controlar la disponibilidad de cada servicio de forma aislada.

Sin embargo, si deseamos tener una gestión centralizada y uniforme de la disponibilidad de los servicios, se puede implementar el circuit breaker en la gateway, esto puede simplificar el mantenimiento, administración y monitoreo de la disponibilidad de los servicios ya que solo se necesita un único punto de control en lugar de varios.

En resumen, la implementación depende de las necesidades del proyecto, ambos tienen sus propios enfoques, ventajas y desventajas, es importante evaluar previamente los requisitos para decidir cuál  es el mejor enfoque para su sistema.

## Prerequisitos

Al menos 2 microservicios para realizar intercomunicaciónes.

## Tecnologias

[[Resilience4j]] , Hystrix (desactualizada)
## Documentacion

Cucumber es un framework con el que se pueden hacer tests de integración de una forma rapida y facil, este framework, ademas usa Gherkin como lenguaje para definir "escenarios" a los que se le pueden añadir multiples "examples"

![[Pasted image 20221119225242.png]]

## Prerequisitos

- cucumber-java
- cucumber-junit
- cucumber-spring

```java
<!-- https://mvnrepository.com/artifact/io.cucumber/cucumber-java -->
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-java</artifactId>
    <version>7.9.0</version>
</dependency>

<!-- https://mvnrepository.com/artifact/io.cucumber/cucumber-junit -->
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-junit</artifactId>
    <version>7.9.0</version>
    <scope>test</scope>
</dependency>

<!-- https://mvnrepository.com/artifact/io.cucumber/cucumber-spring -->
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-spring</artifactId>
    <version>7.9.0</version>
</dependency>
```

## Tecnologias

#Cucumber #Gherkin #Junit 

## Instalacion

## Coding

Lo que necesitamos para correr unos tests con Cucumber con Junit es añadir la anotacion @RunWith(Cucumber.class) y indicar donde tenemos el feature (steps) @CucumberOptions(features = "src/test/testFeature")

Para explicarlo mejor, vamos a realizar un ejemplo de una llamada a la api de registrar un usuario:

1. Crear el feature en src/test/resource/auth.feature
```Gherkin
Feature: Auth controller
  I want to use this template for my auth controller

  Scenario Outline: Client makes register call to /auth/register
    Given I want to given a user
    When I register a user
    Then I verify the register
```

2. Debemos integrar los tests de cucumber con el contexto de la aplicacion de spring
```java
@CucumberContextConfiguration
@SpringBootTest
public class SpringIntegrationTest {

}
```

3. Crear la clase del test con las anotaciones correspondientes, extendiendo de SpringIntegrationTest para obtener el contexto y especificando los pasos que definimos en el .feature
```java
@RunWith(Cucumber.class)
@CucumberOptions(features = "src/test/auth")
public class AuthController {

}
```

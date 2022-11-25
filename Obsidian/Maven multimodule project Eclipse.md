1. Creamos un nuevo proyecto de maven:
![[Pasted image 20221125163850.png]]
![[Pasted image 20221125163933.png]]
![[Pasted image 20221125164044.png]]

2. Configurar el pom del proyecto parent maven
```java
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  	<modelVersion>4.0.0</modelVersion>
  
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.7.5</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
  
  	<groupId>auth</groupId>
  	<artifactId>door</artifactId>
  	<version>0.0.1-SNAPSHOT</version>
  	<packaging>pom</packaging>
  	
  	<properties>
		<spring-cloud.version>2021.0.4</spring-cloud.version>
	</properties>
	
	<modules>
	
	</modules>
	
	<dependencyManagement>
	    <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
	    </dependencies>
	</dependencyManagement>
	
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
	
</project>
```


3. Ahora crearemos el primer modulo de nuestro proyecto

IMPORTANTE: crear el nuevo proyecto dentro de la ruta del proyecto principal maven (En este caso va a ser un starter project de spring)
![[Pasted image 20221125164823.png]]

4. Añadimos el nuevo modulo al pom del maven project parent
```java
<modules>
	<module>auth-service</module>
</modules>
```
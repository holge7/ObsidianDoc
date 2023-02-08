
Para crear una libreria en JAVA y desplegarla en los diferentes proyectos, usaremos este comando sobre la libreria

mvn package

Esto creara un .jar de la aplicacion en:    nombre-aplicacion/target/nombre-aplicacion-version.jar

Ahora solo nos tendremos que mover a la aplicacion en la que queramos instalar esta libreria y ejecutar este otro comando de mvn

mvn install:install-file -Dfile=D:\ProgrammingEnviroment\Java\Spring\Auth-door\commons\target\commons-1.0.1.jar -DgroupId=com -DartifactId=commons -Dversion="1.0.1" -Dpackaging=jar -DgeneratePom=true

evidentemente cambiando los nombres, rutas, versiones, etc.

Y por ultimo aniadir la dependencia al pom:

		<dependency>
		    <groupId>com</groupId>
		    <artifactId>commons</artifactId>
		    <version>1.0.1</version>
		</dependency>
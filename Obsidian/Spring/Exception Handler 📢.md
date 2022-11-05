How we can controll all exceptions in own app without 300000 try/catch??, one way is use a global Exception Handler, with his we can do it more easy and with bit code. (this is one way, we have more options to do it).

First of all, we will need to create a class that will handle all our exceptions and that class must be annoted with @ControllAdvice and exteds of  ResponseEntityExceptionHandler 

```java
@ControllerAdvice
public class ExceptionHandlerImpl extends ResponseEntityExceptionHandler {
	...
}
```

Now, in this class, we will defind all ExeptionHandler that we want to handle, to add a new exception that will handle, we only need create a new method and annotate his with 

*@ExceptionHandler(myClass.class)* or *@ExceptionHandler({myClass.class, myClass2.class})*

```java
@ControllerAdvice
public class ExceptionHandlerImpl extends ResponseEntityExceptionHandler {

	@ExceptionHandler(RolException.class)
	public ResponseEntity<ApiResponse> handleApiException(RolException rolException){
		....
	}
	
}
```

And when we will have a new Exception of type AuthenticationException or ApiException this class take care of it and will do what we have in this method ;)
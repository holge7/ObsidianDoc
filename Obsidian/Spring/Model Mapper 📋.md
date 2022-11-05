#ModelMapper #Mapper #DTO #Entity


What we need if we want transform one Entity in DTO object?? copy manuality all his properties, isn't it? or we could use better Model Mapper, this is a library to easy to use and configure, we only need, add it in .pom:

### Install
```
<!-- https://mvnrepository.com/artifact/org.modelmapper/modelmapper -->
<dependency>
	<groupId>org.modelmapper</groupId>
	<artifactId>modelmapper</artifactId>
	<version>3.1.0</version>
</dependency>
```

### Configure
Create a @bean of the mapper in one Configuration.class or similar.

```java
@Configuration
public class Beans {
	
	@Bean
	public ModelMapper modelMapper() {
		return new ModelMapper();
	}
	
}
```

### Use
And it is all, now all that remains is to use it, its very simple, we only need put as first param the class we want to convert and as second param the class to wich we want conert.

```java
@Component
public class UserMapper {
	
	private ModelMapper modelMapper;
	
	public UserMapper(ModelMapper modelMapper) {
		this.modelMapper = modelMapper;
	}
	
	public UserDTO userDTO(User user) {
		return modelMapper.map(user, UserDTO.class);
	}
	
}
```





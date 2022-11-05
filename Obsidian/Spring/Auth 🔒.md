For own auth service we'll use a model based in Users-Roles and JWT's, let's start:

### Users-Roles

To create this structure, we could use the next entities relationship

![[Pasted image 20221104224309.png]]

This is very simple to do with spring, we only need:

- Rol entity:
```
@Entity
public class Rol {
	
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	public long id;
	public String rol;
	
	...
}
```

- User entity:
```
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	public long id;
	public String email;
	public String name;
	public String password;
	
	@ManyToMany(fetch = FetchType.EAGER)
	public Set<Rol> rol = new HashSet<>();
	
	...
}
```



Right!, to start in code, we could create first the "customAuthenticationManager.java" wich would be own class with generical methos to use in auth with a method to do the password encode:

```
@Component
public class CustomAuthenticationManager {
	
	@Bean
	PasswordEncoder passwordEncoder() {
		return new BCryptPasswordEncoder();
	}
	
}
```
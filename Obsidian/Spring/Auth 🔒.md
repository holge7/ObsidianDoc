For own auth service we'll use a model based in Users-Roles and JWT's, let's start:

## TEORIA

### Users-Roles

To create this structure, we could use the next entities relationship

![[Pasted image 20221104224309.png]]

Diagrama de flujo para Registro/Login/Acceso a recursos
![[Captura.jpg]]


Diagrama de flujo de como funciona la autenticacion con jwt
![[Captura2.jpg]]

## IMPLEMENTACION

### INSTALACION

IMPORTANTE: spring security, trae por defecto ciertas configuraciones, como cors, etc. que si no sabemos cuales son y las modificamos podemos tener problemas de unauthorized, etc. para desactivar estas configuraciones debemos añadir en el Application.java @SpringBootApplication(exclude = {SecurityAutoConfiguration.class })

```java
@SpringBootApplication(exclude = {SecurityAutoConfiguration.class })
public class ApiAuthApplication {

	public static void main(String[] args) {
		SpringApplication.run(ApiAuthApplication.class, args);
	}

}
```

Para usar todas las funciones de spring security, debemos definir sus dependencias en el .pom

```java
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-test</artifactId>
	<scope>test</scope>
</dependency>
```


### MODELOS/PAYLOADS

This is very simple to do with spring, we only need:

#### MODELOS
- Rol enum:
```java
public enum ERol {
  ROLE_USER,
  ROLE_MODERATOR,
  ROLE_ADMIN
}
```

- Rol entity:
```java
@Entity
public class Rol {
	
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	public long id;

	@Enumerated(EnumType.STRING)
	public ERol rol;
	
	...
}
```

- User entity:
```java
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

#### PAYLOADS
```java
public class LoginRequest {
	
	public String username;
	public String password;
	
	...
}

public class SignupRequest {
	
	public String username;
	public String email;
	public Set<String> role;
	public String password;
	
	...
}

public class JwtResponse {
	
	  public String token;
	  public String type = "Bearer";
	  public Long id;
	  public String username;
	  public String email;
	  public List<String> roles;

	...
}
```


### CONFIGURACIÓN

1. Para empezar, podemos crear la clase WebSecurityConfig donde declararemos todos nuestros beans (los he pegado todos menos los cors, seguramente de alguno fallo, comentar y descomentar conforme vayan haciendo falta :) )

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig {
	
	@Autowired
	UserDetailsServiceImpl userDetailsService;

	@Autowired
	private AuthEntryPointJwt unauthorizedHandler;

	@Bean
	PasswordEncoder passwordEncoder() {
		return new BCryptPasswordEncoder();
	}

	@Bean
	public AuthTokenFilter authenticationJwtTokenFilter() {
		return new AuthTokenFilter();
	}

	@Bean
	public DaoAuthenticationProvider authenticationProvider() {
		DaoAuthenticationProvider authProvider = new DaoAuthenticationProvider();

		authProvider.setUserDetailsService(userDetailsService);
		authProvider.setPasswordEncoder(passwordEncoder());

		return authProvider;
	}
	
}
```

- passwordEncoder: permitira encriptar las contraseñas de nuestros usuarios

2. Crear la clase "UserDetailsImpl" que implemantara UserDetails y que usaremos para manejar todos los detalles de un usuario:

```java
public class UserDetailsImpl implements UserDetails {
	private static final long serialVersionUID = 1L;
	
	private Long idLong;
	private String username;
	private String email;
	private String password;
	private Collection<? extends GrantedAuthority> authorities;

	public UserDetailsImpl(Long idLong, String username, String email, String password,
			Collection<? extends GrantedAuthority> authorities) {
		this.idLong = idLong;
		this.username = username;
		this.email = email;
		this.password = password;
		this.authorities = authorities;
	}
	
	public static UserDetailsImpl build(User user) {
		List<GrantedAuthority> authorities = user.getRol().stream()
				.map(role -> new SimpleGrantedAuthority(role.getRol().name()))
				.collect(Collectors.toList());
		
		return new UserDetailsImpl(
				user.getId(), 
				user.getName(), 
				user.getEmail(), 
				user.getPassword(), 
				authorities);
	}

	@Override
	public Collection<? extends GrantedAuthority> getAuthorities() {
		return authorities;
	}

	@Override
	public String getPassword() {
		return password;
	}

	@Override
	public String getUsername() {
		return username;
	}
	
	public Long getID() {
		return idLong;
	}	
	
	public String getEmail() {
		return email;
	}

	@Override
	public boolean isAccountNonExpired() {
		return true;
	}

	@Override
	public boolean isAccountNonLocked() {
		return true;
	}

	@Override
	public boolean isCredentialsNonExpired() {
		return true;
	}

	@Override
	public boolean isEnabled() {
		return true;
	}

	@Override
	public int hashCode() {
		return Objects.hash(authorities, email, idLong, password, username);
	}

	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (obj == null)
			return false;
		if (getClass() != obj.getClass())
			return false;
		UserDetailsImpl other = (UserDetailsImpl) obj;
		return Objects.equals(authorities, other.authorities) && Objects.equals(email, other.email)
				&& Objects.equals(idLong, other.idLong) && Objects.equals(password, other.password)
				&& Objects.equals(username, other.username);
	}
	
}

```


3. Crear la clase "UserDetailsServiceImpl" que implementa "UserDetailsService" y en la que definiremos como se debe buscar a un usuario (En este caso creo loadUserByEmail, pq no quiero usar el ByUsername pq no busco por username, si no por email, podia simplemente buscar por email en ese metodo pero ya no estaria bien la nomenclatura, pero este debe estar ahí pq lo implementamos)

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

	UserRepository userRepository;
	
	public UserDetailsServiceImpl(UserRepository userRepository) {
		this.userRepository = userRepository;
	}
	
	@Override
	public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException 
		return null;
	}
	
	public UserDetailsImpl loadUserByEmail(String email) throws UsernameNotFoundException {
		User user = userRepository.findByEmail(email)
				.orElseThrow(() -> new UserNotFoundException(email));
		
		return UserDetailsImpl.build(user);
	}
	

}

```



4. Despues podemos crear la una clase "CustomAuthenticationManager.java" que implementara de "AuthenticationManager" y sera encargada de controlar la funcion de autenticacion:

```java
@Component
public class CustomAuthenticationManager implements AuthenticationManager {

	private UserDetailsServiceImpl customUserDetailsService;
	private PasswordEncoder passwordEncoder;
	
	public CustomAuthenticationManager (
			UserDetailsServiceImpl customUserDetailsService,
			PasswordEncoder passwordEncoder
			) {
		this.customUserDetailsService = customUserDetailsService;
		this.passwordEncoder = passwordEncoder;
	}
	
	
	@Override
	public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		final UserDetails userDetails = customUserDetailsService.loadUserByEmail(authentication.getName());
		if (!passwordEncoder.matches(authentication.getCredentials().toString(), userDetails.getPassword())) {
			throw new UserRegisterNotValidException();
		}
		return new UsernamePasswordAuthenticationToken(
			userDetails, 
			userDetails.getPassword(), 
			userDetails.getAuthorities());
	}
	
}

```


5. Crearemos JwtUtils que sera una clase con metodos para jwt
```java
@Component
public class JwtUtils {
	private static final Logger logger = LoggerFactory.getLogger(JwtUtils.class);
	
	@Value("${security.app.jwtSecret}")
	private String jwtSecret;
	
	@Value("${security.app.jwtExpirationMs}")
	private int jwtExpirationMs;
	
	public String generateJwtToken(Authentication authentication) {

				//Get details to set email in jwt
		UserDetailsImpl userDetailsImpl = (UserDetailsImpl) authentication.getPrincipal();

		return Jwts.builder()
				.setSubject((userDetailsImpl.getEmail()))
				.setIssuedAt(new Date())
				.setExpiration(new Date(new Date().getTime() + jwtExpirationMs))
				.signWith(SignatureAlgorithm.HS512, jwtSecret)
				.compact();
	}
	
	public String getEmailNameFromJwtToken(String token) {
		return Jwts.parser()
				.setSigningKey(jwtSecret)
				.parseClaimsJws(token)
				.getBody()
				.getSubject();
	}
	
	public boolean validateJwtToken(String authToken) {
		try {
			Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(authToken);
			return true;
		} catch (SignatureException  e) {
			logger.error("Invalid JWT signature: {}", e.getMessage());
		}catch (MalformedJwtException e) {
			logger.error("Invalid JWT token: {}", e.getMessage());
		} catch (ExpiredJwtException e) {
			logger.error("JWT token is expired: {}", e.getMessage());
		} catch (UnsupportedJwtException e) {
			logger.error("JWT token is unsupported: {}", e.getMessage());
		} catch (IllegalArgumentException e) {
			logger.error("JWT claims string is empty: {}", e.getMessage());
		}
		
		return false;
	}
	
}
```

6. Ya con todo lo creado, podemos pasar a crear por fin el userSerevice, el cual va a tener los metodos para logearse/registrarse

```java
@Service
public class UserService {
	
	private UserRepository userRepository;
	private RolRepository rolRepository;
	private PasswordEncoder passwordEncoder;
	private UserMapper userMapper;
	private Mapper mapper;
	private AuthenticationManager authenticationManager;
	private JwtUtils jwtUtils;
	
	public UserService(
			UserRepository userRepository, 
			PasswordEncoder passwordEncoder,  
			UserMapper userMapper,
			RolRepository rolRepository,
			Mapper mapper,
			AuthenticationManager authenticationManager,
			JwtUtils jwtUtils) {
		this.userRepository = userRepository;
		this.passwordEncoder = passwordEncoder;
		this.userMapper = userMapper;
		this.rolRepository = rolRepository;
		this.mapper = mapper;
		this.authenticationManager = authenticationManager;
		this.jwtUtils = jwtUtils;
	}
	
	
	
	public ResponseEntity<ApiResponse> login(LoginRequest userLogin){

		//Validate the existence of the user/password
		if (!userRepository.existsByEmail(userLogin.getEmail())) {
			throw new UserNotFoundException(userLogin.getEmail());
		}
		
		//Authenticate user
		Authentication authentication = authenticationManager.authenticate(
					new UsernamePasswordAuthenticationToken(userLogin.getEmail(), userLogin.getPassword())
				);
		
		//Set it in context
		SecurityContextHolder.getContext().setAuthentication(authentication);
		
		//Generate jwt
		String jwt = jwtUtils.generateJwtToken(authentication);
		
		//Get user details
		UserDetailsImpl userDetails = (UserDetailsImpl) authentication.getPrincipal();
		
	    List<String> roles = userDetails.getAuthorities().stream()
	            .map(item -> item.getAuthority())
	            .collect(Collectors.toList());
		
	    //Return data
		JwtResponse jwtResponse = new JwtResponse(
					jwt,
					userDetails.getID(),
					userDetails.getUsername(),
					userDetails.getEmail(),
					roles					
				);
		
		ApiResponse response = new ApiResponse(jwtResponse);
		
		return new ResponseEntity<ApiResponse>(
					response,
					HttpStatus.OK
				);

	}
	
	
	public ResponseEntity<ApiResponse> register(SignupRequest userRequest) throws Exception {
		//Check if exists user
		if (userRepository.existsByEmail(userRequest.getEmail())) {
			throw new UserAlreadyExistsException(userRequest.getEmail());
		}
		
		//Create user
		User user = new User(
				userRequest.getEmail(),
				userRequest.getName(),
				passwordEncoder.encode(userRequest.getPassword())
				);
		
		//Assign roles
		Set<String> strRole = userRequest.getRole();
		Set<Rol> roles = new HashSet<>();
		
		if (strRole == null) {
			Rol userRole = rolRepository.findByRol(ERol.ROLE_USER)
					.orElseThrow(() -> new RolNotFoundException(ERol.ROLE_USER));
			
			roles.add(userRole);

		}else {
			strRole.forEach(role -> {
				Rol userRole;
				switch (role) {
				case "ROLE_ADMIN": 
					userRole = rolRepository.findByRol(ERol.ROLE_ADMIN)
					.orElseThrow(() -> new RolNotFoundException(ERol.ROLE_ADMIN));
					
					roles.add(userRole);
					break;
					
				case "ROLE_MODERATOR": 
					userRole = rolRepository.findByRol(ERol.ROLE_MODERATOR)
					.orElseThrow(() -> new RolNotFoundException(ERol.ROLE_MODERATOR));
					
					roles.add(userRole);
					break;
					
				case "ROLE_USER": 
					userRole = rolRepository.findByRol(ERol.ROLE_USER)
					.orElseThrow(() -> new RolNotFoundException(ERol.ROLE_USER));
					
					roles.add(userRole);
					break;
					
				default:
					throw new RolNotFoundException(role);
				}
			});

		}
		
		user.setRol(roles);

		//Save and transform user
		UserDTO userDTO = userMapper.userDTO(userRepository.save(user));
		
		//Return ResponseEntity
		ApiResponse response = new ApiResponse(userDTO);
		return new ResponseEntity<ApiResponse>(
					response,
					HttpStatus.CREATED
				);
	}
	
	public User getUserEntity(String email){
		User user = userRepository.findByEmail(email)
				.orElseThrow(() -> new UserNotFoundException(email));

		return user;
	}
	
	public boolean existsUser(String email) {
		return userRepository.existsByEmail(email);
	}

}
```

7. Ahora si que podemos crear endpoints para estos servicios de auth en el AuthController
```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {
	
	private final UserService userService;
	
	public AuthController(UserService userService) {
		this.userService = userService;
	}
	
	@PostMapping("/login")
	public ResponseEntity<ApiResponse> login(
			@RequestBody LoginRequest user
			){
		return userService.login(user);
	}
	
	
	@PostMapping("/register")
	public ResponseEntity<ApiResponse> register(
			@RequestBody SignupRequest singUpRequest
			) throws Exception {
		return userService.register(singUpRequest);
	}
	
}
```


En este punto, nuestra aplicacion es capaz de registrar y logear (otorgando un jwt) a un usuario, ahora vamos a pasar a la parte de filtrar todas las peticones para ver si cuentan con un jwt correcto

8. Definiremos la clase AuthTokenFilter que extiende de OncePerRequestFilter, y modificaremos el metodo doFilterInternal que ejecutara el codigo que tiene cada vez que se haga una peticion http (a no ser que expresemos lo contrario en los cors)

```java

public class AuthTokenFilter extends OncePerRequestFilter {

	@Autowired
	private JwtUtils jwtUtils;
	
    @Autowired
    private UserDetailsServiceImpl userDetailsService;
	
	@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {
		try {
			String jwt = parseJwt(request);
			if (jwt != null && jwtUtils.validateJwtToken(jwt)) {
				
				//Get mail from jwt
				String email = jwtUtils.getEmailNameFromJwtToken(jwt);
				
				//Search user by mail
				UserDetails userDetails = userDetailsService.loadUserByEmail(email);
				
				UsernamePasswordAuthenticationToken authentication = 
						new UsernamePasswordAuthenticationToken(
								userDetails,
								null,
								userDetails.getAuthorities());
				
				// ??
				authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
				
				//Save user in securityContext
				SecurityContextHolder.getContext().setAuthentication(authentication);
			}
		} catch (Exception e) {
			logger.error("Cannot set user authentication: {}", e);
		}
		
		filterChain.doFilter(request, response);
	}
	
	private String parseJwt(HttpServletRequest request) {
		// Get header for auth
		String headerAuth = request.getHeader("Authorization");
		
		// Check if it have text and is of type Bearer
		if (StringUtils.hasText(headerAuth) && headerAuth.startsWith("Bearer ")) {
			return headerAuth.substring(7, headerAuth.length()); // substring of 7 becouse it is "|B|e|a|r|e|r| | ......."
		}
		
		return null;
	}

}
```


10. Añadiremos cors (en la clase WebSecurityConfig), para que las peticiones sean filtradas y especificar que rutas necesitan autorizacion, etc.

```java 
	  @Bean
	  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
		//Disable spring-security csrf, because spring boot already has its own csrf, we
		//also allow all the get requests, but the rest we will apply basic authentication
	    http.cors().and().csrf().disable()
	        .exceptionHandling().authenticationEntryPoint(unauthorizedHandler) // Allows configuring exeption handling
	        .and()
	        .sessionManagement() // If a user without logout logs in again, the first session will be closed
	        .sessionCreationPolicy(SessionCreationPolicy.STATELESS) // Spring Security will never create an HttpSession and it will never use it to obtain the SecurityContext because we will control the session with JWT
	        .and()
	        // permit this routes without authentication
	        .authorizeRequests().antMatchers("/api/auth/**").permitAll()
	        .antMatchers("/api/test/all").permitAll() 
	        .anyRequest().authenticated(); // any request must be authenticated
	    
	    http.authenticationProvider(authenticationProvider());

	    http.addFilterBefore(authenticationJwtTokenFilter(), UsernamePasswordAuthenticationFilter.class);
	    
	    return http.build();
	  }
```

11. Finalmente ya podemos crear nuestro TestController para hacer pruebas de autenticacion y permisos

```java
@RestController
@RequestMapping("/api/test")
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class TestController {
	
	@GetMapping("/all")
	public ResponseEntity<ApiResponse> allAccess(){
		ApiResponse response = new ApiResponse("Public content");
		return new ResponseEntity<ApiResponse>(
					response,
					HttpStatus.OK
				);
	}
	
	@GetMapping("/user")
	@PreAuthorize("hasRole('USER') or hasRole('MODERATOR') or hasRole('ADMIN')")
	public ResponseEntity<ApiResponse> userAccess(){
		ApiResponse response = new ApiResponse("User content");
		return new ResponseEntity<ApiResponse>(
					response,
					HttpStatus.OK
				);
	}
	
	@GetMapping("/mod")
	@PreAuthorize("hasRole('MODERATOR') or hasRole('ADMIN')")
	public ResponseEntity<ApiResponse> modAccess(){
		ApiResponse response = new ApiResponse("Moderator content");
		return new ResponseEntity<ApiResponse>(
					response,
					HttpStatus.OK
				);
	}
	
	@GetMapping("/admin")
	@PreAuthorize("hasRole('ADMIN')")
	public ResponseEntity<ApiResponse> adminAccess(){
		ApiResponse response = new ApiResponse("Admin content");
		return new ResponseEntity<ApiResponse>(
					response,
					HttpStatus.OK
				);
	}
```


## ORDEN Y DEFINICIONES DE CLASES

### ORDEN
1 WebSecurityConfig
2 CustomUserDetailsService
3 UserDetailsServiceImpl
4 CustomAuthenticationManager
5 JwtUtils
6 UserService
7 AuthController

*En este punto, nuestra aplicacion es capaz de registrar y logear (otorgando un jwt) a un usuario, ahora vamos a pasar a la parte de filtrar todas las peticones para ver si cuentan con un jwt correcto*

8 AuthTokenFilter
9 AuthEntryPointJwt
10 Cors (WebSecurityConfig)
11 TestController

### DEFINICION DE CLASES

Clases:

- WebSecurityConfig -> configuracion y declaracion de beans

- UserDetailsImpl -> Para añadir mas datos que podamos obtener del usuario como id, email, etc.

- UserDetailsServiceImpl -> como buscaremos a nuestros usuarios en la bd a la hora del loggeo

- CustomAuthenticationManager -> contiene el metodo authenticate que es el que se va a usar a la hora de autenticar un usuario

- JwtUtils -> Provee metodos para generar, parsear y validar JWT's

- userService -> contiene los metodos de registro/logeo

- AuthController -> controlador para registrarse/logearse :)

- AuthTokenFilter -> encargado de autenticar al usuario por el jwt

- AuthEntryPointJwt -> handler de unauthorized users

- TestController -> controlador para hacer prueba de authentication
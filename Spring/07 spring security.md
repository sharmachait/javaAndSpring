![[Pasted image 20241012203444.png]]

![[Pasted image 20240926120708.png]]
by simply adding this dependency spring security protects all the pages and redirects to /login for everything with a default login page
it gives us a default username and password that can be used to access all the pages

***==username - user==***
***==password - printed on the console when we start the application==***

to configure custom starter credentials
in the application.properties add
```
spring.security.user.name = eazybytes
spring.security.user.password = 12345
```
when we do this any new password will not be logged

we also have a default logout page at /logout

the default behaviour of spring security to authenticate all urls is configured in the SpringBootWebSecurityConfiguration class's defaultSecurityFilterChain(HttpSecurity http) method
```java
@Bean
@Order(SecurityProperties.BASIC_AUTH_ORDER)
SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
	http.authorizeHttpRequests().anyRequest().authenticated();
	http.formLogin();
	http.httpBasic();
	return http.build();
}
```


## customizing our SecurityChainFilter
1. to make all urls public
```java
@Configuration
public class ZapierSecurityConfig {
	@Bean
	SecurityChainFilter defaultSecurityFilterChain(HttpSecurity http) throws Exception {
		
		http.auhtorizeHttpRequests()
			.anyRequest().permitAll()
			.and().formLogin()
			.and().httpBasic();
		
		return http.build();
	}
}
```
2. to deny access regardless of authentication use .denyAll() instead of .permitAll() 
#### we can use permitAll and denyAll on specific url paths as well
3. permitting and denying certain routes, instead of anyRequest use requestMatchers
4. but allowing only api paths will deny access to html css and javascript files, to allow access to them use /assets/**
```java
@Configuration
public class ZapierSecurityConfig {
	@Bean
	SecurityChainFilter defaultSecurityFilterChain(HttpSecurity http) throws Exception {
		
		http.auhtorizeHttpRequests()
			.requestMatchers("", "/", "/home").permitAll()
			.requestMatchers("/holidays").permitAll()
			.requestMatchers("/contact").permitAll()
			.requestMatchers("/courses").permitAll()
			.requestMatchers("/assets/**").permitAll()
			.and().formLogin()
			.and().httpBasic();
		
		return http.build();
	}
}
```
this wont match the route /holidays/all, to match that use /holidays/**
request matchers takes multiple paths as well

## spring security by default blocks all puts, post and deletes owing to CSRF protection

we can disable or handle csrf
internally thymeleaf handles csrf for us
but in case of react we ned to handle csrf

to disbale csrf use .csrf().disable() on the httpsecurity object and inject into the security filter chain
```java
@Configuration
public class ZapierSecurityConfig {
	@Bean
	SecurityChainFilter defaultSecurityFilterChain(HttpSecurity http) throws Exception {
		
		http
			.csrf().disable()
			.auhtorizeHttpRequests()
			.requestMatchers("", "/", "/home").permitAll()
			.requestMatchers("/holidays").permitAll()
			.requestMatchers("/contact").permitAll()
			.requestMatchers("/courses").permitAll()
			.requestMatchers("/assets/**").permitAll()
			.and().formLogin()
			.and().httpBasic();
		
		return http.build();
	}
}
```
not good practice we should handle the csrf token instead
if we tend to have some In Memory Data About Users we can inject a dependency of InMemoryUserDetailsManager

```java
@Configuration
public class ZapierSecurityConfig {
	@Bean
	SecurityChainFilter defaultSecurityFilterChain(HttpSecurity http) throws Exception {
		
		http
			.csrf().disable()
			.auhtorizeHttpRequests()
			.requestMatchers("", "/", "/home").authenticated()
			.requestMatchers("/holidays").permitAll()
			.requestMatchers("/contact").permitAll()
			.requestMatchers("/courses").permitAll()
			.requestMatchers("/assets/**").permitAll()
			.and().formLogin()
			.and().httpBasic();
		
		return http.build();
	}
	@Bean
	public InMemoryUserDetailsManager userDetailsService(){
		UserDetails user = User.withDefaultPasswordEndocer()
								.username("user")
								.password("12345")
								.roles("USER")
								.build();
		UserDetails admin = User.withDefaultPasswordEndocer()
								.username("admin")
								.password("12345")
								.roles("USER","ADMIN")
								.build();
		
		return new InMemoryUserDetailsManager(user, admin);
	}
}
``` 

all these classes User UserDetails InMemoryUserDetailsManager are provided by the spring security framework
this is called a builder pattern
use InMemoryUserDetailsManager only for POC or internal applications

# custom login workflow and page
give the url where the login page should be and implement a controller for it
```java
@Configuration
public class ZapierSecurityConfig {
	@Bean
	SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
		
		http
			.csrf().disable()
			.auhtorizeHttpRequests()
			.requestMatchers("/login/**").permitAll()
			.requestMatchers("/dashboard").authenticated()
			.requestMatchers("", "/", "/home").permitAll()
			.requestMatchers("/contact").permitAll()
			.requestMatchers("/assets/**").permitAll()
			.and().formLogin().loginPage("/login")// to redirect to, to authenticate
			.defaultSuccessUrl("/dashboard")// to redirect to after login
			.failureUrl("/login?error=true").permitAll()// to redirect to incase of failure
			.and().logout().logoutSuccessUrl("/login?logout=true")
					.invalidateHttpSession(true).permitAll()
			.and().httpBasic();
		
		return http.build();
	}
}
```
the failureUrl needs to be permitted for all
the login page needs to be permitted all as well

but we need to write controller for all these as well
in the /dashboard controller we can access the roles and details of the logged in user by receiving Authentication as parameter
```java
@Slf4j
@Controller
public class DashboardController{
	@RequestMapping("/dashboard")
	public String displayDashboard(Model model, Authentication authentication){
		model.addAttribute("username",authentication.getName()):
		model.addAttribute("roles",authentication.getAuthorities().toString()):
	}
}
```
in the login controller the `?error=true` and `?logout=true` can be handled with @RequestParam
```java
@Slf4j
@Controller
public class AuthController{
	@RequestMapping("/login", method = {RequestMethod.GET})
	public String displayDashboard(){
		return "login.html"
	}
	@RequestMapping("/login", method = {RequestMethod.POST})
	public String displayDashboard(
		@RequestParam(value="error", required=false) String error,
		@RequestParam(value="logout", required=false) String logout,
		Model model
	){
		String errorMessage = null;
		if(error != null && error){
			errorMessage = "Incorrect credentials !";
		}
		if(logout != null && logout){
			errorMessage = "you have been logged out !!"
		}
		model.addAttribute("errorMessage", errorMessage);
		return "login.html";
	}
}
```

## RBAC

```java
@Configuration
public class ZapierSecurityConfig {
	@Bean
	SecurityChainFilter defaultSecurityFilterChain(HttpSecurity http) throws Exception {
		
		http
			.csrf().ignoringRequestMatchers("/saveMsg")
			.auhtorizeHttpRequests()
			.requestMatchers("/closeMsg/**").hasRole("ADMIN")
			.requestMatchers("/displayMessages").hasRole("ADMIN");
		return http.build();
	}
}
```
we used two ** becuase we are also expecting a request param after that

to pass the data of the person invoking a method controller we can pass Authentication as a param to that method

```java
@RequestMapping(value = "/closeMsg", method = GET)
public String closeMsg(@RequestParam int id, Authentication auth){
	contactService.updateMsgStatus(id,atuh.getName());
	return "redirect:/displayMessages";
}
```

# Database integration
We can write custom logic to authentication user based on our requirements by implementing the AuthenticationProvider interface

```java
@Component
public class UsernameAndPasswordAuthProvider implements AuthenticationProvider {
	@Autowired
	private UserRepository userRepository;
	
	@Override
	public Authentication authenticate(Authentication auth) throws AuthenticationException{
		String name = auth.getName();
		String password = auth.getCredentials.toString();
		User user = userRepository.findByUsername(username);
		if(ValidCredentials(name, password, user)){
			return new UsernamePasswordAuthenticationToken(
				name, passowrd, getGrantedAuthorities(user.getRoles())
			);
		} else {
			return null;
		}
	}
	
	@Override
	public boolean supports(Class<?> auth){
		return auth.equals(UsernamePasswordAuthenticationToken.class);
	}
	
	public boolean ValidCredentials(String username, String password, User user){
		
		if(user==null)
			return false;
		if(user.getUsername().equals(username))
			return false;
		boolean passwordsEqual = comparePasswords(password,user.getPasswordHash());
		if(!passwordsEqual)
			return false;
		return true;
	}
	
	public boolean comparePasswords(String password, String hash){
		
	}
	
	public List<GrantedAuthorities> getGrantedAuthorities(Roles roles){
		List<GrantedAuthorities> l = new ArrayList<>();
		for(String role : roles.getRoles()){
			l.add(new SimpleGrantedAuthority("ROLE_"+role));
		}
		return granted Authorities
	}
}
```

# password encoders
### BcryptPasswordEncoder - most commonly used
the PasswordEncoder interface has two methods 
- encode(CharSequence rawPassword) and 
- matcher(CharSequence rawPassword, String encodedPassword)
stored password is never decoded

step 1. Inject a bean of BcryptPasswordEncoder
step 2. Use the bean to encode the password wherever we are saving it in the DB like the userService register method
	encoder.encode(user.getPassword())
step 3. use the bean in the auth provider to login the user  


# all the annotation validations are done by spring framework and the data jpa as well
## we can toggle the validations off when saving to database by
```
spring.jpa.properties.javax.persistence.validation.mode=none
```
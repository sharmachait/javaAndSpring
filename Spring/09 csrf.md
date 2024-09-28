the csrf token is not stored in the cookies, it must be used as hidden field of each and every form
instead of disabling the csrf for all the pages and urls we shoudl instead tell spring security withc urls it should ignore the csrf token on
instead of 
```java
http.csrf().disable()
```
do the following
```java
http.csrf().ignoringRequestMatchers("/saveMsg")
```
this will disable csrf on the urls matching the pattern
but this will invalidate the default logout that spring security provides owing to the csrf token, becuase there is no form submitting the csrf token in the default scenario
so permit all the logout  route as well
```java
@Configuration
public class ZapierSecurityConfig {
	@Bean
	SecurityChainFilter defaultSecurityFilterChain(HttpSecurity http) throws Exception {
		
		http
			.csrf().ignoringRequestMatchers("/saveMsg")
			.auhtorizeHttpRequests()
			.requestMatchers("/login/**").permitAll()
			.requestMatchers("/logout").permitAll()
			.requestMatchers("/dashboard").authenticated()
			.requestMatchers("", "/", "/home").permitAll()
			.requestMatchers("/contact").permitAll()
			.requestMatchers("/assets/**").permitAll()
			.and().formLogin().loginPage("/login")// to redirect to, to authenticate
			.defaultSuccessUrl("/dashboard")// to redirect to after login
			.failureUrl("/login?error=true").permitAll()// to redirect to incase of failure
			//.and().logout().logoutSuccessUrl("/login?logout=true")
					//.invalidateHttpSession(true).permitAll()
			.and().httpBasic();
		
		return http.build();
	}
}
```

to handle logout we need to create another controller that server the logout request 
```java
@RequestMapping(value="/logout", method = RequestMethod.GET)
public String logoutPage(HttpServletRequest req, HttpServletResponse res){
	Authentication auth = SecurityContextHolder.getContext().getAuthentication();
	if(auth!=null){
		new SecurityContextLogoutHandler().logout(req,res,auth);
	}
	return "redirect:/login?logout=true";
}
```
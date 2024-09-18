# Servlets
Servlet Containers are web server instances, that understand the HTTP protocol and parse it
Apache Tomcat

It translates the HTTP message into ServletRequest and hands it to a controller method, the ServletResponse  returned by the controller is translated into an HTTP message by the servlet container

Spring uses a Dispatcher servlet for all the url mappings and translations

 Spring boot came with a built in embeded server
# Starter projects 
that include configuration for stuff like kafka and other technologies

Based on the dependencies of the starter projects spring boot can auto configure beans that will be required to use the technologies, and startup configurations will be setup
but also allows us to over ride the default configurations and beans

if we have MYSQL in the starter project it will automatically create the db context by looking for the database connection strings in the application.properties

# Actuators
Built in monitoring and management end points
returns the memory and cpu usage and other useful metrics

# hot reload  devtools
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
	<scope>runtime</scope>
	<optional>true</optional>
</dependency>
```
we also need to install livereload extension in the browser
this also takes care of thymeleaf cache
# adhock
entry point  =>  class with @SpringBootApplication annotation

this annotation is a combination of other annotations like springbootconfiguration, enableautconfiguration, componentscan etc, all providing a default configuration for a spring boot application

changing the default cofiguration
![[Pasted image 20240916125528.png]]
for random port number set server.port = 0

to get a report of everything that was autoconfigured for us use 
debug=true

we can route multiple routes to the same controller method like so
![[Pasted image 20240916234013.png]]
# thymeleaf
Thymeleaf templates are created in the templates folder not the static folder
to make html into a thymeleaf template
![[Pasted image 20240917083407.png]]

we can simply accept the view model in the controller method
![[Pasted image 20240917091201.png]]
the model is used on the template like so
![[Pasted image 20240917091249.png]]

in development mode we want to disable the template cache via application.properties

```properties
spring.thymelead.cache=false
```

create a folder in static called assets and create three more inside assets, css, js and images serve them from there
in the templates if me mention the href location of any file spring boot by default looks in the static folder

![[Pasted image 20240917125019.png]]handler mapper sees what route should call which controller
tomcat converts http request into a servlet request
view resolver makes the html with populated values
![[Pasted image 20240917130052.png]]

to use a partial html in the main thml file
```html
<div th:replace="header :: header"></div>
```
this will tell thymeleaf to replace this div with the html written in the header.html file
the header.html file will have all the required tags like html head body link js and link css etc
thymeleaf anchor tags
```html
<a th:href="@{/route/to/the/place/we/want}">link text</a>
```
but we should have a controller handling that path

but incase we have a static page with no values being populated nothing dynamic, in that case we dont need a controller we can use a shortcut
![[Pasted image 20240917231001.png]]just like controller package create a package for config and put this class there
with this configuration thymeleaf will expect html files with courses and about as name in the templates folder

submitting forms require a thymeleaf annotation th:action and the method the form is using

```html
<form th:action="@{/savemsg}" method="post"></form>
```

# accepting parameters in the controller

### to accept this forms data as input in the controller, bad way
```java
@RequestMapping(value="/savemsg", method = POST)
public ModelAndView saveMessage(@RequestParam String name, @RequestParam String mobileNum, @RequestParam String email){
	// all these params must match the names in the form html
	sout(name);
	sout(mobileNum);
	sout(email);
	return new ModelAndView("redirect:/contact");
}
```

instead of `@RequestMapping(value="/savemsg", method = POST)`
we can also do `@PostMapping(value="/savemsg")`
### better way
using a Pojo class
1. create another package called Model
2. create the view model that has all the properties, with the same name as the names in the html form
```java
public class ContactViewModel{
	private Strign name;
	private Strign email;

	public getName(){return this.name;}
	public setName(String name){this.name = name;}

	public getEmail(){return this.email;}
	public setEmail(String email){this.email = email;}
}
```

with this in place our controller method can expect an object of this ViewModel as parameter
```java
@RequestMapping("/savemsg",method = POST)
public ModelAndView saveMessage(ContactViewModel model){
	sout(model.getName());
	sout(model.getEmail());
	// do validations
	return new ModelAndView("redirect:/contact")
}
```
# use logger

```java
public class ContactController {
	private static Logger log = LoggerFactory.getLogger(ContactController.class);

}
```

or just use the lombok annotation `@Slf4j`

```java
@Slf4j
@Controller
public class ContactController {
	//private static Logger log = LoggerFactory.getLogger(ContactController.class);
	@RequestMapping("/savemsg", Method = POST)
	public String Contact(Model model){
		log.info(model.name);
		return "redirect:/Contact";
	}

}
```

# Service layer
1. create a package called Service
2. all classes in this should have the @Service annotation
```java
@Service
public class ContactService{
	private static Logger log = LoggerFactory.getLogger(ContactService.class);
	public boolean processMessage(ContactViewModel contact){
		log.info(contact.getName());
		log.info(contact.getEmail());
		return true;
	}
}
```
3. autowire this in the constructor of the controller
```java
public class ContactController {
	private final ContactService contactService;
	
	@Autowired
	public ContactController(ContactSerivce cs){
		this.contactService = cs;
	}

	@RequestMapping("/savemsg",method = POST)
	public ModelAndView saveMessage(ContactViewModel model){
		sout(model.getName());
		sout(model.getEmail());
		// do validations
		return new ModelAndView("redirect:/contact")
	}
}
```

# sending data from backend to the frontend

1. have a viewModel whose object will be sent to the frontend
```java
public class Holiday{
	private final String name;// with getters and setters
	public enum Type{
		FESTIVAL, FEDERAL
	}
	private final Type type;// with getters and setters

	public Holiday(String name, Type type){
		this.type = type;
		this.name = name;
	}
}
```

```java
@Controller
public class HolidayController{
	@GetMapping("/holidays)
	public String DisplayHolidays(Model model){
		List<Holiday> holidays = _repo.getHolidays();
		for(Holiday.Type type : Holiday.Type.values()){
			
			List<Holidays> holidaysOfType = 
				holidays.stream().filter(h - > h.getType().equals(type)).collect(Collectors.toList());
			
			model.addAttribute(type.toString(), holidaysOfType)
		}
		return "holidays.html";
	}
}
```

using this data in thymeleaf
```html
<div th:each ="holiday : $ {FESTIVAL}">
	<div class="title">
		<h2 th:text="${holiday.name}"></h2>
	</div>
</div>
<div th:each ="holiday : $ {FEDERAL}">
	<div class="title">
		<h2 th:text="${holiday.name}"></h2>
	</div>
</div>
```

# lombok

automatically creates getters and setters for the models and also constructors
![[Pasted image 20240918095419.png]]
most commonly used lombok annotations
1. @Getter
2. @Setter
3. @NoArgsConstructor
4. @RequiredArgsConstructor -- creates constructor for all the final fields 
5. @AllArgsConstructor
6. @ToString
7. @EqualsAndHashCode
8. @Data -- shortcut that combines the features of 1, 2, 4, 6, 7  (Getter, Setter, ToString, RequiredArgsConstructor, EqualsAndHashCode)
9. @Slf4j -- to ge tthe default logger for that class from the LoggerFactory, simply annotate the class and use the log object

The lombok magic is done during compile time
![[Pasted image 20240918100201.png]]

@Slf4j example
```java
@Slf4j
@Controller
public class ContactController {
	//private static Logger log = LoggerFactory.getLogger(ContactController.class);
	@RequestMapping("/savemsg", Method = POST)
	public String Contact(Model model){
		log.info(model.name);
		return "redirect:/Contact";
	}

}
```

# 1 Singleton default
# 2 Prototype == Transient
# 3 Request - related to web applications
# 4 Session - related to web applications
# 5 Application - related to web applications

## to define the scope of a bean

```java
@Component
@Scope(BeanDefinition.SCOPE_SINGLETON)
public class VehicleService{

}
```
We should not use singleton stateful objects in multithreaded scenario where we want to change the  state
## lazy initialization
we can make the bean initialization lazy with @Lazy
```java
@Component
@Scope(BeanDefinition.SCOPE_SINGLETON)
@Lazy
public class VehicleService{

}
```

lazy initialization can cause runtime exceptions
## Prototype scope
```java
@Component
@Scope(BeanDefinition.SCOPE_PROTOTYPE)
public class VehicleService{

}
```
for prototype scope the default is lazy initialization
safe to use in multithreaded scenarios
Injected at startup time only if some other singleton bean depends on it, and the instance provided to the singleton object by the IOC container wont be destroyed as long as the singleton object is alive
Inside the singleton class the prototype bean will effectively be singleton

# immutable state classes should be made singleton
# Mutable State classes should be made prototype

# Bean Web Scopes
1. Session
	1. @SessionScope
	2. spring keeps the instance in the servers memory for the full http session
	3. the object is linked with the client session
	4. or as long as the browser is open  
2. Request
	1. @RequestScope
	2. new instance for each and every http request
3. Application
	1. @ApplicationScope
	2. same object as long as the app is running

Where to use these scopes?
for services and repositories
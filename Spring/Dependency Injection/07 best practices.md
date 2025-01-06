1. dont autowire private properties ebcuase we want to be able to use these dependencies in unit tests
2. use public setters to autowire these private properties

```java
private CustomerService cs;
@Autowire
public void setCS(CustomerService cs){
	this.cs=cs;
}
```

downsides of this approach is that the CustomerService dependency can be updated at runtime 
to solve that autowire via constructor

```java
@RestController
class CustomerController {
	private final CustomerService cs;
	
	public CustomerController(CustomerService cs){
		this.cs=cs;
	}
}
```

we dont even need to write autowired over the constructor
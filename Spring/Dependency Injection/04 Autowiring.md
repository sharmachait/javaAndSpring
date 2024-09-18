# bad way to inject
manual

```java
@Bean
public Vehicle vehicle(){
	return new Vehicle();
}

@Bean
public Person person(){
	Person p = new Person();
	p.setVehcile(Vehicle());
	return p;
}
```

this may seem like 2 beans of vehicles are created but Spring makes sure only one is as speing understands the dependency graph

# bad way method parameters
```java
@Bean
public Vehicle vehicle(){
	return new Vehicle();
}

@Bean
public Person person(Vehicle vehicle){
	Person p = new Person();
	p.setVehcile(vehicle);
	return p;
}
```

# Better way not best @Autowired fields and @Autowired setter methods
why not best? because this way doesnt allow us to make our dependencies final, anyone can change them
```java
public class Person{
	@Autowired
	private Vehicle veh;
}
```

this will throw an error if there is no bean of type Vehicle
```java
public class Person{
	@Autowired(required=false)
	private Vehicle veh;
}
```

but with this approach we will have to check and handle nulls manually
```java
public class Person{
	private Vehicle veh;
	@Autowired
	public void setVehcile(Vehicle vehicle){
		this.veh = vehicle;
	}
}
```

# Best way to inject
only possible way to make dependency fields final, constructor autowiring
```java
@Component
public class Person{
	private final Vehicle veh;
	@Autowired
	public Person(Vehicle vehicle){
		this.veh = vehicle;
	}
}
```

when there is only one constructor we dont even need to @Autowired
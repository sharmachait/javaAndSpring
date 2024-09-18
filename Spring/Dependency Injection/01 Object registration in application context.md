#### AppConfig.java
```java
package config;

import beans.Vehicle;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {
    @Bean
    public Vehicle vehicle1() {
        var veh = new Vehicle();
        veh.setName("hi");
        return veh;
    }
}
```
#### Main.java
```java
import beans.Vehicle;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
7
public class Main {
    public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext(AppConfig.class);
        
        Vehicle veh = context.getBean(Vehicle.class);
    }
}
```
if in the app config i do multiple beans of same type we are able to register the beans but get NoUniqueBeanDefinitionException when trying to access beans just on the basis of type
#### AppConfig.java

```java
package config;

import beans.Vehicle;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {
    @Bean
    public Vehicle ferrari() {
        var veh = new Vehicle();
        veh.setName("hi");
        return veh;
    }
    @Bean
    public Vehicle audi() {
        var veh = new Vehicle();
        veh.setName("hi");
        return veh;
    }
}
```

in the context the name of these beans are ferrari and audi 
but when we try to get the with context.getBean(Vehicle.class); it will throw the runtime exception

to by pass that do
```java
Vehicle veh = context.getBean("ferrari", Vehicle.class);
```

to give custom names to beans
```java
package config;

import beans.Vehicle;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

	@Primary
    @Bean
    public Vehicle ferrari() {// the primary makes this the default
        var veh = new Vehicle();
        veh.setName("hi");
        return veh;
    }
    @Bean("audi")
    public Vehicle audi() {
        var veh = new Vehicle();
        veh.setName("hi");
        return veh;
    }
	@Bean(name = "ferrari")
    public Vehicle ferrari() {
        var veh = new Vehicle();
        veh.setName("hi");
        return veh;
    }
    @Bean(value = "koinsegg")
    public Vehicle koinsegg() {
        var veh = new Vehicle();
        veh.setName("hi");
        return veh;
    }
}
```

![[Pasted image 20240909090238.png]]
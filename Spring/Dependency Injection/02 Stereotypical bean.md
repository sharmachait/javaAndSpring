# Component

1. create a POJO
2. annotate the class with `@Component`
3. Annotate the appConfig with `@ComponentScan(<Package name>)`

a bean is created automatically
![[Pasted image 20240909085953.png]]![[Pasted image 20240909090131.png]]
![[Pasted image 20240909090224.png]]

# bypassing the disadvantage of @component where we can not tweak the object before bean creation
## use @PostConstruct
we can have multiple @PostConstruct methods in our POJO and all will be called but no order is gauranteed

they can be private as well

![[Pasted image 20240909090518.png]]
![[Pasted image 20240909090532.png]]
![[Pasted image 20240909090544.png]]

similarly to do something before destruction of the bean use @PreDestroy
![[Pasted image 20240909090633.png]]

predestroy methods are called when context.close() is called

```java
try(var context = new AnnotationConfigApplicationContext(AppConfig.class)){

}catch(Exceptio e){
	System.out.println(e.message);
}finally{
	context.close();
}
```
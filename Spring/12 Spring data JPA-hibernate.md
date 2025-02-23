# hibernate required a no args constructor to work properly with entities

JDBC still requires alot of boiler plate like with row mapper and  and passing all the parameters into the query and making prepared statements

Spring data JPA is just specification (interface) hibernate is the implementation
![[Pasted image 20241026110606.png]]
## important interfaces
### Repository - 
its an empty interface
it takes two generic parameters 1. the model and 2. the primary key column type
### CrudRepository
has basic methods for CRUD operations, and method to save, the save method is supposed to insert if new primary key and update if primary key already exists
### ListCrudRepository
Has List instead of an iterable wherever possible like in findall findallbyid and saveall
### PagingAndSortingRepository or ListPagingAndSortingRepository
provides methods to perform pagination and sorting when retrieving
has only two methods `Iterable<T> findAll(Sort sort);` and `Page<T> findAll(Pageable pageable);`
### JPARepository
Interface implements all these interfaces
just like MongoRepository
# Spring data JPA 
this project implements all these interfaces using hibernate

1. to be able to use the repository of a model we need to define that model as an @Entity, and annotations like @Table and @Column,
##### JPA will try to match by removing the underscore as well
use table if pojo and table name are not same same for colmun
```java

@Entity
@Table(name="contact_msg")
public class Contact extends BaseEntity{
	@Id
	@GeneratedValue(strategy = GenerationType.AUTO, generator="native")
	@GenericGenerator(name="native", strategy="native")
	@Column(name = "contact_id")
	private int contactId;
}
```
2. how to make base entity columns part of model as well, use the annotation @MappedSuperClass
```java
@Data
@MappedSuperclass
public class BaseEntity{
	private LocalDataTime createdAt;
	private LocalDataTime updatedAt;
	private String createdBy;
	private String updatedBy;
}
```
2. Create an interface for the Model by extending one of the Repository interfaces
```java
@Repository
public interface CotactRepository extends CrudRepository<Contact,Integer>{

}
```

3. enable JPA by telling the springboot application where to find the interface and where to find the models
```java
@SpringBootApplication
@EnableJpaRepositories("com.sharmachait.wazir.repository")
@EntityScan("com.sharmachait.wazir.models")
public class Wazir {
 
}
```

4. to convert enum to String
```java
@Data
@Entity
@Table(name="holidays")
public class Holiday extends BaseEntity{
	@Id
	private String day;
	private String reason;
	@Enumerated(EnumType.STRING)
	private Type type;
	
	public enum Type{
		FESTIVAL, FEDERAL
	}
}
```

after this we can inject the ContactRepository any where we want

```java
@Service
public class ContactService {
	@Autowired
	private ContactRepository contactrepository;
	public boolean saveMessageDetails(Contact contact){
		boolean isSave = false;
		Contact savedContact = contactRepository.save(contact);
		if(saveContact != null && savedContact.getContactId() > 0){
			isSaved = true;
		}
		return isSaved;
	}
	
}
```

5. to create a repository with all the default methods
```java
@Repository
public interface HolidaysRepository extends CrudRepository<Holiday, String>{

}
```

but all the methods like findAll return an iterable instead of a list
to convert an iterable into a list
```java
Iterable<Holiday> i = holidayRepository.findAll();
List<Holidays> l = StreamSupport.stream(i.spliterator(), false).collect(Collectors.toList());
```

6. updating an entry in the data base requires us to fetch it first make changes and save again
the find methods return an Optional instead of the object
```java
public boolean updateMsg(int contactId, String updatedBy){
	boolean isUpdated  =false;
	Optional<Contact> contact = contactRepository.findById(contactId);
	contact.ifPresent(
		c -> {
			c.setStatus(ContactStatus.close);
			c.setUpdatedBy(updatedBy);
			c.updatedAt(LocalDateTime.now());
		}
	);
	Contact updatedContact = contactRepository.save(contact.get());
	if(updatedContact!=null && updatedContact.getUpdatedBy()!=null){
		isUpdated=true;
	}
	return isUpdated;
}
```

7.  how to fetch data with custom logic? based on some random fields that are not ids, we need to use ==**Derived query methods**==
we just need to define query methods in our interface, and JPA will automatically create implementations that fetches data from the database based on those parameters
```java
List<Person> findByLastName(String lastName);
Person findByEmail(String email);
Person findByEmailAndLastName(String email, String lastName);
```

```java
@Repository
public interface CotactRepository extends CrudRepository<Contact,Integer>{
	List<Contact> findByStatus(String status);
	List<Contact> findByStatusAndDate(String status, String Date);
}
```

8. to see the sql that was generated by the framework in the console your application.properties should look like this
```
spring.datasource.url=""
spring.datasource.username=admin
apring.datasource.password=somepassword
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```


## derived query methods 
the method names are made up of introducer and criteria and the two sections are divided by "By"
```java
@Repository
public interface CotactRepository extends CrudRepository<Contact,Integer>{
	List<Contact> findByStatus(String status);
	List<Contact> findByStatusAndDate(String status, String Date);
}
```
### introducers
1. find
2. read
3. query
4. count
5. get
6. distinct - `findDistinctByStatus`
we can use distinct to tell JPA to get distinct items in the output like `findDistinctByStatus`
### criteria
any of the entity properties seperated by `And` and `Or`

readBy getBy and findBy are equivalent

![[Pasted image 20241002143707.png]]
![[Pasted image 20241002144014.png]]
![[Pasted image 20241002144050.png]]

## built in auditing
for columns like createdAt, createdBy, updatedAt and updatedBy with the help of annotations

to switch on auditing
1. annotate the base entity with the required fields
```java
@Data
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class BaseEntity {
	
	@CreatedDate
	@Column(updatable = false)
	private LocalDateTime createdAt;
	
	@CreatedBy
	@Column(updatable = false)
	private String createdBy;
	
	@LastModifiedDate
	@Column(insertable = false)
	private LocalDateTime updatedAt;
	
	@LastModifiedBy
	@Column(insertable = false)
	private String updatedBy;
}
```

but how will the framework know what is the current time and who is logged in?

2. to let JPA know who is logged in we need to implement AuditorAware Interface and register a bean for it
```java
@Component("auditAwareImpl")
public class JwtAuditAwareImpl implements AuditorAware<String> {
    @Autowired
    private HttpServletRequest request;
    @Autowired 
    private Environment env;
    @Override
    public Optional<String> getCurrentAuditor() {
        // Get Authorization header
        String authHeader = request.getHeader("Authorization");
        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            // Extract token
            String token = authHeader.substring(7);
            try {
                // Parse token and extract username
                // This depends on your JWT structure and library
                String username = extractUsernameFromToken(token);
                return Optional.ofNullable(username);
            } catch (Exception e) {
                // Log the exception
                return Optional.empty();
            }
        }
        return Optional.empty();
    }
    private String extractUsernameFromToken(String token) {
        // Using JWT library like jjwt
        Claims claims = Jwts.parser()
                .setSigningKey(env.getProperty("jwtsecret")) // Use your actual signing key
                .parseClaimsJws(token)
                .getBody();
        return claims.getSubject(); // Assuming subject contains username
    }
}
```

3. enable auditing by annotating the entry point with the bean we just injected
```java
@SpringApplication
@EnableJpaRepositories("com.sharmachait.wazir")
@EntityScan("com.sharmachait.wazir.model")
@EnableJpaAuditing(auditorAwareRef = "auditAwareImpl")
public class Wazir{}
```

remember we can always expect Authentication in the method params of the controller


# custom validations for things like passwords
1. create the annotation interface
```java
@Documented
@Constraint(validatedBy = PasswordStrengthValidator.class)
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface PasswordValidator{
	String message() default "Please choose a string password";
	Class<?>[] groups() default {};
	Class<? extends Payload>[] payload() default {};
}
```
PasswordStrengthValidator is where the actual logic is implmented
### Example
For example, you might have different classes that implement the `Payload` interface:
```java
class MyPayload implements Payload { /* Implementation */ } 
class AnotherPayload implements Payload { /* Implementation */ }
```
Then you could use the `payload` attribute like this:
```java 
@PasswordValidator(payload = {MyPayload.class, AnotherPayload.class})
```

2. implement the class that has the validation logic
```java
public class PasswordStrengthValidator implements 
		ConstraintValidator<PasswordValidator, String>{
	Set<String> weakPasswords;
	@Override 
	public void initialize(PasswordValidator passwordValidator){
		weakPasswords = new HashSet<>(Arrays.asList("12345", "password", "qwerty"));
	}
	@Override
	public boolean isValid(String password, ConstraintValidatorContext ctx){
		return password!=null && (!weakPasswords.contains(password));
	}
}
```

3. use the annotation
```java
@PasswordValidator
private String pwd;
```
# another example, matching two fields like password and confirm password

if we want to perform validations on two fields the field fieldMatch and the list are required
because it takes two values it need to be mentioned on top of the class with the two values that we need to validate
```java
@Constraint(valdiatedBy = FieldsValueMathValidator.class)
@Target({ ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface fieldsValueMatch{
	String message() default "values dont match";
	Class<?>[] groups() default {};
	Class<? extends Payload>[] payload() default {};
	
	String field();
	String fieldMatch();
	
	@Target({ElementType.TYPE})
	@Retention(RetentionPolicy.RUNTIME)
	@interface List{
		FieldsValueMatch[] value();
	}
}
```

for the fieldsValueMatchValidator 
```java
public class FieldsValueMathValidator implements
		ConstraintValidator<FieldsValueMatch, Object> {
	private String field;//these are just the names of the fields
	private String fieldMatch;
	@Override
	public void initialize(FieldsValueMatch ann){
		this.field = ann.field();
		this.fieldMathc = ann.fieldMathc();
	}
	@Override
	public boolean isValid(Object value, ContrainsValidatorContext ctx){
		Object fieldValue = new BeanWrapperImpl(value).getPropertyValue(field);
			//this is just reflection
		Object fieldMatchValue = new BeanWrapperImpl(value).getPropertyValue(fieldMatch);
		if(fieldValue!=null){
			return fieldValue.equals(fieldValueMatch);
		}
		return fieldValueMatch == null;
	}
}
```

to use this multi value custom validation
```java
@Data
@Entity
@FieldsValueMatch.List({
	@FieldsValueMatch(
		field = "password",
		fieldMatch = "confirmPassword",
		message = "Passwords do not match!"
	),
	@FieldsValueMatch(
		field = "email",
		fieldMatch = "confirmEmail",
		message = "Emails do not match!"
	)
})
public class User {
	private String password;
	@Transient
	private String confirmPassword;
	@Email
	private String email;
	@Email
	@Transient
	private String confirmEmail;
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id
}
```
@Transient is used to tell JPA that a field or property in an entity that should not be persisted to the database
# sorting
two kinds of sorting supported by the spring data jpa
1. static 
```java
List<Person> findByOrderByName();
List<Person> findByOrderByNameDesc();
```
2. dynamic
choosing dynamically at runtime what the data should be sorted on when fetching from the database
send the Sort object as well
```java
Sort sort = Sort.by("name").descending().and(Sort.by("age").ascending());
List<Courses> = coursesRepository.findAll(sort);
```

JPA will sort the data based on the sort object for us for the built in methods only that are being implemented by the spring data jpa 
but for any custom methods in the repository methods we need to define them so that they expect a sort object
```java
public interface CoursesRepository extends JpaRepository<Courses, Long> {
    List<Courses> findBySomeCriteria(String criteria, Sort sort);
    List<Courses> findBySomeCriteria();
	
	@Async
	Future<Courses> findByName(String name);
}
```
the string criteria is a must have and it will not work without it
the criteria will be used to filter and sort used to sort them
# pagination
use the Pageable Object like the sort object in the queries
```java
public interface PersonRepository extends JpaRepository<Courses, Long> {
    Page<Person> findByName(String name,Pageable pageable);
}

Pageable pageable = PageRequest.of(0,5,Sort.by("name").desceding());
Page<Person> personPage = personRepository.findByName("chaitanya", pageable);
```

where 0 is the start index of the first page and 5 is the number of records per page
the personPage will have at max 5 entries and meta data about the page, like total number of records, number of pages, current page number and if next page is available   
we can expect the start and the field that is name from the @RequestParams anbd show content dynamically
# custom queries
allowed using three annotations
1. @Query - allows writing queries using JPQL or native SQL
	1. when using native SQL we need to provide `@Query(nativeQuery = true)`
2. @NamedQuery - used to maintain JPQL with names in a single place
3. @NamedNativeQuery - used to maintain native SQL with names in a single place
Named and NamedNative allow us to manage the queries in the entity class itself
### jpql example
queries are written on the Entity instead of the table name
```java
@Query("SELECT c FROM Contact c WHERE c.containsId = ?1 ORDER BY c.createdAt DESC")
List<Contact> findByIdOrderByCreatedDesc(Long id);
```
the ?1 is used to refer to the first parameter being passed to the method
equivalent in derived methods would be
```java
List<Contact> findByContainsIdOrderByCreatedAtDesc(Long id);
```

instead of using ?1 we can also use the name of the parameter directly
```java
@Query("SELECT c FROM c WHERE c.status = :status")
Page<Contact> findByStatus(String status, Pageable pageable);
```

in Native Sql queries we have to use real table names instead of just the model names and also the real column names in the database instead of the field names of the entity class
### update with custom queries
to be able to update insert or delete records in the database with custom queries along with @Query annotation we also need to mention two other annotation
1. @Transactional
2. @Modifying
```java
@Modifying
@Transactional
@Query("UPDATE  Contact c SET c.status = ?1 WHERE c.contactId = ?2")
int updateStatusById(String status, int id);
```
this will return the number of rows that were affected

### NamedQuery & NamedNativeQuery
Named and NamedNative allow us to manage the queries in the entity class itself
the Name of the query is `<Name of Entity>.<Name of method in the repository>`
```java
@Entity
@NamedQeury(name="Contact.findOpenMessages", query="SELECT c FROM Contact c WHERE c.status = :status")
public class Contact extends BaseEntity{

}
```
the NamedNativeQuery annotation also expects the resultClass as a parameter
```java
@Entity
@NamedQeury(name="Contact.findOpenMessages", query="SELECT c FROM Contact c WHERE c.status = :status", resultClass = Contact.class)
public class Contact extends BaseEntity{

}
```
in case of named native query the method in the repository should also have the @query annotation with nativeQuery=true parameter
```java
@Query(nativeQuery = true)
public Contact findOpenMessages(String status);
```

we can have multiple named queries and named native queries with
![[Pasted image 20241022074030.png]]
the methods in the repository can still expect pageable and sort objects as well if we use namedQueries
but if we use named native queries, and want to use pageable as well then we need to write a named native query that counts the number of records for us as well

```java
@SqlResultSetMappings({
@SqlResultSetMapping(name = "SqlResultSetMapping.count" columns = @ColumnResult(name = "cnt"))
})
@NamedNativeQueries({
	@NamedNativeQuery(name = "Contact.findOpenMessages",
		query = "SELECT * FROM contact_msg c WHERE c.status = :status",
		resultClass = Contact.class),
	@NamedNativeQuery(name = "Contact.findOpenMessages.count",
		query = "SELECT count(*) as cnt FROM contact_msg c WHERE c.status = :status",
		resultSetMapping = "SqlResultSetMapping.count"),
})
public class Contact extends BaseEntity {

}
```
where findOpenMessages should be a function  in the Repository that expects a Pageable

# Entity Inheritance in DB

Imagine a scenario where we have an entity that us resource but there are three kinds of resources, Audio Video and Text
should we have three tables?
should we have one table Resource with columns of all the properties? should we have a column that defined the type of resource we are working with?
how to do that with JPA?

![[Pasted image 20241026151958.png]]

## single table strategy
```java
@Data
@Entity
@Inheritance
@DiscriminatorColumn(name = "resource_type")
public class Resource{
	@Id
	@Generated
	private Long id;
	private String name;
}

@Data
@EqualsAndHashCode(callSuper = true)
@Entity
@DiscriminatorValue("Video")
public class Video extends Resource{
	private int length;
}

@Data
@EqualsAndHashCode(callSuper = true)
@Entity
@DiscriminatorValue("File")
public class File extends Resource{
	private String type;
}

@Data
@EqualsAndHashCode(callSuper = true)
@Entity
@DiscriminatorValue("Text")
public class Text extends Resource{
	private String content;
}
```

this will create a single resource table in the database with three columns from the children entity as well, namely length type content
to differentiate the rows of different types from each other we have the discriminator column
we dont have to populate that value. Spring will do it for us. it will create a column named resource_type and populate the discriminator values for us, all we have to do is save a Video type object or file or text
create repositories for the children classes not the parent

## Join table strategy
in this strategy each subclass will have its own table and have a foreign key into the resource table so that we can get the common values out from that table
not suitable when we have to query the entire inheritance hierarchy all the time as joins are slow
doesnt require discriminator column
```java
@Data
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public class Resource{
	@Id
	@Generated
	private Long id;
	private String name;
}

@Data
@EqualsAndHashCode(callSuper = true)
@Entity
public class Video extends Resource{
	private int length;
}

@Data
@EqualsAndHashCode(callSuper = true)
@Entity
public class File extends Resource{
	private String type;
}

@Data
@EqualsAndHashCode(callSuper = true)
@Entity
public class Text extends Resource{
	private String content;
}
```

and thats all we need to do
now when ever we do something like videoRepository.save(video)
it will run two inserts for us

## separate table strategy
3 tables for the 3 types are created
with no common properties fetched out into a 4th table
most efficient queries
good for small number of types or sub classes
```java
@Data
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public class Resource{
	@Id
	@Generated
	private Long id;
	private String name;
}

@Data
@EqualsAndHashCode(callSuper = true)
@Entity
@Polymorphism(type = PolymorphismType.EXPLICIT)
public class Video extends Resource{
	private int length;
}

@Data
@EqualsAndHashCode(callSuper = true)
@Entity
@Polymorphism(type = PolymorphismType.EXPLICIT)
public class File extends Resource{
	private String type;
}

@Data
@EqualsAndHashCode(callSuper = true)
@Entity
@Polymorphism(type = PolymorphismType.EXPLICIT)
public class Text extends Resource{
	private String content;
}
```

now if we query the resource repository it will create a union on all the tables and thats slow
if we want only the columns related to the resource, and query on resource, we want to exclude Video, file and text from that query
we can do that by mentioning `@Polymorphism(type = PolymorphismType.EXPLICIT)` on top of the sub classes

# composite primary keys and embedded fields
achieved via Embedded entities
create a class OrderId.java and the columns in it will server as the primary key
```java
@Data
@AllArgsContructor
@NoArgsConstructor
@Embeddable
public class OrderId implements Serializable{
	private String username;
	private LocalDateTime orderDate;
}

@Data
@AllArgsContructor
@NoArgsConstructor
@Entity
public class Order {
	@EmbeddedId
	private OrderId id;
	private String orderInfo;
	@Embedded
	private OtherProperties;
}

@Embeddable
public class OtherProperties{

}
```

![[Pasted image 20240920135741.png]]
![[Pasted image 20240920140209.png]]
use @Valid annotation on method or fields to tell spring we want the value to be validated

![[Pasted image 20240920141258.png]]
![[Pasted image 20240920141309.png]]

# with models using lombok
```java
import lombok.Data;
import jakarta.validation.constraints.*;

@Data
public class Contact {

	@NotBlank(message="error message")
	@Size(min=3, message = "Name must be at least 3 characters long")
	private String name;
	
	@NotBlank(message="error message")
	@Pattern(regexp = "(^$|[0-9]{10})", message="error message")
	private String mobileNum;
	
	@NotBlank(message="error message")
	@Email(message = "error message")
	private String email;
	
	@NotBlank(message="error message")
	@Size(min=5, message = "Subject must be at least 5 characters long")
	private String subject;
	
	@NotBlank(message="error message")
	@Size(min=10, message = "Message must be at least 10 characters long")
	private String message;
}
```
# @NotNull vs @NotEmpty vs @NotBlank
### @NotNull - allows empty values of length zero, checks not null
### @NotEmpty - size should be greater than zero, checks not null
### @NotBlank - trimmed length should be greater than zero, checks not null

without the @Valid annotation validations will not performed
```java
@PostMapping  
public ResponseEntity<ResponseDto> createAccount(@Valid @RequestBody CustomerDto customer) throws CustomerAlreadyExistsException {  
    accountsService.createAccount(customer);  
    return ResponseEntity  
            .status(HttpStatus.CREATED)  
            .body(new ResponseDto(AccountsConstants.STATUS_201, AccountsConstants.MESSAGE_201));  
}
@GetMapping  
public ResponseEntity<CustomerDto> getAccountDetails(@RequestParam  
                                                         @Pattern(regexp="(^$|[0-9]{10})", message = "Mobile number must be 10 digits")  
                                                         String mobileNumber){  
    CustomerDto customerDto = accountsService.getAccount(mobileNumber);  
    return ResponseEntity  
            .status(HttpStatus.OK)  
            .body(customerDto);  
}
```
if any errors are detected they are thrown and caught in the errors object

with this validations are performed and error is thrown but we need to specify how to send the error to the frontend application
we can do that by extending our global exception handling class and overriding the default implementation of the handler
```java
@ControllerAdvice  
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {
	@Override  
	protected ResponseEntity<Object> handleMethodArgumentNotValid(MethodArgumentNotValidException ex,   
	HttpHeaders headers,   
	HttpStatusCode status,   
	WebRequest request) {  
	    Map<String, String> validationErrors = new HashMap<>();  
	    List<ObjectError> validationErrorList = ex.getBindingResult().getAllErrors();  
	    validationErrorList.forEach(error -> {  
	        String errorField = ((FieldError)error).getField();  
	        String errorMessage = error.getDefaultMessage();  
	        validationErrors.put(errorField, errorMessage);  
	    });    
	    return new ResponseEntity<>(validationErrors, HttpStatus.BAD_REQUEST);  
	}
}
```
this class has a method to handle validation exceptions

## validating embedded properties

lets say we have a customer class and an address class, the customer has an address and the fields of the address class has annotations on it to validate the data, we  will need to validate the address field in the customer class to tell spring boot that the address needs to be validated as well

```java
@Embeddable
public class Address {
    private String street;
	@Length(max=30)
    private String city;
    private String zipCode;
}
@Entity
public class Person {
    @Id
    private Long id;
    private String name;
    @Valid
    @Embedded
    private Address address;
}
```
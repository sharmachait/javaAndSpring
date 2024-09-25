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

we need to inform the view that some validations need to happen for the model 
```java
@RequestMapping("/contact")
public String displayContact(Model model){
	model.addAttribute("contact", new Contact());
	return "contact.html";
}
```
this is the original get method that returns the view

for the post endpoint that will receive the model from the frontend
we basically need to tell the framework that this value is from the view and that it has to perform validations on it when mapping the view contact object to it
```java
@RequestMapping("/saveMsg", method = POST)
public String saveMessage(@Valid @ModelAttribute("contact") Contact contact, Errors errors){
	if(errors.hasErrors()){
		return "contact.html";
	}
	contactService.saveMessage(contact);
	return "redirect:/contact.html";
}
```
without the @Valid annotation validations will not performed
if any errors are detected they are thrown and caught in the errors object

the difference between the two is that contact.html will return the errors to the frontend with the values that were populated
but the redirect one will trigger the entire workflow of that page again

and now in the view when ever we add to this attribute MVC framework will do the validations for us
in the view on the form where we are populating the values we need to specify the object
and instead of name and id for the input tags we can use thymeleaf field tag
```html
<form th:object="${contact}" th:action="@{/saveMsg}" method="post" class="signin-form">
	<input type="text" th:field="*{name}"/>
</form>
```
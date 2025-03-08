# MVC
@ControllerAdvice - specialized @Component, used to create interceptors of requests
@ExceptionHandler

```java
@ControllerAdvice  
public class GlobalExceptionHandler {  
    @ExceptionHandler(Exception.class)  
    public ResponseEntity<ErrorResponseDto> handleException(Exception ex, WebRequest request) {  
        ErrorResponseDto errorResponseDto = new ErrorResponseDto(  
                request.getDescription(false), // returns the api path where this exception was thrown  
                HttpStatus.INTERNAL_SERVER_ERROR,  
                ex.getMessage(),  
                LocalDateTime.now()  
        );        
        return new ResponseEntity<>(errorResponseDto, HttpStatus.INTERNAL_SERVER_ERROR);  
    }  
    @ExceptionHandler(CustomerAlreadyExistsException.class)  
    public ResponseEntity<ErrorResponseDto> handleCustomerAlreadyExistsException(CustomerAlreadyExistsException ex, WebRequest request) {  
        ErrorResponseDto errorResponseDto = new ErrorResponseDto(  
                request.getDescription(false), // returns the api path where this exception was thrown  
                HttpStatus.BAD_REQUEST,  
                ex.getMessage(),  
                LocalDateTime.now()  
        );        
        return new ResponseEntity<>(errorResponseDto, HttpStatus.BAD_REQUEST);  
    }  
    @ExceptionHandler(ResourceNotFoundException.class)  
    public ResponseEntity<ErrorResponseDto> handleResourceNotFoundException(ResourceNotFoundException ex, WebRequest request) {  
        ErrorResponseDto errorResponseDto = new ErrorResponseDto(  
                request.getDescription(false), // returns the api path where this exception was thrown  
                HttpStatus.NOT_FOUND,  
                ex.getMessage(),  
                LocalDateTime.now()  
        );        
        return new ResponseEntity<>(errorResponseDto, HttpStatus.BAD_REQUEST);  
    }
}
```

control only reaches this method in case of an uncaught exception of the same type as provided in the @ExceptionHandler
we can also achieve this with regular AOP
```java
@Aspect
@Component
public class ExceptionHandlingAspect {
    @Around("execution(* com.example.controller..*(..))")
    public Object handleExceptions(ProceedingJoinPoint joinPoint) {
        try {
            return joinPoint.proceed();
        } catch (Exception e) {
            ModelAndView modelAndView = new ModelAndView("error");
            modelAndView.addObject("errorMessage", e.getMessage());
            return modelAndView; // return your ModelAndView here
        }
    }
}
```

if @ExceptionHandler method is defined inside a controller then that method to handle exceptions is applicable to only the methods in that class

# API
```java
@Slf4j
@RestControllerAdvice
public class GlobalApiExceptionHandler 
	extends ResponseEntityExceptionHandler {
	
	@Override
	protected ResponseEntity<Object> handlerMethodArgumentsNotValid(
		MethodArgumentNotValidException ex
		, HttpHeaders headers
		, HttpStatus status
		, WebRequest req){
		String res = ex.getBindingResult().toString();
		return new ResponseEntity(res, HttpStatus.BAD_REQUEST);
	}
	
	@ExceptionHandler({Exception.class})
	public ResponseEntity<String> exceptionHandler(Exception exp){
		return new ResponseEntity(exp.getMessage()
			,HttpStatus.INTERNAL_SERVER_ERROR);
	}
}
```
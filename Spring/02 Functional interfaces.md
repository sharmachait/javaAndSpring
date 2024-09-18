# Runnable doesnt expect input and doesnt return
runnable.run
# Runnable + return == Callable
Callable.call
# Runnable + input == Consumer
Consumer.accept
# Consumer + return == Supplier
supplier.get

Supplier returns the same Type it takes
functional interface that takes an input and doesnt return anything
![[WhatsApp Image 2024-09-08 at 22.37.17_99da59aa.jpg]]
## Function
functional interface that takes input as well as return 
![[WhatsApp Image 2024-09-08 at 22.35.22_793b9fd5.jpg]]
#### function applys

to create a bean programmatically we need to use a Supplier in the 
`context.registBean("name",datatype,supplier)`
method
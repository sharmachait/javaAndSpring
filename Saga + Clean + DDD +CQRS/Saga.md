means to create a chain of local ACID transactions to finalize a long running transaction across services 
the idea is that local transactions publish domain events that trigger local transactions in other services
to maintain consistency across transactions in multiple services Outbox pattern is used

![[Pasted image 20241127125038.png]]
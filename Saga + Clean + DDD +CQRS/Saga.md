means to create a chain of local ACID transactions to finalize a long running transaction across services 
the idea is that local transactions publish domain events that trigger local transactions in other services to achieve distributed transactions
to maintain consistency across transactions in multiple services Outbox pattern is used

![[Pasted image 20241127125038.png]]

CQRS means command and query segregation principle, we should do write operations to a database and use read replicas for queries. the read replicas can only be eventually consistent
## DDD
1. Aggregates - 
	1. encapsulation of Entities and Value Objects
	2. The aggregate encapsulates these objects and enforces business rules and invariants within its boundary, ensuring that all modifications happen through a single entry point called the **aggregate root**
	3. entire aggregate should be persisted in a single transaction, as the data needs to be in consistent state for all the participating entities in the aggregate
	4. **Order** is the **aggregate root**. It controls access to `OrderItem`s.
	5. The `addItem` and `removeItem` methods enforce business rules and maintain consistency.
	6. External code cannot modify `OrderItem`s directly; it must go through the `Order`.
	7. The `Order` maintains its own invariants, such as valid quantities and allowed status transitions.
	8. The entire aggregate (`Order` and its `OrderItem`s) is treated as one consistency boundary.
2. Entites
3. Value Objects
### CQRS
Materialized views can be used to do read operations in CQRS
Event sourcing can be used in CQRS as the write operations, events are like AOF files in redis to get eventual consistency by apply atomic events

![[project-overview-section-1.png]]

## hexagonal architecture
uses ports and adaptors to divide the software into outside(infrastructure layer) and inside(domain layer)
infrastructure will include things that may change over time like DB cloud services UI message queues api, even the framework we are working on
##### primary adaptors
interact with the domain layer directly, the use  the input ports to interact with the domain layer,

example - rest api controllers or CLI application that can talk to the domain layer to do operations

**primary adapters** are the entry points into your application (e.g., REST controllers, message listeners). They:
- Accept external data, often as DTOs.
- Kafka Consumer
- and a redis context to read data

input ports are typically placed in the **application layer** (sometimes called the service or use case layer), sitting between the domain (business logic) and the external adapters (like controllers, APIs, or UI)
##### secondary adaptors
implement the external dependencies by implementing the output ports

**output ports are interfaces defined in the domain layer** that represent the operations the domain needs to perform on external systems or infrastructure (like databases, message queues, or external APIs). These interfaces allow the **business logic to remain independent of any specific implementation details** of those external systems

user input is handled by the primary adaptors and passed to the core logic
the core logic handles the business and uses the secondary adaptors  to persist the data 

user receives the data from the application core using the primary adaptors

The domain defines an interface `OrderRepository` (output port) with methods like `save(Order order)`. The infrastructure layer implements this interface using Spring Data JPA

Entities should encapsulate critical business rules in object creation, like validations
Use Cases should do application specific business rules like the logic for temporary discounts

DDD features two types of services - Domain and Application services, domain services are the services that interact with the domain layer and have logic that can not be encapsulated in the domain classes, while application services talk to the outside would using ports and adaptors lie email service
the application service should do the data adaptor calls and forward it into the domain service and entities

![[Pasted image 20250507080958.png]]
if domain leys depends on data layer, data layer should have an interface that needs to be implemented by the domain layer, so now data layer depends on the domain layer to implement the interface, thats called the dependency inversion principle

what that allows us to do is mock the data layer safely and we can test the business layer without the data layer
and we can change the under lying data layer without affecting the domain layer
the domain layer should not have any framework dependency in it, so how to do dependency injection of the data layer ? 
can be done using a bean registration class in the container of the application 
The domain/application layer defines _what_ operations it offers (input ports as interfaces). External layers (like UI, REST controllers, CLI, tests) depend on these interfaces, not on concrete implementations
the API layer can use the input port and choose the implementation in the domain layer to use, allowing us to develop the presenation layer independently like rest controllers or CLI or TCP application
![[order-service-hexagonal-section-2-share.png]]
All the interfaces must be in the domain layer 

### to visualize the dependency graph  
```shell  
mvn com.github.ferstl:depgraph-maven-plugin:aggregate -DcreateImage=true -DreduceEdges=false -Dscope=compile "-Dincludes=com.food.ordering.system*:*"  
```

![[Pasted image 20250508000448.png]]
## DDD
2 types of DDD models
1. Strategic DDD - focuses on boundaries, creates Bounded context per each domain
	1. boundary in DDD help group functionalities in a system
	2. a bounded context can have one or more domains 
	3. ![[Pasted image 20250508090641.png]]
2. Tactical DDD - focuses on the implementation details of the domain logic
	1. Entities - 2 Entities with the same identifier are considered to be the same object even if other properties are different
		1. entities are mutable, when we perform some validation or operation on a entity it will update its own fields, but we should not create setter methods 
	2. Aggregates - group of Entity object connected and updated together for consistency
		1. example - OrderProcessAggregate that processes Order and handles updated to OrderItema nd Product as well as Order, done via the AggregateRoot, Aggregates should only be referenced via the Aggregate Root, all the state altering operations should also go through the aggregate root
		2. the root of the aggregate should be the one that is accessed from outside the aggregate by other aggregates maybe
	3. Value Object - Money class instead of BigDecimal, Address class instead of String, allows to validate the values as well when creating the object, Value Objects are immutable, once created should not be updated. means 2 objects with the same value should be interchangeable
	4. Domain Events - allows De coupling different domains that are in different contexts. this leads to eventual consistency. when we save the changes for one domain, we can fire events to trigger transactions in other domains. via an even source system, like kafka that can play back all the events
	5. domain service - encapsulates the logic that can not fit into an aggregate, logic that spans multiple aggregates. can communicate with other domain services if necessary
	6. Application services - used to communicate with components outside the domain and orchestrate transactions in the domain. Does not contain any business logic. only orchestration and communication logic. Event listeners are special kind of application services which should have a domain service to perform business logic, should use the JPA repositories 
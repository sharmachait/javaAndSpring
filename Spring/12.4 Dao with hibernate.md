#### Session factory - heavy object, should only be one instance of it
#### Entity Manager factory - session factory
#### Session - single threaded short lived object cheap to create
wrapper of JDBC connection
#### Entity Manager - JPA equivalent of session
#### Transaction - single threaded short lived object to define transaction boundaries
#### Entity Transaction - JPA equivalent of transaction

##### persistence context - a context(lifecycle) of data to be persisted

the states are 
1. Transient - the entity has just been instantiated and is not associated with a persistence context, has no representation in the database and no ID has been assigned to it (unless generator used)
2. Managed or Persistent  - entity has been associated with an ID and is present in the persistence context, may or may not exist in the database yet, hibernate can save something, but it may occur some time later
3. Detached - the entity is associated with an identifier but is no longer associated with a persistence context. maybe the persistence context was closed or the instance was evicted from the context
4. Removed - the entity has an associated ID and is associated with persistence context, but is scheduled for removal from the database

Detached entities throw an error very common to see, happens when working outside the session scope or a closed session
Spring Data JPA will do a transaction implicitly, therefore the error can happen when accessing entity properties outside of a transaction

#### caching 

First level cache - default caching, in the persistence context
changes made to DB outside the context will not be reflected

second level cache - is disabled by default, recommended to be handled per entity basis, JVM level

issues caching can cause
when we try to save alot of objects it can cause out of memory error, long running transactions can deplete the transaction pool
flush() and clear() methods can be used to clear session cache


## DAO with hibernate

```java
@RequiredArgsConstructor  
@Component  
public class AuthorHibernateDao implements AuthorDao {  
    private final EntityManagerFactory emf;  
    private EntityManager getEntityManager(){  
        return emf.createEntityManager();  
    }  
    @Override  
    public Author getById(Long id) {  
        return getEntityManager().find(Author.class, id);  
    }  
    @Override  
    public Author getByName(String firstName, String lastName) {  
        TypedQuery<Author> query = getEntityManager().createQuery("select a from Author a" +  
                " where a.firstName = :first_name and a.lastName = :last_name", Author.class);  
        query.setParameter("first_name", firstName);  
        query.setParameter("last_name", lastName);  
        return query.getSingleResult();  
    }  
  
    @Override  
    public Author saveAuthor(Author author) {  
        EntityManager em = getEntityManager();  
        em.getTransaction().begin();  
        em.persist(author);  
        em.flush();  
        em.getTransaction().commit();  
        return author;  
    }  
}
```
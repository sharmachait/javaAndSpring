We dont need to create objects an populate them, we can use RowMappers instead

```java
@Component 
public class AuthorMapper implements RowMapper<Author> {  
    @Override  
    public Author mapRow(ResultSet rs, int rowNum) throws SQLException {  
        Author author = new Author();  
        author.setId(rs.getLong("id"));  
        author.setFirstName(rs.getString("first_name"));  
        author.setLastName(rs.getString("last_name"));  
        return author;  
    }  
}
  
@RequiredArgsConstructor  
@Repository  
public class AuthorDaoImplJdbcTemplate implements AuthorDao{  
    private final AuthorMapper authorMapper;  
    private final JdbcTemplate jdbcTemplate;  
    @Override  
    public Author getById(Long id) {  
        return jdbcTemplate.queryForObject("SELECT * FROM author where id = ?", authorMapper,  id);  
    }  
  
    @Override  
    public void deleteById(Long id) {  
        jdbcTemplate.update("delete from author where id = ?", id);  
    }  
  
    @Override  
    public Author getByName(String firstName, String lastName) {  
  
        return jdbcTemplate.queryForObject("SELECT * FROM author where first_name = ? and last_name = ?",  
                authorMapper,  
                firstName,  
                lastName);  
    }  
  
    @Override  
    public Author saveAuthor(Author author) {  
        jdbcTemplate.update("INSERT INTO author (first_name, last_name) VALUES (?, ?)",  
                author.getFirstName(),  
                author.getLastName());  
        return getByName(author.getFirstName(), author.getLastName());  
    }  
  
    @Override  
    public Author updateAuthor(Author author) {  
        author = getByName(author.getFirstName(), author.getLastName());  
        jdbcTemplate.update(  
                "UPDATE author SET first_name = ?, last_name = ? where id = ?",  
                author.getFirstName()+"stephen",  
                author.getLastName()+"king",  
                author.getId());  
        return getById(author.getId());  
    }  
}
```

  ## joins with jdbc template and mappers

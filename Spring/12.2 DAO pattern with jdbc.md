
## Raw JDBC Dao pattern

The datasource is injected from the  config we have in the application.yml file

```java
@RequiredArgsConstructor  
@Repository  
public class AuthorDaoImpl implements AuthorDao {  
  
  private final DataSource source;
  
  @Override  
  public Author getById(Long id) {  
  
    Connection connection = null;  
    PreparedStatement ps = null;  
    ResultSet resultSet = null;  
  
    try {  
      connection = source.getConnection();  
      ps = connection.prepareStatement("select * from author where id = ?");  
      ps.setLong(1, id);  
      resultSet = ps.executeQuery();  
      if (resultSet.next()) {  
        Author author = new Author();  
        author.setId(id);  
        author.setFirstName(resultSet.getString("first_name"));  
        author.setLastName(resultSet.getString("last_name"));  
        return author;  
      }  
    } catch (SQLException e) {  
      e.printStackTrace();  
    } finally {  
      try {  
        if (resultSet != null)  
          resultSet.close();  
        if(ps!=null)  
          ps.close();  
        if (connection != null)  
          connection.close();  
      } catch (Exception e) {  
        e.printStackTrace();  
      }  
    }    
    return null;  
  }  
}
```


## Calling StoredProcedures and functions with JDBC / Callable statements

```java
@Autowired
private JdbcTemplate jdbcTemplate;

// Call stored procedure
public void callProcedureWithJdbcTemplate() {
    jdbcTemplate.execute("{ call update_user_status(?, ?) }",
        (CallableStatementCallback<Object>) cs -> {
            cs.setInt(1, userId);
            cs.setString(2, "ACTIVE");
            return cs.execute();
        }
    );
}

// Call function
public Integer callFunctionWithJdbcTemplate() {
    return jdbcTemplate.execute(
        "{? = call calculate_total(?)}",
        (CallableStatementCallback<Integer>) cs -> {
            cs.registerOutParameter(1, Types.INTEGER);
            cs.setInt(2, userId);
            cs.execute();
            return cs.getInt(1);
        }
    );
}

@Autowired
private JdbcTemplate jdbcTemplate;

// Stored Procedure returning ResultSet
public List<User> getUsersFromProcedure() {
    return jdbcTemplate.execute(
        "{call get_users_by_status(?)}",
        (CallableStatementCallback<List<User>>) cs -> {
            cs.setString(1, "ACTIVE");
            ResultSet rs = cs.executeQuery();
            List<User> users = new ArrayList<>();
            
            while(rs.next()) {
                User user = new User();
                user.setId(rs.getLong("id"));
                user.setName(rs.getString("name"));
                user.setEmail(rs.getString("email"));
                users.add(user);
            }
            return users;
        }
    );
}

// Function returning ResultSet
public List<Order> getOrdersFromFunction() {
    return jdbcTemplate.execute(
        "{? = call get_orders_by_customer(?)}",
        (CallableStatementCallback<List<Order>>) cs -> {
            cs.registerOutParameter(1, Types.REF_CURSOR); // For Oracle
            cs.setLong(2, customerId);
            cs.execute();
            
            ResultSet rs = (ResultSet) cs.getObject(1);
            List<Order> orders = new ArrayList<>();
            
            while(rs.next()) {
                Order order = new Order();
                order.setId(rs.getLong("order_id"));
                order.setAmount(rs.getBigDecimal("amount"));
                order.setDate(rs.getDate("order_date"));
                orders.add(order);
            }
            return orders;
        }
    );
}

// Using RowMapper for cleaner code
private final RowMapper<User> userRowMapper = (rs, rowNum) -> {
    User user = new User();
    user.setId(rs.getLong("id"));
    user.setName(rs.getString("name"));
    user.setEmail(rs.getString("email"));
    return user;
};

public List<User> getUsersWithRowMapper() {
    return jdbcTemplate.execute(
        "{call get_users_by_department(?)}",
        (CallableStatementCallback<List<User>>) cs -> {
            cs.setString(1, "IT");
            ResultSet rs = cs.executeQuery();
            List<User> users = new ArrayList<>();
            
            while(rs.next()) {
                users.add(userRowMapper.mapRow(rs, rs.getRow()));
            }
            return users;
        }
    );
}

// Multiple ResultSets
public Map<String, List<Object>> getMultipleResultSets() {
    return jdbcTemplate.execute(
        "{call get_employee_details(?)}",
        (CallableStatementCallback<Map<String, List<Object>>>) cs -> {
            cs.setLong(1, employeeId);
            Map<String, List<Object>> results = new HashMap<>();
            
            // First ResultSet - Employee details
            ResultSet rs = cs.executeQuery();
            List<Employee> employees = new ArrayList<>();
            while(rs.next()) {
                Employee emp = new Employee();
                emp.setId(rs.getLong("id"));
                emp.setName(rs.getString("name"));
                employees.add(emp);
            }
            results.put("employees", employees);
            
            // Second ResultSet - Salary details
            if (cs.getMoreResults()) {
                rs = cs.getResultSet();
                List<Salary> salaries = new ArrayList<>();
                while(rs.next()) {
                    Salary salary = new Salary();
                    salary.setAmount(rs.getBigDecimal("amount"));
                    salary.setDate(rs.getDate("payment_date"));
                    salaries.add(salary);
                }
                results.put("salaries", salaries);
            }
            
            return results;
        }
    );
}
```

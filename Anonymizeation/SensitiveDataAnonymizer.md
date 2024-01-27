# Anonymize Sensitive Data of DB in larger system
Anonymization Script

```JAVA
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;
import com.github.javafaker.Faker;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.List;
import java.util.Map;
import java.util.HashMap;
import java.util.Map;
import javax.transaction.Transactional;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

@Component
public class SensitiveDataAnonymizer {

    private final Faker faker = new Faker();

    @Autowired
    private JdbcTemplate jdbcTemplate;

    private String scriptUpdateTime;
    private String lineBreak;
    private ExecutorService executorService;
    private AnonymizeEmployeeAssociations employeeAssociation;

    public SensitiveDataAnonymizer() {
        this.scriptUpdateTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        this.lineBreak = "-".repeat(100);
        this.employeeAssociation = new AnonymizeEmployeeAssociations(scriptUpdateTime);
        this.executorService = Executors.newFixedThreadPool(8);
        printClassNames();
    }

    @Transactional
    public void anonymize() {
        employeeAssociation.executeSql("SET foreign_key_checks = 0;");
        truncateTables();
        System.out.println("\n" + lineBreak);
        employeeAssociation.addIndexes();
        System.out.println("ADDED ADDITIONAL INDEXES TO SeatOpenBookiking AND SeatOprnBookingAction TABLES");
        System.out.println(lineBreak);
        anonymizeTablesByEmployee();
        System.out.println("\n" + lineBreak);
        System.out.println("ANONYMIZE SENSITIVE INFORMATION ASSOCIATED WITH EMPLOYEES");
        System.out.println(lineBreak);
        employeeAssociation.anonymize();
        System.out.println(lineBreak);
        System.out.println("\n" + lineBreak);
        employeeAssociation.dropIndexes();
        truncateTable("anonymized_sensitive_values");
        System.out.println("TRUNCATE anonymized_sensitive_values TABLE");
        System.out.println("DROPPED ADDITIONAL INDEXES ON SeatOpenBookiking AND SeatOprnBookingAction TABLES");
        System.out.println(lineBreak);
        employeeAssociation.executeSql("SET foreign_key_checks = 1;");
        System.out.println("\n" + lineBreak);
        System.out.println("REBUILD PeopleReportCache started at " + LocalDateTime.now().toString());
        PeopleReportCache.rebuildCache();
        System.out.println("REBUILD PeopleReportCache completed at " + LocalDateTime.now().toString());
        System.out.println(lineBreak);
        System.out.println("\n");
    }

    public void truncateTables() {
        jdbcTemplate.execute("TRUNCATE TABLE anonymized_sensitive_values");
        jdbcTemplate.execute("TRUNCATE TABLE api_keys");
        jdbcTemplate.execute("TRUNCATE TABLE archived_events");
        jdbcTemplate.execute("TRUNCATE TABLE employee_batches");
        jdbcTemplate.execute("TRUNCATE TABLE tracking_codes");
        jdbcTemplate.execute("TRUNCATE TABLE smart_filter_row_values");
        jdbcTemplate.execute("TRUNCATE TABLE badge_stagings");
    }

    public void dataInsert(String firstName, String lastName, String email, String fullName, 
                           String newFirstName, String newLastName, String newEmail, 
                           String newFullName, String employeeId) {
        String sql = "INSERT INTO anonymized_sensitive_values (old_first_name, old_last_name, old_email, " +
                     "old_full_name, new_first_name, new_last_name, new_email, new_full_name, employee_id) " +
                     "VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)";
        jdbcTemplate.update(sql, firstName, lastName, email, fullName, newFirstName, newLastName, 
                            newEmail, newFullName, employeeId);
    }

    public void anonymizeTablesByEmployee() {
        System.out.println("\nANONYMIZE EMPLOYEES SENSITIVE DATA AND STORE INTO TEMPORARY TABLE");
        System.out.println("\nEmployee Records Anonymization started at " + java.time.LocalTime.now());

        List<Map<String, Object>> employees = jdbcTemplate.queryForList("SELECT * FROM Employee");
        for (Map<String, Object> employee : employees) {
            executorService.submit(() -> {
                Map<String, String> anonymizedEmployee = employeeAttributes(employee.get("id").toString());
                String fullName = anonymizedEmployee.get("FirstName") + " " + anonymizedEmployee.get("LastName");

                dataInsert(employee.get("FirstName").toString(), employee.get("LastName").toString(), 
                           employee.get("Email").toString(), employee.get("full_name").toString(), 
                           anonymizedEmployee.get("FirstName"), anonymizedEmployee.get("LastName"), 
                           anonymizedEmployee.get("Email"), fullName, employee.get("id").toString());

                anonymizeTable("EmployeeSeed", anonymizedEmployee, "EmployeeId", employee.get("EmployeeId").toString());
                anonymizeTable("EmployeeChange", anonymizedEmployee, "employee_id", employee.get("id").toString());
                anonymizeTable("Employee", anonymizedEmployee, "id", employee.get("id").toString());
            });
        }

        executorService.shutdown();
        while (!executorService.isTerminated()) {
        }

        System.out.println("Completed Employee Records at " + java.time.LocalTime.now());

        anonymizeEmployeeImages();
        anonymizeEmployeeChangeTable();
        System.out.println("\nANONYMIZED EMPLOYEES SENSITIVE DATA AND STORED INTO TEMPORARY TABLE");
    }

    public void anonymizeEmployeeChangeTable() {
        anonymizeMissingRows("EmployeeChange", "updated_at", scriptUpdateTime);
        anonymizeMissingRows("EmployeeSeed", "updated_at", scriptUpdateTime);
    }

    public void anonymizeMissingRows(String tableName, String columnName, String time) {
        List<Map<String, Object>> rows = jdbcTemplate.queryForList("SELECT * FROM " + tableName + " WHERE ? > " + columnName, time);
        if (rows.size() > 0) {
            System.out.println(tableName + " of " + rows.size() + " Records Started At " + java.time.LocalTime.now());
            for (Map<String, Object> row : rows) {
                executorService.submit(() -> {
                    Map<String, String> anonymizedAttributes = employeeAttributes(row.get("id").toString());
                    jdbcTemplate.update("UPDATE " + tableName + " SET ? WHERE id = ?", anonymizedAttributes, row.get("id"));
                });
            }
            executorService.shutdown();
            while (!executorService.isTerminated()) {
            }
            System.out.println(tableName + " Completed At " + java.time.LocalTime.now());
        }
    }

    public void truncateTable(String tableName) {
        String sql = "TRUNCATE TABLE " + tableName;
        if (isTestEnvironment()) {
            sql = "DELETE FROM " + tableName;
        }
        jdbcTemplate.execute(sql);
    }

    public void anonymizeEmployeeImages() {
        System.out.println("Employee Images Started At " + java.time.LocalTime.now());
        jdbcTemplate.update("UPDATE Employee SET image_file_name = NULL");
        jdbcTemplate.update("UPDATE EmployeeSeed SET ImageData = NULL");
        jdbcTemplate.update("UPDATE User SET encrypted_password = '$2a$10$4SL5YurMASqu5qrkLO2BguGFpB.6G7SpzB3zsWAc3C5.orZAL2mXq', initials = 'OSS', reset_password_token = NULL, reset_password_sent_at = NULL WHERE email NOT LIKE '%@oss.com'");
        jdbcTemplate.update("UPDATE MailQueue SET toName = ?, toAddress = ?, subject = ?, content = ?", Faker.name(), Faker.email(), Faker.sentence(), Faker.paragraph());
        jdbcTemplate.update("UPDATE Request SET subject = ?", Faker.sentence());
        System.out.println("Employee Images Completed At " + java.time.LocalTime.now());
    }

    public Map<String, String> employeeAttributes(String id) {
        Map<String, String> attributes = new HashMap<>();
        attributes.put("FirstName", faker.name().firstName());
        attributes.put("LastName", faker.name().lastName());
        attributes.put("WorkPhone", faker.phoneNumber().phoneNumber());
        attributes.put("Extension", faker.phoneNumber().extension());
        attributes.put("Photo", null);
        attributes.put("Bio", faker.lorem().sentence());
        attributes.put("Email", id + faker.internet().emailAddress());
        /* for (int i = 0; i <= 34; i++) {
            attributes.put("Udf" + i, faker.lorem().sentence());
        } */
        attributes.put("updated_at", String.valueOf(System.currentTimeMillis()));
        return attributes;
    }
}

```

The provided code is written in JAVA and it seems to be part of a larger system that anonymizes sensitive data in a database. The code uses the JPA library, which is a part of Rails, but it can also be used standalone. Java Persistence API (JPA) tool, which means it allows you to interact with your database, like you would with SQL. In other words, it's a way to create, retrieve, update, and delete records from your database, without having to write raw SQL statements. In a Java Spring Boot application that utilizes MySQL DB, you can use `JdbcTemplate` to interact with your database with RAW SQL queries.

Let's break down each function:

1. `anonymize`: This is the main function that orchestrates the anonymization process. It first disables foreign key checks, truncates certain tables, and adds indexes to improve performance. It then anonymizes tables associated with employees, adds more indexes, and truncates the anonymized_sensitive_values table. It finally re-enables foreign key checks and rebuilds the PeopleReportCache.

2. `anonymize_table`: This function anonymizes a specific table. It takes in a model class, updated attributes, a field, and a value. It first counts the number of records where the field equals the value. If there are any, it updates all those records with the provided attributes.

3. `data_insert`: This function inserts a new record which will have new values of senstive data and exisitng values of employees for the future reference into the `anonymized_sensitive_values` table in the database. This method is part of a larger system that anonymizes sensitive data.
    The method takes nine parameters: first_name, last_name, email, full_name, new_first_name, new_last_name, new_email, new_full_name, and employee_id. These parameters represent the old and new (anonymized) values of certain attributes of an employee, as well as the employee's ID. The sql variable is assigned a string that represents an SQL INSERT statement. This statement inserts a new record into the anonymized_sensitive_values table with the old and new values of the employee's attributes and the employee's ID.
    Finally, the execute method of the `jdbcTemplate` object is called with the sanitized SQL statement as an argument. This method executes the SQL statement, inserting the new record into the anonymized_sensitive_values table.

4. `anonymize_missing_rows`: This function anonymizes rows that haven't been anonymized yet. It checks if the model instance has certain attributes and if it does, it updates those attributes.

5. `truncate_table`: This function truncates a specific table. It uses the sanitize_sql_array method to prevent SQL injection attacks. If the environment is test, it deletes the records instead of truncating the table.

6. `employee_attributes`: This function returns a hash of anonymized employee attributes. It uses the Faker gem to generate fake data.

The types you've mentioned like Time, `AnonymizeEmployeeAssociations`,   `Employee`, `EmployeeChange`, `EmployeeSeed`, `User`, `MoveEmployee`, `SeatOpenBooking`, `SeatOpenBookingAction`, `ConferencingResource`, `Request`, `RequestFeedbackSurvey`, Some of them are built-in JAVA classes like Time, some are from library like Faker, and others are likely defined in your application like Employee or User.

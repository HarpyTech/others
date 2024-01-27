# 
##

```JAVA
import com.github.javafaker.Faker;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;

import java.util.HashMap;
import java.util.Map;

public class AnonymizeEmployeeAssociations {

    @Autowired
    private NamedParameterJdbcTemplate namedParameterJdbcTemplate;
    private final Faker faker = new Faker();
    private final Map<String, String> employee = new HashMap<>();
    private final Map<String, String> additionalIndexes = new HashMap<>();
    private final long scriptUpdateTime;

    public AnonymizeEmployeeAssociations(long time) {
        this.scriptUpdateTime = time;

        this.additionalIndexes.put("canceledBy", "SeatOpenBooking");
        this.additionalIndexes.put("checkInByEmail", "SeatOpenBooking");
        this.additionalIndexes.put("checkOutBy", "SeatOpenBooking");
        this.additionalIndexes.put("empSittingEmail", "SeatOpenBooking");
        this.additionalIndexes.put("updatedBy", "SeatOpenBooking");
        this.additionalIndexes.put("performer_email", "seat_open_booking_actions");

        this.employee.put("first_name", faker.name().firstName());
        this.employee.put("last_name", faker.name().lastName());
        this.employee.put("email", faker.internet().emailAddress());
        this.employee.put("name", faker.name().fullName());
        this.employee.put("subject", faker.lorem().sentence());
        this.employee.put("content", faker.lorem().paragraph());
        this.employee.put("updated_at", String.valueOf(time));
    }

    public void anonymize() {
        updateUsers();
        System.out.println("\nWhite listed the OSS Emails so that we don't lose access, \nCompleted Users Records at " + new Date());
        updateMoveEmployee();
        System.out.println("Completed MoveEmployee Records at " + new Date());
        updateSeatOpenBooking();
        System.out.println("Completed SeatOpenBooking Records at " + new Date());
        updateSeatOpenBookingActions();
        System.out.println("Completed SeatOpenBookingAction Records at " + new Date());
        updateSafeguardResult();
        System.out.println("Completed Safeguard::Result Records at " + new Date());
        updateMailQueue();
        System.out.println("Completed MailQueue Records at " + new Date());
    }

    public void updateUsers() {
        String queryWithEmployee = "UPDATE users " +
                "INNER JOIN anonymized_sensitive_values ON email = anonymized_sensitive_values.old_email " +
                "SET email = anonymized_sensitive_values.new_email, " +
                "first_name = anonymized_sensitive_values.new_first_name, " +
                "last_name = anonymized_sensitive_values.new_last_name, " +
                "updated_at = :updated_at " +
                "WHERE (email NOT LIKE :ossEmailDomain)";

        Map<String, Object> params = new HashMap<>();
        params.put("updated_at", System.currentTimeMillis());
        params.put("ossEmailDomain", "%" + OSS_EMAIL_DOMAIN);

        namedParameterJdbcTemplate.update(queryWithEmployee, params);

        String queryWithoutEmployee = "UPDATE users " +
                "SET email = CONCAT(id, :email), " +
                "first_name = CONCAT(id, :first_name), " +
                "last_name = :last_name, " +
                "updated_at = :updated_at " +
                "WHERE (email NOT LIKE :ossEmailDomain AND updated_at < :updated_at)";

        params.put("email", "@anonymous.com");
        params.put("first_name", "Anonymous");
        params.put("last_name", "User");

        namedParameterJdbcTemplate.update(queryWithoutEmployee, params);
    }

    public void updateMoveEmployee() {
        String queryWithEmployee = "UPDATE MoveEmployee " +
                "INNER JOIN anonymized_sensitive_values ON MoveEmployee.employee_id = anonymized_sensitive_values.employee_id " +
                "SET empFirstName = anonymized_sensitive_values.new_first_name, " +
                "empLastName = anonymized_sensitive_values.new_last_name, " +
                "empEmail = anonymized_sensitive_values.new_email, " +
                "modifiedDate = :updated_at " +
                "WHERE (MoveEmployee.employee_id = anonymized_sensitive_values.employee_id)";

        String queryWithoutEmployee = "UPDATE MoveEmployee " +
                "SET empFirstName = CONCAT(idMoveEmployee, :first_name), " +
                "empLastName = :last_name, " +
                "empEmail = CONCAT(idMoveEmployee, :email), " +
                "modifiedDate = :updated_at " +
                "WHERE (modifiedDate < :updated_at)";

        Map<String, Object> params = new HashMap<>();
        params.put("first_name", "Anonymous");
        params.put("last_name", "User");
        params.put("email", "@anonymous.com");
        params.put("updated_at", System.currentTimeMillis());

        namedParameterJdbcTemplate.update(queryWithEmployee, params);
        namedParameterJdbcTemplate.update(queryWithoutEmployee, params);
    }

    public void updateSeatOpenBooking() {
        String queryWithEmployee = "UPDATE SeatOpenBooking " +
                "LEFT JOIN anonymized_sensitive_values requestor ON checkInByEmail = requestor.old_email " +
                "LEFT JOIN anonymized_sensitive_values occupant ON empSittingEmail = occupant.old_email " +
                "LEFT JOIN anonymized_sensitive_values terminator ON checkOutBy = terminator.old_full_name " +
                "LEFT JOIN anonymized_sensitive_values canceled_emp ON canceledBy = canceled_emp.old_full_name " +
                "LEFT JOIN anonymized_sensitive_values updated_emp ON updatedBy = updated_emp.old_full_name " +
                "SET checkOutBy = terminator.new_full_name, " +
                "empSittingFirstName = occupant.new_first_name, " +
                "empSittingLastName = occupant.new_last_name, " +
                "empSittingEmail = occupant.new_email, " +
                "checkInBy = requestor.new_full_name, " +
                "checkInByEmail = requestor.new_email, " +
                "canceledBy = canceled_emp.new_full_name, " +
                "updatedBy = updated_emp.new_full_name, " +
                "updated_at = :updated_at " +
                "WHERE (checkInByEmail = requestor.old_email OR canceledBy = canceled_emp.old_full_name " +
                "OR empSittingEmail = occupant.old_email OR updatedBy = updated_emp.old_full_name " +
                "OR checkOutBy = terminator.old_full_name)";

        String queryWithoutEmployee = "UPDATE SeatOpenBooking " +
                "SET checkOutBy = CONCAT(id, :name), " +
                "empSittingFirstName = CONCAT(id, :first_name), " +
                "empSittingLastName = :last_name, " +
                "empSittingEmail = CONCAT(id, :email), " +
                "checkInBy = CONCAT(id, :name), " +
                "checkInByEmail = CONCAT(id, :email), " +
                "canceledBy = :name, " +
                "updatedBy = :name, " +
                "updated_at = :updated_at " +
                "WHERE (updated_at < :updated_at)";

        Map<String, Object> params = new HashMap<>();
        params.put("name", "Anonymous User");
        params.put("first_name", "Anonymous");
        params.put("last_name", "User");
        params.put("email", "@anonymous.com");
        params.put("updated_at", System.currentTimeMillis());

        namedParameterJdbcTemplate.update(queryWithEmployee, params);
        namedParameterJdbcTemplate.update(queryWithoutEmployee, params);
    }

    public void updateSeatOpenBookingActions() {
        String queryWithEmployee = "UPDATE seat_open_booking_actions " +
                "INNER JOIN anonymized_sensitive_values ON performer_email = anonymized_sensitive_values.old_email " +
                "SET performed_by = anonymized_sensitive_values.new_full_name, " +
                "performer_email = anonymized_sensitive_values.new_email, " +
                "updated_at = :updated_at " +
                "WHERE (performer_email = anonymized_sensitive_values.old_email)";

        String queryWithoutEmployee = "UPDATE seat_open_booking_actions " +
                "SET performed_by = CONCAT(id, :name), " +
                "performer_email = CONCAT(id, :email), " +
                "updated_at = :updated_at " +
                "WHERE (updated_at < :updated_at)";

        Map<String, Object> params = new HashMap<>();
        params.put("name", "Anonymous User");
        params.put("email", "@anonymous.com");
        params.put("updated_at", System.currentTimeMillis());

        namedParameterJdbcTemplate.update(queryWithEmployee, params);
        namedParameterJdbcTemplate.update(queryWithoutEmployee, params);
    }

    public void updateSafeguardResult() {
        String queryWithEmployee = "UPDATE safeguard_results " +
                "INNER JOIN anonymized_sensitive_values ON email = anonymized_sensitive_values.old_email " +
                "SET email = anonymized_sensitive_values.new_email, " +
                "updated_at = :updated_at " +
                "WHERE (email = anonymized_sensitive_values.old_email)";

        String queryWithoutEmployee = "UPDATE safeguard_results " +
                "SET email = CONCAT(id, :email), " +
                "updated_at = :updated_at " +
                "WHERE (updated_at < :updated_at)";

        Map<String, Object> params = new HashMap<>();
        params.put("email", "@anonymous.com");
        params.put("updated_at", System.currentTimeMillis());

        namedParameterJdbcTemplate.update(queryWithEmployee, params);
        namedParameterJdbcTemplate.update(queryWithoutEmployee, params);
    }

    public void updateMailQueue() {
        String queryWithEmployee = "UPDATE mail_queues " +
                "INNER JOIN anonymized_sensitive_values ON fromAddress = anonymized_sensitive_values.old_email " +
                "SET fromName = anonymized_sensitive_values.new_full_name, " +
                "fromAddress = anonymized_sensitive_values.new_email, " +
                "deliveredDate = :updated_at " +
                "WHERE (fromAddress = anonymized_sensitive_values.old_email)";

        String queryWithoutEmployee = "UPDATE mail_queues " +
                "SET fromName = CONCAT(id, :name), " +
                "fromAddress = CONCAT(id, :email), " +
                "toName = CONCAT(id, :name), " +
                "toAddress = CONCAT(id, :email), " +
                "subject = :subject, " +
                "content = :content, " +
                "deliveredDate = :updated_at " +
                "WHERE (deliveredDate < :updated_at)";

        Map<String, Object> params = new HashMap<>();
        params.put("name", "Anonymous User");
        params.put("email", "@anonymous.com");
        params.put("subject", "Anonymized Subject");
        params.put("content", "Anonymized Content");
        params.put("updated_at", System.currentTimeMillis());

        namedParameterJdbcTemplate.update(queryWithEmployee, params);
        namedParameterJdbcTemplate.update(queryWithoutEmployee, params);
    }
}

```

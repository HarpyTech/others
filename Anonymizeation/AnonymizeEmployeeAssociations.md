# 
##

```JAVA
import com.github.javafaker.Faker;

import java.util.HashMap;
import java.util.Map;

public class AnonymizeEmployeeAssociations {

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

}

```

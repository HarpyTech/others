# Main  Schema

## Here is the schema for the given tables:
* `Employee` table with the following columns:
    id (integer),
    FirstName (string),
    LastName (string),
    Email (string),
    full_name (string),
    EmployeeId (string)
  
*  `users` table:
    email (string) is foreign key and refers to Employee.Email column, 
    first_name (string),
    last_name (string),
    updated_at (datetime)
   
* `MoveEmployee` table:
    employee_id (integer) is foreign key and refers to Employee.id column,
    empFirstName (string),
    empLastName (string),
    empEmail (string),
    modifiedDate (datetime)
  
* `SeatOpenBooking` table:
    checkInByEmail (string) is foreign key and refers to Employee.Email column,
    empSittingEmail (string) is foreign key and refers to Employee.Email column,
    checkOutBy (string)  is foreign key and refers to Employee.full_name column,
    canceledBy (string) is foreign key and refers to Employee.full_name column,
    updatedBy (string) is foreign key and refers to Employee.full_name column,
    empSittingFirstName (string),
    empSittingLastName (string),
    checkInBy (string),
    updated_at (datetime)
  
* `seat_open_booking_actions` table:
    performer_email (string) is foreign key and refers to Employee.Email column,
    performed_by (string),
    updated_at (datetime)
  
* `safeguard_results` table:
    email (string) is foreign key and refers to Employee.Email column,
    updated_at (datetime)
  
* `mail_queues` table:
    fromName (string)
    fromAddress (string) is foreign key and refers to Employee.Email column,
    toName (string),
    toAddress (string),
    subject (string),
    content (string),
    deliveredDate (datetime)
  
* `anonymized_sensitive_values` table:
    old_email (string),
    new_email (string),
    new_first_name (string),
    new_last_name (string),
    old_first_name (string),
    old_last_name (string),
    employee_id (integer),
    old_full_name (string),
    new_full_name (string)

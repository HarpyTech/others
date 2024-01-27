# Main  Schema

## Here is the schema for the given tables:
* `Employee` table with the following columns:
    id (integer)
    FirstName (string)
    LastName (string)
    Email (string)
    full_name (string)
    EmployeeId (string)
  
*  `users` table:
    email (string) 
    first_name (string)
    last_name (string)
    updated_at (datetime)
   
* `MoveEmployee` table:
    employee_id (integer)
    empFirstName (string)
    empLastName (string)
    empEmail (string)
    modifiedDate (datetime)
  
* `SeatOpenBooking` table:
    checkInByEmail (string)
    empSittingEmail (string)
    checkOutBy (string)
    canceledBy (string)
    updatedBy (string)
    empSittingFirstName (string)
    empSittingLastName (string)
    checkInBy (string)
    updated_at (datetime)
  
* `seat_open_booking_actions` table:
    performer_email (string)
    performed_by (string)
    updated_at (datetime)
  
* `safeguard_results` table:
    email (string)
    updated_at (datetime)
  
* `mail_queues` table:
    fromName (string)
    fromAddress (string)
    toName (string)
    toAddress (string)
    subject (string)
    content (string)
    deliveredDate (datetime)
  
* `anonymized_sensitive_values` table:
    old_email (string)
    new_email (string)
    new_first_name (string)
    new_last_name (string)
    old_first_name (string)
    old_last_name (string)
    employee_id (integer)
    old_full_name (string)
    new_full_name (string)

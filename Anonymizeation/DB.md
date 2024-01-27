# Main  Schema

## Here is the schema for the given tables:
* `Employee` table with the following columns:
    id (integer),
    FirstName (string),
    LastName (string),
    Email (string),
    full_name (string),
    EmployeeId (string),
    Bio (string),
    WorkPhone (string),
    Extension (string)
  
*  `users` table:
    email (string) is foreign key and refers to Employee.Email column, 
    first_name (string),
    last_name (string),
    updated_at (datetime)
   
  
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


![Sequence Diagram](https://github.com/HarpyTech/others/assets/77878864/9cf4a523-504c-4177-b884-86c07356475d)
![Sequence Diagram D](https://github.com/HarpyTech/others/assets/77878864/976cdf3c-126e-4efb-b644-32ccec667ce0)
![Screenshot 2024-01-27 144559](https://github.com/HarpyTech/others/assets/77878864/d64626d7-9243-4d73-9669-00438146f203)
![Screenshot 2024-01-27 144545](https://github.com/HarpyTech/others/assets/77878864/3380a552-7172-45da-972d-09058460d165)
![Screenshot 2024-01-27 143645](https://github.com/HarpyTech/others/assets/77878864/8189c0a7-2834-4d0d-b6e0-01dbf32114a2)
![Screenshot 2024-01-27 143619](https://github.com/HarpyTech/others/assets/77878864/a308ca0b-422f-4225-a80f-c78a4a705945)
![Screenshot 2024-01-27 142528](https://github.com/HarpyTech/others/assets/77878864/058bd34c-ea6c-4127-85a8-77c5e51d9752)
![Screenshot 2024-01-27 142449](https://github.com/HarpyTech/others/assets/77878864/fa185dfc-29d1-4cd2-be15-0e0b0f2c314e)
![Screenshot 2024-01-27 141906](https://github.com/HarpyTech/others/assets/77878864/11057ed1-8f51-4132-9d1c-a2591db2410c)
![Screenshot 2024-01-27 141851](https://github.com/HarpyTech/others/assets/77878864/2845d905-1bd4-4c16-a044-45a29c4f6ee3)
![Screenshot 2024-01-27 141636](https://github.com/HarpyTech/others/assets/77878864/b5e71210-7e0e-47c1-a06c-ee4b6fc142cc)



When preparing documentation for a case study that involved identifying an issue, implementing a solution, and addressing the time-consuming task, consider including the following elements:

Introduction:

Provide an overview of the case study, including the problem statement and its significance.
Summarize the objectives of the study and outline the structure of the documentation.
Background:

Describe the context and background information relevant to the case study.
Explain the specific task or process that was identified as time-consuming.
Problem Analysis:

Outline the process used to identify and analyze the issue.
Document the factors contributing to the time-consuming task, such as inefficiencies, bottlenecks, or resource constraints.
Include any data or evidence gathered during the analysis phase.
Solution Implementation:

Describe the solution that was implemented to address the identified issue.
Explain the rationale behind the chosen solution and its expected benefits.
Provide details about the steps taken to implement the solution, including any changes made to processes, tools, or resources.
Results and Outcomes:

Present the outcomes and results achieved after implementing the solution.
Include quantitative and qualitative data, such as time savings, productivity improvements, or user feedback.
Discuss any challenges encountered during the implementation and how they were overcome.
Lessons Learned:

Reflect on the lessons learned from the case study.
Identify any insights or best practices that emerged during the process.
Discuss recommendations for future improvements or similar situations.
Conclusion:

Summarize the main findings, outcomes, and impacts of the case study.
Emphasize the significance of the implemented solution in addressing the time-consuming task.
Mention any further steps or actions recommended based on the case study.
Appendices:

Include any supporting documents, charts, diagrams, or additional data that may be relevant to the case study.
Provide references or citations for any external sources or studies referenced.
Remember to maintain a clear and coherent structure throughout the documentation, using headings, subheadings, and bullet points where appropriate. Use a writing style that is concise, yet comprehensive, ensuring that the document is easily understandable to others who may read it.



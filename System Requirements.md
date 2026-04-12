## Choice Development Model
Model: Agile (Scrum)

UniGate is developed using the Agile (Scrum) methodology. The project is split into short, focused sprints in which each is delivering a working feature such as the program catalogue, authentication, application form, document upload, or the admin dashboard. Agile was chosen because the team has five members each owning a distinct module, so parallel development and continuous integration are essential. Regular sprint reviews allow us to catch integration issues early and adapt the design based on feedback before the deadline.

## User Requirements

a. Stakeholders
Guest (Unregistered Visitor)

A guest is any anonymous visitor who opens UniGate without an account. Their interest is to freely explore and compare available bachelor's and master's programmes before deciding whether to register and apply.

Student (Registered Applicant)

A student is a registered user who has created an account on UniGate. Their interest is to complete the full online application process, filling out the application form, uploading required documents, and submitting their application to a chosen program, without needing to visit the university in person.

University Staff

University staff are representatives of a university who are responsible for keeping the platform's content accurate and up to date. Their interest is to add, edit, and remove program listings for their institution, and to review and update the status of submitted applications.

Admin (System Administrator)

The admin is the technical role responsible for keeping the UniGate platform running smoothly. Their interest is to manage user accounts, assign roles, monitor system activity, resolve technical issues, and ensure the platform is secure, stable, and available at all times.

b. User Stories
Guest
- As a guest, I want to browse all available bachelor's and master's programs without creating an account, so that I can explore my options before committing to registering.
- As a guest, I want to filter programs by degree type, language of instruction, and duration, so that I can quickly narrow down programs that match my preferences.
- As a guest, I want to view the full details of a program including entry requirements, tuition fees, and duration, so that I can compare programs and make an informed decision.
- As a guest, I want to be prompted to register when I click Apply, so that I understand I need an account before I can submit an application.
- As a guest, I want the website to work on my mobile phone, so that I can browse programmes while commuting or away from a computer.

Student
- As a student, I want to register with my email and a password, so that I can create a personal account and access the full application process.
- As a student, I want to log in and out securely, so that my personal data and application details remain private.
- As a student, I want to land on a personal dashboard after logging in, where I can see all my submitted applications, their current statuses, and my uploaded documents.
- As a student, I want to fill out a step-by-step application form covering personal details, academic history, program choice, and document upload, so that I can submit a complete application in one continuous flow.
- As a student, I want real-time validation on the application form, so that I can fix mistakes before submitting and avoid having my application rejected due to errors.
- As a student, I want the system to check that all required documents are uploaded before I can submit, so that I do not send an incomplete application by mistake.
- As a student, I want to be able to apply to multiple universities, with one program per university, so that I can keep my options open without submitting duplicate applications to the same institution.

University Staff
- As a university staff member, I want to log in through a dedicated staff login page, so that my access to program management is separate from the student-facing area.
- As a university staff member, I want to add new program listings with all required details, so that prospective students can find and apply to our program on UniGate.
- As a university staff member, I want to edit existing program details such as entry requirements, tuition fees, and duration, so that the information students see is always accurate and up to date.
- As a university staff member, I want to remove a program that is no longer offered, so that students are not able to apply to programs that are closed or discontinued.
- As a university staff member, I want to review submitted applications for my university's programs and update each application's status to accepted or rejected, so that the outcome of each review is recorded in the system.

Admin (System Administrator)
- As an admin, I want to log in through a protected admin panel, so that system management tools are not accessible to students or university staff.
- As an admin, I want to create, edit, and deactivate user accounts for students and university staff, so that only authorised users have access to the platform.
- As an admin, I want to assign and change user roles such as student, university staff, or admin, so that each user only has access to the features relevant to their role.
- As an admin, I want to monitor system activity and view error logs, so that I can detect and resolve technical issues before they affect users.
- As an admin, I want to manage file storage and uploaded documents, so that the server does not run out of space and all stored files remain organised and accessible.

##  Functional Requirements
a. Description
- Any  can brguestowse the program catalogue without an account.
- Each program displays degree type, duration, language of instruction, entry requirements, and tuition fees.
- Students can filter programs by degree type, language of instruction, and duration.
- New students can register with a unique email and a password of at least 8 characters including one number.
- Registration is blocked if the email address is already in use.
- Registered students can log in with email and password; a signed JWT token is issued upon successful authentication.
- Unauthenticated students who attempt to access the application form or document upload are redirected to the login page.
- The application form is split into four clearly labelled steps: personal details, academic history, program selection, and document upload, all completed in a single continuous flow.
- All required fields are validated in real time with inline error messages for missing or incorrectly formatted inputs.
- Each submitted application is saved linked to the logged-in student's account and the selected program.
- Students can upload documents in PDF, JPG, and PNG formats only; any other file type is rejected with an error message.
- Uploaded file paths are stored in the documents table and associated with the correct application.
- University staff can log in through a dedicated staff login page and access only the program and application management area for their linked university.
- University staff can add a new program with all required fields; it appears immediately in the public catalogue.
- University staff can edit existing program details and delete programs that are no longer offered.
- University staff can view all submitted applications for their university's programs, open individual applications, and update each application's status to accepted or rejected.
- The admin panel is accessible only to users with the admin role and returns HTTP 403 for all other users.
- The admin can create, edit, and deactivate user accounts and assign or change user roles.
- The admin can view system logs and monitor uploaded file storage.
- All pages are served responsively and remain usable on screens with a minimum width of 320 px.
- After logging in, a student is directed to a personal dashboard displaying all submitted applications, the current status of each (pending, accepted, or rejected), and the documents attached to each application.
- UniGate supports multiple Albanian universities, each stored as a distinct record in the system. Every program is linked to exactly one university. A student may apply to more than one university but is restricted to one program per university at any given time.
- Before submission, the system checks that all required documents have been uploaded. If any are missing, submission is blocked and the student is shown a message identifying exactly which documents are still needed.

b. Acceptance Criteria

1. New students can register with a unique email and a password of at least 8 characters including one number.

Acceptance Criteria:
- The registration form accepts an email address and a password field.
- The system checks that the email has not been previously used by another account.
- The system checks that the password is at least 8 characters long and contains at least one number.
- If all conditions are met, the account is created and the student is redirected to the login page.
- If any condition fails, an inline error message is displayed next to the relevant field.

2. Registered students can log in with email and password; a signed JWT token is issued upon successful authentication.

Acceptance Criteria:
- The login form accepts an email and a password.
- The system verifies the submitted credentials against the bcrypt-hashed value stored in the database.
- Upon successful verification, a signed JWT token is generated and stored in an HTTP-only cookie.
- The student is redirected to their role-appropriate destination after login.
- If credentials are incorrect, an error message is returned and no token is issued.

3. The application form is split into four clearly labelled steps: personal details, academic history, program selection, and document upload, all completed in a single continuous flow.

Acceptance Criteria:
- The form renders four distinct steps, each with a visible label identifying its purpose.
- The student cannot skip to a later step without completing the current one.
- A progress indicator reflects which step the student is currently on.
- All data entered in previous steps is retained when moving forward or backward through the form.
- The entire form is submitted as one complete record upon final confirmation.

4. Students can upload documents in PDF, JPG, and PNG formats only; any other file type is rejected with an error message.

Acceptance Criteria:
- The upload input accepts file selection from the student's device.
- The system checks both the MIME type and file extension of each uploaded file on the server side.
- Files in formats other than PDF, JPG, and PNG are rejected before being saved to the server.
- An inline error message is displayed to the student identifying the rejected file.
- Accepted files are displayed in a list showing the file name and type before final submission.

5. Before submission, the system checks that all required documents have been uploaded. If any are missing, submission is blocked and the student is shown a message identifying exactly which documents are still needed.

Acceptance Criteria:
- The system identifies which document types are marked as required for the selected program.
- At the point of submission, the system cross-checks uploaded documents against the required list.
- If one or more required documents are missing, the submit button does not process the form.
- The student is shown a message that lists the specific document types that have not yet been uploaded.
- Submission proceeds only when all required documents are present and all other form fields are valid.


## Non-Functional Requirements
a. Description
Usability
- The main navigation bar must expose all key sections of the platform from every page, so that a user never needs more than two clicks to reach any primary feature from any location on the site.
- All form error messages must be written in plain language and displayed adjacent to the relevant field.

Responsiveness
- The interface must be fully functional on screen widths from 320 px (mobile) to 1920 px (desktop).

Security
- All passwords must be hashed with bcrypt at a cost factor of 10 or higher before being stored.
- File uploads must be restricted to PDF, JPG, and PNG by both MIME type and file extension on the server side.
- Each role, student, university staff, and admin  must only be able to access the routes and data that belong to that role.

Reliability
- The platform must handle at least 50 concurrent users without errors or timeouts.

Maintainability
- All backend routes, database models, and frontend components must include inline comments describing their purpose.

Availability
- The platform must maintain a minimum uptime of 99.5% measured on a monthly basis, excluding scheduled maintenance windows.
- Scheduled maintenance must be communicated to students, university staff at least 24 hours in advance via an on-site banner.

Scalability
- The system architecture must be designed to scale to support 500 concurrent students without requiring structural code changes.

Data Privacy
- All personally identifiable information (PII) collected from students must be handled in accordance with GDPR principles, including a clear data retention and deletion policy.

Session Management
- Authenticated student`s sessions are governed by a JWT with a 24-hour expiry. Upon expiry the user must re-authenticate to obtain a new token.

Error Handling
- All unhandled server-side errors must return a generic, user-friendly error page with no stack traces or internal system details exposed.

Audit Logging
- Every admin action, including application approvals, rejections, and user account deletions  must be written to an audit log recording the timestamp, admin user ID, and the affected record ID.

Browser Compatibility
- The platform must function correctly on the two most recent major versions of Chrome, Firefox, Safari, and Microsoft Edge.

Password Storage
- Passwords are hashed using bcrypt with a minimum cost factor of 10 before being written to the database.
- Passwords are never recorded in server logs, error messages, or API responses under any circumstance.

Input Validation
- All text inputs submitted via API are stripped of leading and trailing whitespace before processing.
- Inputs containing SQL special characters are handled exclusively through Sequelize parameterised queries and do not alter database behaviour.
- Inputs containing HTML or script tags are escaped or rejected server-side and never rendered as executable markup.
- Fields with defined maximum lengths reject input that exceeds those limits and return a structured error response identifying the offending field.

Test Coverage
- The test suite must achieve a minimum of 70% line coverage across all backend modules.
- All critical route handlers — authentication, application submission, document upload, and role-based access control — must have at least one dedicated unit test covering both a success path and an expected failure path.
- The test suite must run to completion with zero failures on a clean environment using only the project's listed dependencies.

API Response Consistency
- All API endpoints must return responses in a consistent JSON structure containing a status field, a data field for successful responses, and a message field for errors.
- All error responses must include an appropriate HTTP status code: 400 for validation errors, 401 for unauthenticated requests, 403 for unauthorised access, and 500 for unhandled server errors.

b. Acceptance Criteria

1. All passwords must be hashed with bcrypt at a cost factor of 10 or higher before being stored.

Acceptance Criteria:
- When a student registers or changes their password, the system applies bcrypt hashing before writing to the database.
- The cost factor used is set to a minimum of 10 in the bcrypt configuration.
- A direct inspection of any user record in the database reveals no plaintext or reversibly-encoded password.
- Login verification uses bcrypt's comparison function rather than string equality.
- Passwords are never recorded in server logs, error messages, or API responses under any circumstance.

2. The platform must handle at least 50 concurrent users without errors or timeouts.

Acceptance Criteria:
- A load test simulating 50 simultaneous users performing typical actions is run against the platform.
- No request during the test returns a server error (HTTP 5xx) or times out.
- Response times remain within acceptable limits throughout the duration of the test.
- The test covers core actions including browsing the catalogue, logging in, and submitting an application.
- Results are recorded and reviewed to confirm the platform meets the 50-user threshold.

3. The platform must maintain a minimum uptime of 99.5% measured on a monthly basis, excluding scheduled maintenance windows.

Acceptance Criteria:
- Uptime is monitored continuously using a system that records availability at regular intervals throughout each month.
- At the end of each month, total downtime excluding scheduled maintenance is calculated and must not exceed 0.5% of the monthly total time.
- Any unplanned outage is logged with a start time, end time, and cause.
- Scheduled maintenance windows are excluded from the uptime calculation provided they were communicated at least 24 hours in advance via an on-site banner.
- A monthly uptime report is generated and accessible to the admin for review.

4. All unhandled server-side errors must return a generic, user-friendly error page with no stack traces or internal system details exposed.

Acceptance Criteria:
- A test request that triggers an unhandled server error receives a response with an appropriate HTTP status code (500).
- The response body contains a generic, plain-language message that does not include a stack trace, file path, or internal variable name.
- The error page matches the visual style of the rest of the platform.
- The actual error details are written to a server-side log accessible only to the admin, not to the end user.
- This behaviour is consistent across all routes and for all user roles.

5. All API endpoints must return responses in a consistent JSON structure containing a status field, a data field for successful responses, and a message field for errors.

Acceptance Criteria:
- Every API endpoint, when called successfully, returns a JSON object that includes a status field and a data field containing the relevant response payload.
- Every API endpoint, when an error occurs, returns a JSON object that includes a status field and a message field describing the error in plain language.
- The HTTP status code of each response matches the nature of the outcome: 200 for success, 400 for validation errors, 401 for unauthenticated requests, 403 for unauthorised access, and 500 for unhandled server errors.
- A review of all route handlers confirms no endpoint returns a response that deviates from this structure.
- Automated tests covering both success and error paths for each critical endpoint verify the structure is enforced consistently.

## Application Specifications

a. Architecture

UniGate follows a three-tier architecture consisting of a frontend, backend, and database. The frontend is a React single-page application that sends requests to the backend via REST API calls. The backend is a Node.js/Express server that handles business logic, authentication, file uploads, and role-based access control for four user roles: guest, student, university staff, and admin. The database is a MySQL relational database accessed through Sequelize ORM for efficient data management. Uploaded files are stored in a dedicated directory on the server and referenced in the database by file path. Staff and admin sections are protected areas of the frontend, accessible only to users with the appropriate role.

b. Database Model

The database consists of five tables:
Universities stores the institution's id, name, and contact reference, linked to its associated staff accounts and programs.
Users stores the id, name, email, hashed password, and role (student, staff, or admin). Staff accounts also include a university_id foreign key linking them to their institution.
Programs stores the id, title, degree type, duration, language of instruction, entry requirements, tuition fee, and a university_id foreign key referencing the offering institution.
Applications stores the id, user_id and programme_id as foreign keys, along with personal details, academic history fields, and application status (pending, accepted, or rejected).
Documents stores the id, application_id as a foreign key, file path, and file type for uploaded supporting documents.

All foreign keys enforce referential integrity with CASCADE on delete, ensuring that related records are automatically removed when a parent record is deleted.


c. Technologies Used

JavaScript (React.js) is used for frontend development to build a component-based and reusable user interface across the application.
TailwindCSS is used for styling due to its utility-first approach, enabling a fully responsive layout with minimal custom code.
JavaScript (Node.js with Express) is used for backend development due to its lightweight nature and suitability for building REST APIs.
MySQL with Sequelize ORM is used as the database for its relational structure and efficient data management, with Sequelize simplifying queries and schema migrations.
JWT and bcrypt are used for authentication due to their reliability in managing secure sessions and password hashing.
Multer is used for file upload handling due to its seamless integration with Express and built-in file validation.
Git and GitHub are used for version control to support collaborative development across the team.

d. User Interface Design

The platform consists of nine main views:
Home and Catalogue Page displays a searchable and filterable grid of all available programmes, accessible to both guests and students.
Program Detail Page shows full information for a single program, including an Apply button that redirects unauthenticated users to register.
Register and Login Pages contain simple centered forms with inline validation feedback.
Student Dashboard serves as the landing page for authenticated students, displaying all submitted applications with their current statuses and attached documents.
Application Form Page is a stepped form with a progress indicator guiding the student through personal details, academic history, program selection, and document upload.
University Staff Dashboard has two sections:Programs for managing listings, and Applications for reviewing submissions and updating statuses.
Admin Panel provides user account management, role assignment, system log viewing, and file storage monitoring.
All pages share a top navigation bar with the UniGate logo and a login/logout button. 

e. Security Measures

Passwords are hashed using bcrypt before storage, ensuring plaintext passwords are never saved in the database.
Authentication is managed through JWT tokens with a 24-hour expiry, stored in HTTP-only cookies to prevent client-side access.
Role-based access control is enforced on the server, ensuring students, staff, and admins can only access their permitted routes.
File uploads are validated by MIME type and file extension, rejecting any files that are not PDF, JPG, or PNG before they are saved.
All database queries use Sequelize parameterised queries to protect against SQL injection attacks.
HTTPS is enforced in production to encrypt all data transmitted between the client and the server.






 


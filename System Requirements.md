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
•	As a guest, I want to browse all available bachelor's and master's programs without creating an account, so that I can explore my options before committing to registering.
•	As a guest, I want to filter programs by degree type, language of instruction, and duration, so that I can quickly narrow down programs that match my preferences.
•	As a guest, I want to view the full details of a program including entry requirements, tuition fees, and duration, so that I can compare programs and make an informed decision.
•	As a guest, I want to be prompted to register when I click Apply, so that I understand I need an account before I can submit an application.
•	As a guest, I want the website to work on my mobile phone, so that I can browse programmes while commuting or away from a computer.

Student
•	As a student, I want to register with my email and a password, so that I can create a personal account and access the full application process.
•	As a student, I want to log in and out securely, so that my personal data and application details remain private.
•	As a student, I want to land on a personal dashboard after logging in, where I can see all my submitted applications, their current statuses, and my uploaded documents.
•	As a student, I want to fill out a step-by-step application form covering personal details, academic history, program choice, and document upload, so that I can submit a complete application in one continuous flow.
•	As a student, I want real-time validation on the application form, so that I can fix mistakes before submitting and avoid having my application rejected due to errors.
•	As a student, I want the system to check that all required documents are uploaded before I can submit, so that I do not send an incomplete application by mistake.
•	As a student, I want to be able to apply to multiple universities, with one program per university, so that I can keep my options open without submitting duplicate applications to the same institution.

University Staff
•	As a university staff member, I want to log in through a dedicated staff login page, so that my access to program management is separate from the student-facing area.
•	As a university staff member, I want to add new program listings with all required details, so that prospective students can find and apply to our program on UniGate.
•	As a university staff member, I want to edit existing program details such as entry requirements, tuition fees, and duration, so that the information students see is always accurate and up to date.
•	As a university staff member, I want to remove a program that is no longer offered, so that students are not able to apply to programs that are closed or discontinued.
•	As a university staff member, I want to review submitted applications for my university's programs and update each application's status to accepted or rejected, so that the outcome of each review is recorded in the system.

Admin (System Administrator)
•	As an admin, I want to log in through a protected admin panel, so that system management tools are not accessible to students or university staff.
•	As an admin, I want to create, edit, and deactivate user accounts for students and university staff, so that only authorised users have access to the platform.
•	As an admin, I want to assign and change user roles such as student, university staff, or admin, so that each user only has access to the features relevant to their role.
•	As an admin, I want to monitor system activity and view error logs, so that I can detect and resolve technical issues before they affect users.
•	As an admin, I want to manage file storage and uploaded documents, so that the server does not run out of space and all stored files remain organised and accessible.

##  Functional Requirements
a. Description
•	Any  can brguestowse the program catalogue without an account.
•	Each program displays degree type, duration, language of instruction, entry requirements, and tuition fees.
•	Students can filter programs by degree type, language of instruction, and duration.
•	New students can register with a unique email and a password of at least 8 characters including one number.
•	Registration is blocked if the email address is already in use.
•	Registered students can log in with email and password; a signed JWT token is issued upon successful authentication.
•	Unauthenticated students who attempt to access the application form or document upload are redirected to the login page.
•	The application form is split into four clearly labelled steps: personal details, academic history, program selection, and document upload, all completed in a single continuous flow.
•	All required fields are validated in real time with inline error messages for missing or incorrectly formatted inputs.
•	Each submitted application is saved linked to the logged-in student's account and the selected program.
•	Students can upload documents in PDF, JPG, and PNG formats only; any other file type is rejected with an error message.
•	Uploaded file paths are stored in the documents table and associated with the correct application.
•	University staff can log in through a dedicated staff login page and access only the program and application management area for their linked university.
•	University staff can add a new program with all required fields; it appears immediately in the public catalogue.
•	University staff can edit existing program details and delete programs that are no longer offered.
•	University staff can view all submitted applications for their university's programs, open individual applications, and update each application's status to accepted or rejected.
•	The admin panel is accessible only to users with the admin role and returns HTTP 403 for all other users.
•	The admin can create, edit, and deactivate user accounts and assign or change user roles.
•	The admin can view system logs and monitor uploaded file storage.
•	All pages are served responsively and remain usable on screens with a minimum width of 320 px.
•	After logging in, a student is directed to a personal dashboard displaying all submitted applications, the current status of each (pending, accepted, or rejected), and the documents attached to each application.
•	UniGate supports multiple Albanian universities, each stored as a distinct record in the system. Every program is linked to exactly one university. A student may apply to more than one university but is restricted to one program per university at any given time.
•	Before submission, the system checks that all required documents have been uploaded. If any are missing, submission is blocked and the student is shown a message identifying exactly which documents are still needed.

b. Acceptance Criteria
1. User Authentication Acceptance Criteria
•	A new student registers with a unique email and a password of at least 8 characters containing at least one number.
•	If the email is already in use, registration is blocked and an inline error is shown adjacent to the email field.
•	Upon successful registration the user is redirected to the login page.
•	The system verifies submitted credentials against the bcrypt-hashed value stored in the database.
•	Upon successful login a signed JWT is issued and stored in an HTTP-only cookie.
•	The student is redirected to their role-appropriate destination: students to their dashboard, staff to the staff dashboard, and admins to the admin panel.
•	Incorrect credentials return an error message and no token is issued.
•	The JWT expires after 24 hours, requiring the user to log in again.

2. Program Catalogue Acceptance Criteria
•	Any visitor, guest or authenticated student, can access the catalogue without restriction.
•	The catalogue displays all programs in a searchable, filterable grid.
•	Filtering by degree type, language of instruction, or duration updates the list without a full page reload.
•	Selecting a program opens a detail page showing entry requirements, tuition fees, duration, and language of instruction.
•	A guest clicking Apply is redirected to the registration page with a message that an account is required.
•	A logged-in student clicking Apply is taken directly into the application form for that program.

3. Application Form Acceptance Criteria
•	Only authenticated students can access the application form; unauthenticated users are redirected to the login page.
•	The form is divided into clearly labelled steps: personal details, academic history, programme selection, and document upload.
•	A student cannot proceed to the next step if any required field is incomplete or incorrectly formatted.
•	Each validation error is displayed inline and in plain language describing what is wrong.
•	The system enforces the one-programme-per-university rule and blocks selection if the student has already applied to that university.
•	The final step presents a full summary of all entered information before submission.
•	If any required document is missing, submission is blocked and the student is informed of exactly which documents are needed.
•	Upon successful submission the application is saved and linked to the student's account and selected programme.

4. Document Upload Acceptance Criteria
•	Document upload is presented as a dedicated step within the application form flow.
•	Students can upload files in PDF, JPG, and PNG formats only.
•	Attempting to upload a file of any other format returns an inline error and the file is not saved.
•	Each uploaded file is displayed in a list showing the file name, type, and a removal option before submission.
•	Upon successful upload each file is saved to the server and its path is recorded in the documents table linked to the correct application.
•	The system identifies which document types are required and marks each as pending or uploaded so the student can track what is still missing.

5. Application Status & Student Dashboard Acceptance Criteria
•	A logged-in student lands on their dashboard immediately after authentication.
•	The dashboard displays all submitted applications, each showing the programme name, university, submission date, and current status.
•	Status values are limited to pending, accepted, and rejected, and reflect the most recent update made by university staff.
•	A student cannot edit or withdraw an application after it has been submitted.
•	Selecting an application from the dashboard opens a read-only detail view showing all submitted fields and uploaded documents.

## Non-Functional Requirements
a. Description
Usability
•	The main navigation bar must expose all key sections of the platform from every page, so that a user never needs more than two clicks to reach any primary feature from any location on the site.
•	All form error messages must be written in plain language and displayed adjacent to the relevant field.

Responsiveness
•	The interface must be fully functional on screen widths from 320 px (mobile) to 1920 px (desktop).

Security
•	All passwords must be hashed with bcrypt at a cost factor of 10 or higher before being stored.
•	File uploads must be restricted to PDF, JPG, and PNG by both MIME type and file extension on the server side.
•	Each role, student, university staff, and admin  must only be able to access the routes and data that belong to that role.

Reliability
•	The platform must handle at least 50 concurrent users without errors or timeouts.

Maintainability
•	All backend routes, database models, and frontend components must include inline comments describing their purpose.

Availability
•	The platform must maintain a minimum uptime of 99.5% measured on a monthly basis, excluding scheduled maintenance windows.
•	Scheduled maintenance must be communicated to students, university staff at least 24 hours in advance via an on-site banner.

Scalability
•	The system architecture must be designed to scale to support 500 concurrent students without requiring structural code changes.

Data Privacy
•	All personally identifiable information (PII) collected from students must be handled in accordance with GDPR principles, including a clear data retention and deletion policy.

Session Management
•	Authenticated student`s sessions are governed by a JWT with a 24-hour expiry. Upon expiry the user must re-authenticate to obtain a new token.

Error Handling
•	All unhandled server-side errors must return a generic, user-friendly error page with no stack traces or internal system details exposed.

Audit Logging
•	Every admin action, including application approvals, rejections, and user account deletions  must be written to an audit log recording the timestamp, admin user ID, and the affected record ID.

Browser Compatibility
•	The platform must function correctly on the two most recent major versions of Chrome, Firefox, Safari, and Microsoft Edge.

Password Storage
•	Passwords are hashed using bcrypt with a minimum cost factor of 10 before being written to the database.
•	Passwords are never recorded in server logs, error messages, or API responses under any circumstance.

Input Validation
•	All text inputs submitted via API are stripped of leading and trailing whitespace before processing.
•	Inputs containing SQL special characters are handled exclusively through Sequelize parameterised queries and do not alter database behaviour.
•	Inputs containing HTML or script tags are escaped or rejected server-side and never rendered as executable markup.
•	Fields with defined maximum lengths reject input that exceeds those limits and return a structured error response identifying the offending field.

Test Coverage
•	The test suite must achieve a minimum of 70% line coverage across all backend modules.
•	All critical route handlers — authentication, application submission, document upload, and role-based access control — must have at least one dedicated unit test covering both a success path and an expected failure path.
•	The test suite must run to completion with zero failures on a clean environment using only the project's listed dependencies.

API Response Consistency
•	All API endpoints must return responses in a consistent JSON structure containing a status field, a data field for successful responses, and a message field for errors.
•	All error responses must include an appropriate HTTP status code: 400 for validation errors, 401 for unauthenticated requests, 403 for unauthorised access, and 500 for unhandled server errors.

b. Acceptance Criteria
1. Password Storage Acceptance Criteria
•	Passwords are hashed using bcrypt with a minimum cost factor of 10 before being written to the database.
•	A database inspection of any user record reveals no plaintext or reversibly-encoded password.
•	Login verification uses bcrypt's comparison function, not string equality.
•	Passwords are never logged in server logs, error messages, or API responses.
•	If a registration or password-change request fails, no partial plaintext data is persisted.

2. Input Validation Acceptance Criteria
•	All text fields submitted via API are stripped of leading and trailing whitespace before processing.
•	Inputs containing SQL special characters are safely handled via Sequelize parameterised queries and do not alter database behaviour.
•	Inputs containing HTML or script tags are escaped or rejected server-side and never rendered as executable markup in the browser.
•	Fields with defined maximum lengths reject input exceeding those limits with a 400 response identifying the offending field.
•	Validation errors return a structured JSON response specifying which field failed and why.

3. Responsiveness Acceptance Criteria
•	All interactive elements including buttons, forms, and file upload controls remain fully tappable and functional on a mobile touchscreen.
•	The application layout adjusts correctly at breakpoints for mobile (320 px–767 px), tablet (768 px–1024 px), and desktop (1025 px and above).
•	No horizontal scrolling occurs on any page at any of the defined breakpoints.
•	Font sizes remain legible at a minimum of 14 px body text across all screen sizes.
•	The application form and document upload step are both fully operable on a mobile device without zooming.

4. Session Management Acceptance Criteria
•	After 24 hours, a test session token is automatically invalidated.
•	Any subsequent request to a protected route using an expired token returns HTTP 401 and redirects the user to the login page.
•	The original protected page is not rendered until the user successfully re-authenticates.
•	JWT tokens are stored in HTTP-only cookies and are not accessible via client-side JavaScript.
•	Logging out invalidates the session on the client by clearing the HTTP-only cookie.

5. Security Acceptance Criteria
•	Inspecting the database confirms no plaintext passwords are stored in any user record.
•	Attempting to upload a file that is not PDF, JPG, or PNG is rejected server-side before the file is written to disk, and an error message is returned.
•	A student session token cannot access the staff or admin routes; a staff token cannot access the admin panel. Both return HTTP 403.
•	All API endpoints validate the JWT token on every request; a missing or tampered token returns HTTP 401.
•	No stack traces, file paths, or internal variable names are present in any error response body.

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






 


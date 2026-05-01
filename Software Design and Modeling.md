# Software Architecture
UniGate is built in three layers that work together: the Frontend (what users see), the Backend (the brain that processes requests), and the Database (where all data is stored).

Here's how each user type interacts with the system:
Guest browses programs:
- Guest selects a program to view (Frontend)
- Request goes to Backend
- Backend checks Database (programs table — degree type, duration, language, fees, entry requirements)
- Backend sends program details back to Frontend
- Guest sees the full program information displayed

Student submits an application:
- Student fills out the application form and uploads documents (Frontend)
- Request goes to Backend
- Backend checks Database (applications table:verifies student hasn't already applied to that university)
- Backend sends confirmation back to Frontend
- Student confirms submission
- Application and documents stored in Database

University Staff reviews an application:
- Staff selects an application to review (Frontend)
- Request goes to Backend
- Backend checks Database (applications table + documents table: fetches submitted details and uploaded files)
- Backend sends application details back to Frontend
- Staff updates application status (accepted/rejected)
- New status stored in Database

Admin manages users:

- Admin selects a user account to manage (Frontend)
- Request goes to Backend
- Backend checks Database (users table:fetches account details and assigned role)
- Backend sends user data back to Frontend
- Admin confirms changes (role update / deactivation)
- Changes stored in Database

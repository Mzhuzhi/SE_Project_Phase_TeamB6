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

- Component Diagram (UML)

   <img width="545" height="332" alt="image" src="https://github.com/user-attachments/assets/3dd1cd89-ea6f-4ce4-b731-668cd164e6b8" />

   - Detailed Design
   - Class Diagram

     <img width="940" height="626" alt="image" src="https://github.com/user-attachments/assets/2567644e-0f17-49ba-b92a-de7dad7d5dc1" />


-Sequence Diagram

<img width="648" height="614" alt="image" src="https://github.com/user-attachments/assets/579d46a5-8c64-4753-80d9-32b9c6377f88" />

<img width="614" height="448" alt="image" src="https://github.com/user-attachments/assets/290f51f2-0af8-4ae7-8f8b-e8815d9279cf" />

<img width="585" height="650" alt="image" src="https://github.com/user-attachments/assets/597509f5-f94e-4021-8506-c8d1c70eb4c1" />

- USE CASE DIAGRAM

  <img width="940" height="575" alt="image" src="https://github.com/user-attachments/assets/948f6422-e14a-47f6-8344-5d294f8e4c5a" />

  - Activity Diagram

    <img width="446" height="372" alt="image" src="https://github.com/user-attachments/assets/d2c3a743-8862-4451-86e9-a91e2fdbcfea" />

<img width="242" height="359" alt="image" src="https://github.com/user-attachments/assets/95b6a6a1-7058-4c09-88c2-873b2427655e" />







  

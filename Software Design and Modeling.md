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

  # Component Diagram (UML)

   <img width="1249" height="762" alt="image" src="https://github.com/user-attachments/assets/f8a31607-c1d5-4c06-9532-94268efbb1d7" />


  # Detailed Design
   # Class Diagram

     <img width="940" height="626" alt="image" src="https://github.com/user-attachments/assets/2567644e-0f17-49ba-b92a-de7dad7d5dc1" />


# Sequence Diagram
- Register/Login

<img width="648" height="614" alt="image" src="https://github.com/user-attachments/assets/579d46a5-8c64-4753-80d9-32b9c6377f88" />






- Submit Application

<img width="614" height="448" alt="image" src="https://github.com/user-attachments/assets/290f51f2-0af8-4ae7-8f8b-e8815d9279cf" />








- Review Application


<img width="585" height="650" alt="image" src="https://github.com/user-attachments/assets/597509f5-f94e-4021-8506-c8d1c70eb4c1" />


# USE CASE DIAGRAM


<img width="940" height="575" alt="image" src="https://github.com/user-attachments/assets/eecdb119-5924-4850-8474-aedc4cb28e10" />


  
  
  # Activity Diagram
  - Student Registration and Login



<img width="662" height="849" alt="image" src="https://github.com/user-attachments/assets/458f4695-be80-48e5-a19b-0cf01b5adef2" />



    
     
     
     - Submit Application & Upload Documents



<img width="630" height="1045" alt="image" src="https://github.com/user-attachments/assets/3251a7a8-79ea-4e56-9da4-0a9d2424d6b6" />




 # State Diagram


 <img width="603" height="385" alt="image" src="https://github.com/user-attachments/assets/cf439973-a603-4eaf-9afd-705058f95dba" />



  
  
  
  <img width="489" height="367" alt="image" src="https://github.com/user-attachments/assets/61e53f25-8816-4e13-adff-c01b199c7962" />








  

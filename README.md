# Team Information
## Team Name: UniGate
## Team Leader:
  - Name: Mei Zhuzhi
  - GitHub: MZhuzhi
## Team Members:
1. Mei Zhuzhi - GitHub: MZhuzhi - Email: mzhuzhi23@epoka.edu.al
2. Flavja Llukaj - GitHub: fllukaj - Email: fllukaj23@epoka.edu.al
3. Klea Doci - GitHub: Kdoci - Email: kdoci23@epoka.edu.al
4. Stiv Karabeci - GitHub: SKarabeci - Email: skarabeci23@epoka.edu.al
5. Rei Greca - GitHub: rgreca - Email: rgreca23@epoka.edu.al

# Project Details
## Project Title:UniGate
## Problem Statement
Applying to a university in Albania means going there in person, waiting in long lines, and handing in paper documents one by one. This process takes a lot of time and can be very stressful, especially for students who live in smaller cities or rural areas far from the university. For students with health issues or disabilities, or for international applicants, it can be almost impossible to travel just to submit an application. On top of that, handling everything on paper means documents can get lost, information can be entered incorrectly, and different universities process things in different ways, which makes the whole experience confusing and unfair. Administrative staff also have a hard time keeping track of everything manually. There is clearly a need for a simple, digital solution that makes the application process easier for everyone in Albania.

## Proposed Solution
UniGate is a website that brings the whole university application process online for students in Albania. Instead of going to a university in person, students can do everything from their computer or phone :browsing programmes, filling out an application, and uploading their documents. The goal is to make applying to university easier, faster, and more accessible for everyone, no matter where they live. The platform is designed for three types of users:students,administrator and university staff. Students who have not made up their mind yet can visit the website as guests and look through all available bachelor's and master's programmes without needing to create an account. Once they are ready to apply, they can register and start the application process. University staff, on the other hand, have their own admin area where they can manage programme listings and review applications that come in. UniGate basically replaces the old paper-based system with a clean, organised digital one that works for both students and university staff.

- Programme Catalogue: Anyone who visits the website can browse all available bachelor's and master's programmes, even without an account. Each programme shows information about the degree type, duration, the language it is taught in, conditions of admission, and tuition fees. This helps students compare their options before they decide to apply.

- User Registration and Login: To apply for a programme, students need to create an account. The registration and login system is secure and checks that all the information entered is correct. Guests can still browse programmes without an account, but they need to register if they want to submit an application.

- Online Application Form: Once logged in, students can fill out an application form step by step. The form asks for personal details, school background, and the programme they want to apply to. It also checks for mistakes as the student fills it in, so they can fix any problems before sending it.

- Document Upload: Students can upload all the documents they need to send with their application, such as their passport, ID, school grades, CV, motivation letter, and recommendation letters. The platform accepts PDF, JPG, and PNG files. Everything is saved safely and connected to the student's application.

- Admin Dashboard: University staff have their own section on the platform where they can add, edit, or remove programme listings. They can also see all submitted applications, check the uploaded documents, and update the status of each application. This replaces the old way of doing things on paper.

## Project Scope
- Aim
To build a working website that lets students in Albania browse university programmes and apply to them online, without needing to go there in person.

- Objectives
•	Build a website that works on computers, tablets, and phones
•	Create a list of all available bachelor's and master's programmes with full details for each one
•	Add a login and registration system so students can create accounts and apply
•	Build an application form that students fill out step by step, with error checking as they go
•	Let students upload their documents like passport, CV, and school grades in PDF, JPG, or PNG format
•	Create an admin page where university staff can manage programmes and check submitted applications
•	Make sure all user data and uploaded files are stored safely
•	Write clean code and prepare proper documentation and a final presentation

Within Scope
•	Web platform development: full front-end and back-end implementation
•	User authentication: registration, login, and session management for applicants
•	Programme catalogue : browsable listings with detailed information for each degree
•	Online application form structured, multi-field form for submitting an application
•	Document upload module : secure upload and storage of application documents
•	Admin dashboard : interface for managing programmes and reviewing applications

Outside of Scope
•	Payment processing or scholarship management
•	Physical document verification or notarisation
•	Post-submission application tracking or status notifications
•	Direct email communication between applicants and universities via the platform
•	Mobile native applications (iOS / Android)

## Application Description
Overview
UniGate is a website that brings the whole university application process online for students in Albania. The platform is designed to be simple and easy to use, whether you are a student looking at your options for the first time or a university staff member reviewing applications. There are three types of users on the platform. Guests can visit the website and look through all available programmes without needing to sign up. This is useful for students who are still deciding what to study or where to apply. Registered students have access to the full application process: they can fill out their application, upload documents, and manage their profile. Admins are university staff members who have a separate dashboard where they can add and update programme listings and go through the applications that students have submitted.

Key Features
- Programme Catalogue: Students and guests can browse and search through all available bachelor's and master's programmes. Each programme page shows important details like the degree type, duration, language of teaching, entry requirements, and tuition fees, so students have everything they need to make a decision.
- Login and Registration :Students can create a free account using their email and a password. Once logged in, they can access the full application process. Guests who have not registered can still browse all programmes, but they will need an account before they can apply.
- Application Form: When a student is ready to apply, they fill out a form that is split into several steps. Each step covers a different part of the application, such as personal details, academic history, and programme choice. The form also checks for mistakes in real time so students can fix them before submitting.
- Document Upload : As part of the application, students can upload all the documents they need to include, such as their passport, school transcripts, CV, motivation letter, and letters of recommendation. The platform accepts PDF, JPG, and PNG files, and all uploads are saved securely.
- Admin Dashboard : University staff have their own login and a separate dashboard where they can manage everything on their side. They can add new programmes, update existing ones, and go through submitted applications. They can also open uploaded documents and update the status of each application.
- Responsive Design : The website is built to work on any screen size, so students can use it on a phone, tablet, or computer without any issues.

# Roles and Tasks
## Team Leader: Mei Zhuzhi
Frontend: Build the admin dashboard where university staff can see all submitted applications in a table and click to view more details.
Backend: Create routes to get all applications, get details of one application, and update the application status (accept/reject).
Database: Use data from the applications and documents tables.
Other: Help integrate all features and make sure everything works together at the end.

## Team Members: 
1. Rei Greca – Application Form
Frontend: Create a one-page form where users fill in basic information like name, email, phone number, programme, and school.
Backend: Send the form data to the server and handle saving it.
Database: Store the data in the applications table and connect it with the user and selected programme.

2. Klea Doci – Document Upload
Frontend: Build a simple upload section where users can select and upload files.
Backend: Handle uploading files (PDF, JPG, PNG) and saving them.
Database: Store file paths in the documents table and connect them with the correct application.
  
 3. Stiv Karabeci – Login and Register
Frontend: Create login and registration pages.
Backend: Implement authentication logic (sign up, login, password handling).
Database: Create and manage the users table.
Other: Handle validation such as  correct email, password rules and test login functionality.

4. Flavja Llukaj – Programme Catalogue
Frontend: Create pages to display a list of programmes and a detailed page for each programme.
Backend: Implement functionality to add, edit, delete, and fetch programmes.
Database: Create the programmes table and store all programme data.
Other: Add search/filter and test the pages.

## Shared Responsibilities
All members will help connect frontend, backend, and database parts of the application
All members will participate in testing and fixing bugs
All members will contribute to documentation and final presentation














##Vue Frontend – After-School Lessons App

This is the frontend for the After-School Lessons Web Application, developed for the CST3144 – Full Stack Development coursework.
It is built using Vue.js (Vite) and communicates with the Express.js backend hosted on Render.

#Features

Displays all available lessons

Search and sort system (subject, location, price, spaces)

Add/remove items from shopping cart

Checkout form with validation (name + phone)

Order submission to backend API

Dynamic update of lesson spaces

“View Cart” toggle and success message system

#Project Setup

Install dependencies:

npm install


Run the development server:

npm run dev


Build for production:

npm run build


The production build will be created inside the dist folder.

API Connection

The frontend fetches lesson and order data from the Express backend.

Update the API base URL in App.vue after deploying your backend:

https://your-backend.onrender.com


#Endpoints used:

GET /lessons

POST /orders

PUT /lessons/:id

#Hosting

The frontend is deployed using GitHub Pages.

Live URL:
(add when deployed)

##Author
#Muhammad Rayan
Student ID: M00957926
Full Stack Development

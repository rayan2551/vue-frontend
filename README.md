##Vue Frontend – After-School Lessons App

This is the frontend application for the After-School Lessons Web App, developed for the CST3144 – Full-Stack Development coursework.
The frontend is built with Vue.js (Vite) and communicates with the Express.js backend deployed on Render. It displays lessons, allows users to search, sort, add lessons to a cart, and complete checkout with full validation and backend integration.

#Features

Display all available lessons from the backend

Search lessons (subject, location, price, spaces)

Sorting system (ascending / descending)

Add & remove lessons from the shopping cart

Checkout form with validation (name + phone)

POST order submission to backend API

PUT request to update lesson spaces after checkout

“View Cart” toggle and success confirmation banner

#Project Setup

Install dependencies
npm install

Run development server
npm run dev

Build for production
npm run build


The production build is generated inside the /dist folder.

#API Connection

The frontend communicates with the backend deployed on Render.

Update the API base URL inside App.vue if needed:

https://express-backend-1g5j.onrender.com

#Endpoints Used

GET /lessons – retrieve all lessons

POST /orders – submit a new order

PUT /lessons/:id – update lesson spaces

#Hosting

The frontend is deployed using GitHub Pages.

🔗 Live Site:
https://rayan2551.github.io/vue-frontend/

#Author

Muhammad Rayan
Student ID: M00957926
CST3144 – Full-Stack Development

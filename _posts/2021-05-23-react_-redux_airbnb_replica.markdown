---
layout: post
title:      "React Redux Airbnb Replica"
date:       2021-05-23 18:06:16 -0400
permalink:  react_-redux_airbnb_replica
---

For my final project, I created an Airbnb replica. 

The first thing I did was create a Rails backend consisting of 4 migrations, models and controllers - Users, Rentals, Reservations and Trips. 

The next step was to associate my models to one another. Using Rails associations, my model objects are able to understand they belong to one another or have many of another object. Each user has_many Reservations, Rentals and Trips. Each Reservation belongs_to a Rental and to a User as a "Guest". A Rental has_many Guests through Reservations. 

Because I am using Rails backend API to connect to the frontend of the application, it was important for me to implement a serializer service class for all of my models. I decided to use the Active Model Serializers gem to assist. Once the gem was downloaded, it was very easy and flexible to get my serializers prepared. I entered the data needed when fetching from the frontend and left behind the extra data that was not necessary.

The next step in creating my application was to write the routes in my config file and actions in my controllers. I also created a Sessions controller and used the bcrypt gem to provide authorization when logging in or signing up Users. Because I wanted to guarantee the objects persisted to the database were valid, I created custom validations in my Reservation model. Using this prevented Reservations from overlapping one another or Reservations being in the past. 

Once I had my backend successfully set up, I began my frontend journey using React front end framework. This framework provides a standardized way of deploying parts in a web application. Using the create-react-app node package, I was able to quickly create the barebones file structure for my front end and execute an initial npm install.

I used a few middleware’s in my application – React Router, Thunk, Redux. Since I am creating a single page application, I used React Router to handle the Client-Side routing. Redux stores all necessary data from the application in a Javascript object that is separate from the components providing every component with the ability to get state data regardless of their position in the dependency tree. Thunk helps us incorporate asynchronous code into our React app (such as fetch requests). 

Using functions from the Redux library, I was able to create an application keeping my state separate from my components. React is formatted such that an action is dispatched which calls a reducer and then renders the review. The dispatch method is listening for updates in the state data and will re-render once the state is changed. I used this format to do a few different things, such a fetch Rentals, create a Rental and fetch Reservations. I created and held my store in the Index.js file where I used the CreateStore function from the Redux library. Passing in the store at the index.js using Provider allowed for all of my components to have access to the store. 

I did my best to keep my components stateless so that in the future if I wanted to use a different middleware, it would be an easy change. 




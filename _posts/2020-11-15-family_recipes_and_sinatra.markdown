---
layout: post
title:      "Family Recipes and Sinatra  "
date:       2020-11-15 16:46:53 -0500
permalink:  family_recipes_and_sinatra
---

My grandmother and aunt cook a lot. Growing up they taught me how to make chicken & dumplings, biscuit donuts, cheesy potatoes, red velvet cupcakes and much more. Over the years, I have come to slightly memorize the recipes but do not know them by heart. My grandmother stores all her recipes in a drawer in the kitchen and it has started to worry me. What if her kitchen catches on fire? What happens if all her recipes are destroyed?

When starting my Sinatra Application Project, I decided to create something that was important to me, a place to store these family recipes. I call the application – Family Recipes. Super original. 

I started the application with the Corneal gem which assists in scaffolding out a Sinatra Project. This created a lot for me automatically, a Gemfile, Rakefile, App directory, db directory, config directory as well as other folders and files. Inside the App directory, the gem created a models, views and controllers directory. It also helped configure the sqlite3 database connection in a config/environment.rb file. 

In order to create this MVC (models, views, controllers) application, I used ActiveRecord with Sinatra. These gems accelerated the application design by providing pre-configured methods that assisted in letting the user create, read, destroy and update recipes. 

In my application I wanted to have a users table and a recipe table. I created two migrations, one that created a users table and one that created the recipes table. Each user has a First Name, Last Name, Username, Email and Password. Each recipe has a Name, Ingredients, Directions, Meal Type and User Id. 

I then created a model for both User and Recipe. Remember the pre-configured methods that I mentioned earlier? In order for my models to access these, I ensured they inherited ActiveRecord. This allowed for some very important methods are required when navigating through a recipe book – initialize, .find_by, .find, .all, .create, .new, .update.

But how does a recipe know it belongs to a user? Or how does a user know which recipes it created? In order to solve this, I used a belongs to and has many relationship. In the Recipe model, I listed that a recipe belongs to a user. In the User model, I listed that a user has many recipes. Listing these relationships in my models added additional methods that allowed for my application to connect the two models & tables and provide relationships. This is also the reason I created a User Id attribute for the Recipe model. 

After building the tables and models, I created a seed.rb file in the db directory with users and recipes. This filled my table with seed information so I could test my routes and methods. Using the rake gem, I ran a rake db:seed command to feed the information into the database. 

My last step was to create the controllers and views. The User model required a controller as did the Recipe model. To ensure the User could login, signup and logout, I created dynamic routes to access each of these pages and form.  I also implemented sessions so that I could reference the current user in my routes and methods as well as encrypt passwords for each user. 

At the top of each webpage, the logged in user has options to navigate to the recipes page, the users page, a new page and a logout link. 

The recipes controller allows a user to view, edit, create and destroy recipes. The recipes index file lists all recipes by meal type for the user to view. These links navigate the user to a show view which provides the recipe details. It also provides the user with the option to edit or delete a recipe. If the user created the recipe, they will be able to edit/delete. Otherwise, they will be redirected to the recipe show page and given a flash error message. 

The users page will provide a list of users. Each user link will navigate to a user show page with all recipes listed that the user created. 

The new page will bring the user to a form where they may create a new recipe. The name, ingredients, directions and meal type are all required. Once this form is submitted, it will use a create method (from ActiveRecord) to create a new Recipe. It will also assign the recipe a user_id with the id of the session user (using Sessions). The .save method will then be implemented to save the object to the Recipe database. 

With the help of the Sinatra gem, the ActiveRecord gem, and many others, I was able to easily create a place for my family to store recipes and share with other users. 


---
layout: post
title:      "BeautyMe Rails Application  "
date:       2021-01-24 12:08:14 -0500
permalink:  beautyme_rails_application
---


To complete the requirements for the third module of Flatiron’s program, I decided to create a Rails application based on scheduling for beauticians, stylists, salon owners, etc. This application allows Users (stylists) to create Customers, create Categories and Appointments for their Customers. I designed BeautyME using the MVC (model, view, controller) framework. The MVC Framework is used to separate concerns between the login (models), the views (user interaction) and the controller (routes). 

After using the Rails app generator (```$ rails new```) to build the base of the application, I ensured the Gem file had everything it needed to make BeautyMe run smoothly. 

The pre-configured Gem file included Rails, ActiveRecord, and Bcrypt. I added Simple Calendar (to easily display a calendar), Pry, Omniauth (for logging in via Facebook), Omniauth-Facebook and Dotenv-Rails. 

Once the Gem file was installed, I focused on designing the models and database. I created multiple migrations with the following attributes:

```
class CreateUsers < ActiveRecord::Migration[6.0]
  def change
    create_table :users do |t|
      t.string :first_name
      t.string :last_name
      t.string :email 
      t.string :password_digest
    end
  end
end
```

```
class CreateCustomers < ActiveRecord::Migration[6.0]
  def change
    create_table :customers do |t|
      t.string :first_name
      t.string :last_name
      t.string :email 
      t.string :phone_number
      t.integer :user_id 
    end
  end
end
```

```
class CreateAppointments < ActiveRecord::Migration[6.0]
  def change
    create_table :appointments do |t|
      t.string :name 
      t.datetime :start_time
      t.datetime :end_time
      t.float :price 
      t.integer :customer_id
      t.integer :category_id
      t.integer :user_id 
      t.timestamps
    end
  end
end
```

```
class CreateCategories < ActiveRecord::Migration[6.0]
  def change
    create_table :categories do |t|
      t.string :name 
      t.integer :user_id 
    end
  end
end
```

```
class CreateSessions < ActiveRecord::Migration[6.0]
  def change
    create_table :sessions do |t|
      t.integer :user_id
    end
  end
end
```



After running the migrations, I created a Ruby file for each model and ensured the files inherited ActiveRecord. The ActiveRecord Gem accelerated the application design by providing pre-configured methods that assisted in letting the user create, read, destroy and update appointments, users, customers and categories. These methods include .find, .find_by, .all, .find_or_create_by, .build, .save, .destroy and many others. 

To further enhance the models intelligence, I created associations between them. This allows the model objects to understand they belong to one another or have many of another object. I used the belongs_to, has_many, and has_many through to create the below associations:


```
class User < ApplicationRecord
    has_many :customers
    has_many :appointments
    has_many :categories
	```


```
class Customer < ApplicationRecord
    belongs_to :user 
    has_many :appointments
    has_many :categories, through: :appointments 
```

```
class Category < ApplicationRecord
    belongs_to :user 
    has_many :appointments
    has_many :customers, through: :appointments 
```

```
class Appointment < ApplicationRecord
    belongs_to :customer
    belongs_to :user
    belongs_to :category 
```

These associations, of course, performed additional magic and added more logic to the models. Additional methods were provided through these relationships. Category objects can now call a method .appointments to see all Appointments associated with the Category. User objects can now call a method .categories to view all Categories associated with the User. 

To guarantee the objects created in the application do not persist to the database if not valid, I created validations in the models. 


User 
```
    validates :first_name, :last_name, {:length => { :maximum => 12, :minimum => 2}}
    validates_uniqueness_of :email, { case_sensitive: false }
```
		

Customer 
```
    validates :first_name, :last_name, :email, :phone_number, presence: true 
    validates :first_name, :last_name, {:length => { :maximum => 12, :minimum => 2}}
    validates :email, uniqueness: { case_sensitive: false }
    validates :phone_number, {:length => {is: 10}}
    validates :phone_number, numericality: { only_integer: true }
 ``` 
 

Appointments 
```
   validates :name, :start_time, :end_time, :price, :customer_id, :category_id, presence: true 
    validates :name, {:length => { :maximum => 20, :minimum => 2}}
    validate :appointment_date_cannot_be_in_the_past, :appointment_end_time_cannot_be_before_start_time, :no_appointments_overlap
```



Category
```
    validates :name, presence: true 
    validates :name, {:length => { :maximum => 12, :minimum => 2}}
    validate :uniqueness_of_category_per_user 
```

As you see above, some validations are simple, validating length, uniqueness, presence, but, I also needed to create custom validations to meet all requirements. 

Appointment times needed to be in the future, end time could not be before start time and no Appointments were allowed to overlap. In order to create custom validations, a custom method is required. Additionally, the custom method must be called on in the same model. Below you will see a few custom validations. Each validation includes a custom error message that will appear if the .errors method is called upon the object. 

```
def appointment_date_cannot_be_in_the_past
      if start_time.present? && start_time < Date.today
          errors.add(:date, "can't be in the past")
      end
end
```

```
def appointment_end_time_cannot_be_before_start_time
      if start_time.present? && end_time.present? && end_time < start_time 
            errors.add(:end_time, "can't be before start time")
      end
end
```

In order to control the routes and views in BeautyMe, I created a controller file and a view folder for each model. I then added basic controller routes to the config/routes file using resources :controller_name. After doing so, I added multiple custom routes to the config/routes file to provide the user with access to sign up, login, logout. I associated them with the correct controller action. 

```
root 'sessions#home'
get '/signup' => 'users#new'
post '/signup' => 'users#create'
get '/auth/facebook/callback' => 'sessions#facebook'
get '/login' => 'sessions#new'
post '/login' => 'sessions#create'
delete '/logout' => 'sessions#destroy'
delete '/users/:id' => 'users#delete'
```
 
To give the User additional login options, I used the Oauth and Dotenv gems. These allowed the use of an Facebook Developer Account to create the option for a User to sign in/signup via their Facebook credentials. 

When creating the sign up and login page, I used the Bcrypt gem to assist with passwords. By adding has_secure_password to the Users model and using :password_digest in the Users table, the application was provided with additional methods. The :password_digest field salts the User’s password so it is safely stored in the database. When logging in, the Sessions controller uses the method .authenticate to ensure the :password stored in the database matches the :password given upon login. After each successful authentication, the Sessions controller assigns the Session’s :user_id with the Users :id before logging the User in. 

To help the controllers and views easily access the current_user, I created the below helper methods in the application_helper.rb file. I called on the redirect_if_not_logged_in method in the Appointments, Customers, and Categories controllers to confirm the User was logged in prior to giving access and displaying information. 


``` 
def current_user 
      @current_user ||= User.find_by(id: session[:user_id]) 
end

def logged_in?
      !!session[:user_id]
end

def redirect_if_not_logged_in 
      redirect_to root_path if !logged_in?
end 
```



In order to create Users, Categories, Appointments, and Customers, I designed forms within each view folder. I used form_for versus form_tag. Form_for creates a form specifically for a model object and provides a number of convenient features. 

When creating the Appointment form, I used a partial form. A partial form provides the opportunity to use the same form for New and Edit, therefore, your code is less repetitive and DRY. In order to access this partial form, I rendered the form in both the :edit and :new view. I created a nested form so the User could also create new Categories and Customers as needed. The form uses a collection select field to provide the User with the option to choose an already existing Category or Customer. 

If there is a validation error for any of the three objects, the page will render the :new view and display all the errors. If the objects have no errors and a new customer or category is created, these attributes are stored in nested hashes. In Rails, you may use a method accepts_nested_attributes_for but, it does not check for duplicates. Therefore, I had to create custom methods inside the Appointment model to confirm the nested attributes would be created. The methods below check to validate the fields are not blank. If the fields are not blank, the methods call the .find_or_create_by method. This method either finds and assigns an object or will create if no object is found. The method then assigns the Appointment’s :customer_id with the new Customer :id to complete the association. 

``` 
def customer_attributes=(attr)
      if attr[:first_name] != ""
            customer = Customer.find_or_create_by(first_name: attr[:first_name], last_name: attr[:last_name], email: attr[:email], phone_number: attr[:phone_number], user_id: self.user_id) 
            self.customer_id = customer.id
      end
end
```

```
def category_attributes=(attr)
      if attr[:name] != ""
            category = Category.find_or_create_by(name: attr[:name], user_id: self.user_id) 
            self.category_id = category.id
      end
end
```


The above methods are called on because of the below .build method in the Appointments create controller action. Rails recognizes that I am calling a User’s Appointments and therefore, associates and builds any other objects in the appointment_params that are associated with a User. The beauty(me) of associations! ;) 

```
def create 
      appointment = current_user.appointments.build(appointment_params)
      if appointment.save
            redirect_to appointment_path(appointment)
      else 
            render :new 
      end       
end
```

In order for the customer_attributes= and category_attributes= to be included in the appointment_params, they must be added to the private appointment_params method. This method is located in the Appointments Controller and creates strong params.

```  
private 

def appointment_params
      params.require(:appointment).permit(:name, :start_time, :end_time, :price, :customer_id, :category_id, :user_id, :customer_attributes => [:first_name, :last_name, :email, :phone_number, :user_id], :category_attributes => [:name, :user_id])
end
```



Because objects have many associations, I used nested routes for additional functionality. This way, a Customer :show view will link to a Category :show view and the route will match as such. To do this, I created the below nested routes in the congif/routes file. 

```
resources :categories, only: [:show] do 
  resources :appointments, only: [:show, :new, :index]
  resources :customers, only: [:show, :index]
end

resources :customers, only: [:show] do 
  resources :appointments, only: [:show, :new, :index]
  resources :categories, only: [:show, :index]
end
```


These routes allowed the :show views for Customers and Categories to have nested routes with Appointments :show, :new and :index, Categories :show, :index and Customer :show :index. In order to guarantee the nested resources worked, additional controller actions were necessary. For example, I had to add an appointments_index and appointments action on both the Categories and Customers controller. This provided views with access to the @appointments/@appointment instance variables and rendered the correct view template. 

Another step I had to take was adding the new nested paths that were provided by the above routes. Nesting the routes in the routes file provided me with access to new paths such as category_customer_path, category_appointment_path, customer_appointment_path, new_customer_category_path. These helped me easily redirect the user to a nested path. 

```
link_to customer_name(customer), category_customer_path(@category, customer)
link_to appointment_name(appointment), category_appointment_path(@category, appointment)
```

A little extra work was still needed to make the :new nested routes for Appointments.  Of course, it was not difficult to have the Customer and Category :show pages lead to a new appointment form but, I needed to ensure the customer_id or category_id were automatically assigned to the appointment once the new form was displayed. Because the link to the :new Appointment form from the Customer and Category :show page was using a new_path (new_customer_appointment_path, new_category_appointment_path), a customer_id and category_id were being added to the params. 

I added a bit to the new action in the Appointments controller. If a category_id or customer_id were present in params, this added functionality allowed these ids to automatically be assigned to the new instance variable @appointment. 

```
def new 
      if params[:category_id] && !Category.exists?(params[:category_id])
            redirect_to categories_path, alert: "Category not found"
      elsif params[:customer_id] && !Customer.exists?(params[:customer_id])
            redirect_to customers_path, alert: "Customer not found"
      else 
            @appointment = Appointment.new(category_id: params[:category_id], customer_id: params[:customer_id], user_id: current_user.id)
      end
end
```

Then, I added a hidden field to the appointments form. This hidden field picks up the customer_id or category_id on the @appointment new object if present. When the appointment is created, the customer_id or category_id are provided in the appointment_params. 

```
<%= f.hidden_field :category_id %>
<%= f.hidden_field :customer_id %>
<%= f.hidden_field :user_id %>
```

The last items I worked on were views. I added many helper methods to manipulate the datetime for Appointment :date, :start_time and :end_time. Because the methods were only needed for views, I added them to the appointment_helper.rb file (reminder – separation of concerns). Luckily for all of us, Rails Action View Forms have a lot of convenient methods to take advantage of, such as number_to_currency or number_to_phone. 

```
def time(appointment)
      appointment.start_time.strftime('%I:%M%P')
end
 
def date(appointment)
      appointment.start_time.strftime("%A") + " " + appointment.start_time.strftime("%m/%d/%Y")
end

def date_time(appointment)
      date(appointment) + " " + time(appointment)
end

def price(appointment)
      number_to_currency(appointment.price)
end

def appointment_name(appointment)
      appointment.name.capitalize 
end
```


I used these helper methods to assist in creating a pleasant view for the user. I also implemented the Simple Calendar Gem to provide the user with a calendar view on the home page. This was quite simple as the methods are written out for you to manipulate on the Simple Calendar README. 

In conclusion, Ruby on Rails was very convenient compared to the last project using Sinatra. With that being said, there is so much intelligence going on behind the scenes it is important to understand how the methods work to ensure you are getting the most out of the magic. 


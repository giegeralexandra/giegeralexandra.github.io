---
layout: post
title:      "React -Redux Airbnb Replica"
date:       2021-05-23 18:06:16 -0400
permalink:  react_-redux_airbnb_replica
---

For my final project, I created an Airbnb replica. 

The first thing I did was create a Rails backend consisting of 4 migrations, models and controllers - Users, Rentals, Reservations and Trips. 

```class CreateUsers < ActiveRecord::Migration[6.0]
  def change
    create_table :users do |t|
      t.string :name
      t.string :email 
      t.string :password_digest
      t.timestamps
    end
  end
end

class CreateRentals < ActiveRecord::Migration[6.0]
  def change
    create_table :rentals do |t|
      t.string :name 
      t.string :description
      t.string :address
      t.string :rental_type 
      t.float :price
      t.integer :owner_id
      t.text :image
      t.timestamps
    end
  end
end

class CreateReservations < ActiveRecord::Migration[6.0]
  def change
    create_table :reservations do |t|
      t.date :checkin
      t.date :checkout
      t.integer :rental_id
      t.integer :guest_id 
      t.integer :total_price 
      t.timestamps
    end
  end
end

class CreateTrips < ActiveRecord::Migration[6.0]
  def change
    create_table :trips do |t|
      t.integer :reservation_id
      t.integer :review_id 
      t.integer :rental_id 
      t.integer :owner_id
      t.integer :guest_id
      t.timestamps
    end
  end
end

```










































































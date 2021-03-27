---
layout: post
title:      "JavaScript & Memory Games "
date:       2021-03-26 20:09:28 -0400
permalink:  javascript_and_memory_games
---


In module 4 of Software Engineering Bootcamp at Flatiron School, I was required to create a single page application using CSS, HTML, JavaScript, Ruby and Rails. I decided to make a memory game for my project consisting of 9 pairs of cards. 

The first step in my project was to create the rails backend application. By running rails new memory_game, Rails created the basic structure for my backend application, including the ActiveRecord Gem. I then created 2 different migrations, one for Users and one for Games.
 
```def change
    create_table :users do |t|
      t.string :username

      t.timestamps
    end
  end

 def change
    create_table :games do |t|
      t.integer :user_id
      t.integer :score
      t.datetime  :start_time
      t.datetime :end_time

      t.timestamps
    end
  end```


I order to associate Games with Users, I created a has_many relationship and set user_id as the foreign key in the Games table. This association between User and Game allows the model objects to understand they belong to one another or have many of another object. 


```
class Game < ApplicationRecord
    belongs_to :user

end

class User < ApplicationRecord
    has_many :games

end```

Because I am using Rails backend API to connect to the frontend of the application, it was important for me to implement a serializer service class for both Game and User. I decided to use the Active Model Serializers gem to assist. Once the gem was downloaded, it was very easy and flexible to get my serializers prepared. I entered the data I needed to be serialized when fetching the data from the frontend and left behind the extra data that was not necessary. 

```
class GameSerializer < ActiveModel::Serializer
    attributes :id, :user_id, :score, :start_time, :end_time, :turns
  
    belongs_to :user
end

class UserSerializer < ActiveModel::Serializer
    attributes :username, :id
  
    has_many :games 
end```
 
The next step was creating the routes in my config file and controllers. I needed an index, show, create for User and an index, new, create, show, edit, and update for Games.

```
Rails.application.routes.draw do
  resources :users
  resources :games
end
```

After I finished setting up the basics of my Ruby Rails backend, it was time for me to move on to the frontend of the application. I started off by creating the User entry form. It was important that each player logged in prior to playing so that the final scores could be saved. Once the form was finished in the index.html file, I needed to grab the information, and post it to the rails API. To do this, I created a User Class in my index.js file and used three static functions to fetch the API and POST the new User data to my database. userFormEventListener() adds a ‘click’ event listener to the submit button on the form. Once it is clicked, it grabs the username entered, sets it to variable ‘username’  and then sends the data to the submitUser() function. The submitUser() function fetches the usersUrl, posts the data to the create route on the User Controller and creates or finds an existing user. Once that is complete, I took the response, turned it into a json string, then into an object . From there, I created a new User object in the User class and assigned it to the variable currentUser.

```
static userFormEventListener() {
        userSubmit.addEventListener('submit', function(e) {
            e.preventDefault(); //prevents page from refreshing
            username = document.getElementById('username').value;
            let data = {
                username, 
            };
            User.submitUser(data);
        })
    }

    static submitUser(data){
        fetch(usersUrl, {
            method: "POST", 
            headers: {
                Accept: "application/json", 
                "Content-Type": "application/json",
            }, 
            body: JSON.stringify({user: data}),
        }).then((res) => {
            return res.json()
        }).then(user => {
            User.assignUser(user);
            // console.log('user fetch worked'),
        })
        logoutNavBarDisplay();
        frozen = false,
        signedIn = true,
        Game.startGame()
        
    }

    static assignUser(user){
        currentUser = new User(user.username, user.id, user.games)
    } 
```


Once the user is created on the backend and the frontend, the startGame() function is called. When a user is not logged in, the board is frozen and no cards will flip over. Once the User has logged in, the variable ‘frozen’ is set to false allowing the cards to move. 

To set up the cards, I created 18 different <divs> with a class name of “memory-card”  inside of a <section> with a class name of “memory-game”. Each div had a front facing image tag and a back facing image tag. The goal was to toggle between the two classes when a card was clicked. Using CSS, I created a different style for each class name flipping back and forth between the front facing and back facing images.

When a game is initiated, the first few things that happen are a ‘welcome’ header and a highest scores header created on the side nav bar. The welcomeNavBarDisplay uses the newly created user and welcomes them to the game using their username attribute. The insertFastestScores() function fetches all games from the rails backend API and sets them to a variable allGames. It then uses two separate functions (fastestGame() and userFastestGame()) to filter and map through the variable allGames. These functions locate the Memory Game’s fastest All Time Score as well as the specific Users All Time Fastest Score. 

```
function welcomeNavBarDisplay(){
    welcomeUser.innerHTML = `<h3 class= "px-4 py-2 border-b border-gray-800">Welcome ${currentUser.username}</h3>`
}


static insertFastestScores() {
        Game.getGames();
        setTimeout(function(){
            Game.fastestGame();
            Game.userFastestGame()
        }, 500)
        setTimeout(function() {
            console.log(userMin)
            document.querySelector('.fastest-score').innerHTML = `<h3 class= "px-4 py-2 border-b border-gray-800">All Time Fastest Time: ${gameMin} seconds</h3>`
            document.querySelector('.user-fastest-score').innerHTML = `<h3 class= "px-4 py-2 border-b border-gray-800">${currentUser.username}'s Fastest Time: ${userMin} seconds</h3>`
        },800)
    }

    static fastestGame(){
        gameMin = Math.min.apply(Math, allGames.map(game => {return game.score == null ? Infinity : game.score;}))
    }

    static userFastestGame(){
        currentUserGames = allGames.filter(game => game.user.username === username)
        userMin = Math.min.apply(Math, currentUserGames.map(game => {return game.score == null ? Infinity : game.score;}))
    }
```




The startGame() function also creates a new game calling upon the createGame() function. This uses a fetch to gamesUrl, posts the new information using the current new users id. It then sets the game to a variable called currentGame through a function, setCurrentGame(game). 

```
static createGame(){
        let gameData = {
            user_id: currentUser.id,
        };
        fetch(gamesUrl, {
            method: "POST", 
            headers: {
                Accept: "application/json", 
                "Content-Type": "application/json",
            }, 
            body: JSON.stringify({game: gameData}),
        }).then((res) => {
            return res.json();
        }).then(game => {
            Game.setCurrentGame(game);
        })
    }

static setCurrentGame(game){
        currentGame = new Game(game.id, game.start_time, game.user_id)

    }
```

Once the game is created, a function activateCardsListener() is called. This goes through each of the <div> cards and adds a ‘click’ event listener. The event listener does a few things. Once a card is clicked, if the user is signed in and the board is not frozen, the click will result in the card class name to toggle between “memory-card” and “memory-card flip”. If the card that is clicked upon is not flipped over, the card will be assigned to a variable called “firstCard”. If the next click is not the same card, the card will be assigned to a variable secondCard. The function will then check to see if the cards are a match or if all cards have been flipped over. If the cards are a match, they will stay frozen on the back side image and the player can select additional cards. If all cards have been turned over, the endgame() function will be called. 

```
static activateCardsListener() {
        cards.forEach(card => card.addEventListener('click', Game.flipCard.bind(card)));
    }

    static flipCard() {
        if (!signedIn){return;} //dont give flip card access if not signed in
        if (frozen){return}; //give flip card access if frozen 
        this.classList.toggle('flip')
        //check to see if a card is flipped, if flipped assign it to this 
        if (!flipped){
            flipped = true; 
            firstCard = this;
        } else {
            if(this===firstCard){return;}//to prevent double clicking first card
            flipped = false;
            secondCard = this;
            //if two cards are flipped, check to see if they match
            Game.checkForMatch();
            Game.checkForGameOver();
        }

    }
```

Once the endgame() function has been called, it fetches to the backend API to patch the new score. Using a fetch to (`http://localhost:3000/games/${currentGame.id}`), the function updates the attribute score of the current game on the backend as well as on the frontEnd. The window is then alerted that the game is over and the final score is pasted to the nav bar. The fastest scores on the nav bar are updated if necessary. 

```
  static checkForGameOver(){
        let cardsLeft = 0; 
        cards.forEach(card=>{
            if (card.className === "memory-card"){
                cardsLeft +=1;
            }
        })
        if (cardsLeft === 0) {
            Game.endGame();
        }
    }

    static endGame(){
        this.updateScore();
        setTimeout( () => {
            window.alert(`Game Over! Final Time: ${currentGame.score} seconds`)
            this.finalScoreNavBarDisplay();
            Game.insertFastestScores();
            }, 2000)
        setTimeout( ()=> {
            Game.startOver()}, 6000)
    }

    static updateScore(){
        let newData = {
            end_time: new Date(),
        }
        fetch(`http://localhost:3000/games/${currentGame.id}`, {
            method: "PATCH", 
            headers: {
                Accept: "application/json", 
                "Content-Type": "application/json",
            }, 
            body: JSON.stringify({game: newData}),
        }).then((res) => {
            return res.json();
        }).then(game => {
            // console.log(game),
            currentGame.score = game.score
        })
    }
```


Once the scores are updated and the patch request is completed, the startOver() function is called and refreshes the board. In order to do so, it changes all class names of the <div> cards to “memory-card” resulting in the front image of the card to be shown. createGame() is called upon and the process starts over. 

```
static startOver(){
        console.log('made it to start over')
            cards.forEach(card => {card.className = "memory-card"});
            Game.createGame();
    }
```


It was tricky getting everything to work but it was well worth it. I hope you enjoyed the game and the cute kitties that you are able to match!


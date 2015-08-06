---
layout: post
title: Odin Project - JQuery and the DOM
---


># [Project: jQuery and the DOM](http://www.theodinproject.com/javascript-and-jquery/jquery-and-the-dom)
> <!--more-->
>Let's give you a chance to dive deeply into working with events by building a Snake game.  If you never played it (likely many years ago in your childhood), check out the [Wikipedia Page for Snake](http://en.wikipedia.org/wiki/Snake_(video_game)).  Basically, it's a snake that you have to move around the board and which grows longer with each piece of food it eats.  You lose if it gets so big it hits itself or if it goes off the board.
>
>If you think about what's required to make a game like this work, it's all well within your grasp -- you'll have to set up a grid and listen for user input events to determine where to turn the snake, then add some logic around how it gets larger by "feeding".  This is where things start getting really fun!
>
>## Your Task
>
>### Setup
>
>1. Set up a Github Repo for this project.  Follow the instructions atop the [Google Homepage project](/web-development-101/html-css) if you need help.
>1. Set up a blank HTML document
>2. Think about what kinds of objects and functions you'll probably need in order to make this snake work.
>2. Create a javascript object that represents a grid of 40x40 squares that all start out containing just a space `" "`.  
>3. Create a `render()` function which cycles through the grid and turns it into a grid of divs on the screen.  Having a simple border around each will make it easier to see (just remember the border increases the box size).  This should be pretty easy with Javascript and a bit of CSS wrangling.  Remember the grid you created in the [Web Dev 101 jQuery project](/web-development-101/javascript-and-jquery)...?
>3. Make one of the center squares an "O" to represent your snake.  Run the `render()` function on page load.
>4. Create a Javascript object to represent your snake.  Give it an initial position of `[20,20]` and an initial direction to the right (`r`).  Also include a variable to represent the current snake, which will be an array of coordinate pairs (so an array of arrays, e.g. `[[20,20]]` to start)
>
>### Bring it to Life
>
>5. Listen for keyboard input from the arrow keys (see the [keypress() docs](http://api.jquery.com/keypress/)) and change the snake's current direction if a different arrow is pressed.  It's probably helpful to use `console.log()` to track this so you know you're doing it right.
>5. Now bring the snake to life by running a game loop that uses `setTimeout()` to perform a new "turn". 
>6. Each turn, invoke a `move()` function which moves the snake one square in the current direction of travel for each turn.
>6. Create logic which ends the game if the snake head goes off the board.
>7. Now create a Javascript object representing "food".  Randomly place a food on the board.
>8. Create the functionality to eat food, which causes the snake to grow one unit of length on the next turn.
>9. Modify your `move()` function (if you need to) to account for a snake of multiple units length.  This is why you need to think of the snake as an array of coordinate pairs.
>10. Create the logic which ends the game if your snake moves onto itself.
>11. Play! 
>
>### The Fun Stuff
>
>1. Instead of a boring old "O", make your snake consist of images from the web.
>2. Keep score -- each food is worth more points the longer your snake is.
>3. Speed up -- speed up the game slightly with each food consumed.
>4. Play!
>12. Upload your snake to Github and put your code down below.


OK, so now the fun stuff starts. I had coded a few games in C#, but the gameloop in javascript is a bit different. I'm not sure if I did it the more efficient way, but thats ok for a first time.

 The `setTimeout(move, speed);` part was implemented in the `move()` function, not in a dedicated gameloop, maybe thats something I should change.

I first tried recording a 2 dimensional array, putting info inside it, changing the info at each step and redrawing the whole board making sure that it matched the grid. It worked but seemed very inefficient. I ended up scraping it all, and only changing the class of my divs on the fly when necessary.


Here's [code](https://github.com/AtActionPark/odin_jquery_and_the_dom), and the [result](http://htmlpreview.github.io/?https://github.com/AtActionPark/odin_jquery_and_the_dom/blob/master/index.html). 

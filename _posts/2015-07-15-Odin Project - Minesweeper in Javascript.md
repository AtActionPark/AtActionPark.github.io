---
layout: post
title: Odin Project - Minesweeper in Javascript
---

># [Project: Minesweeper in Javascript](http://www.theodinproject.com/javascript-and-jquery/minesweeper)
> <!--more-->
>As you continue to expand your knowledge of Javascript, the programs you build should keep feeling more and more natural.  In this project, you'll get a chance to build another classic browser game -- Minesweeper.  It'll take some thought to work out the different logic cases, but it's really rewarding to be able to play something like this.  
>
>## Your Task
>
>Build [Minesweeper](http://en.wikipedia.org/wiki/Minesweeper_(video_game)) in the browser. Stick with a small board (9x9 with 10 mines) to start.  
>
>The basic rules are that the player must flag every mine on the board without clicking on it.  The player gets enough information to solve this because every square that is immediately next to a mine will display the number of mines it is touching.  The player only has exactly as many flags as mines.  Victory is declared when all squares have been cleared except the mines.
>
>1. Set up a Github Repo for this project.  Follow the instructions atop the [Google Homepage project](/web-development-101/html-css) if you need help.
>1. Set up a blank HTML document
>1. Think about how you would set up the different elements within the game.  What objects and functions will you need? A few minutes of thought can save you from wasting an hour of coding.  The best thing you can do is whiteboard the entire solution before even touching the computer.
>2. Build your board object and the `render()` function which displays and updates it.  
>3. To start the game, randomly place the mines within the board and update your board squares to each show the appropriate number of mines it is touching.
>3. Create the logic necessary to update the board whenever a user clicks in a square.  You will want to differentiate between when the user places a flag and when they would like to reveal the square, so [see this Stack Overflow post on listening for Right Clicks with jQuery](http://stackoverflow.com/questions/1206203/how-to-distinguish-between-left-and-right-mouse-click-with-jquery).
>4. Now create the main game logic which checks for failure or victory and then re-renders the board.  You'll need to reveal the clicked square and, if that square doesn't directly touch a mine, also appropriately reveal nearby squares until you get to those that do touch mines directly (play the sample game above to see this behavior in action).
>5. Now add a user input for a new game which asks which board size to use.
>6. Create a timer which counts how long it takes for the user to win the game.
>7. Play!
>8. Push your solution to Github and include it below.


So this one took me a bit longer (also because I started watching Daredevil). I'm still not very comfortable with the js gameloop, the js classes system (or lack of), and the event listener for clicks.
I took some inspiration from [Donald's](https://github.com/donaldali/odin-js-jquery/tree/master/minesweeper) solution, and shamelessly stole all his images.

My code organisation is quite ugly and possibly inefficient, I feel like I still have a lot of work with that.

The code in itself is pretty standard, the only cute function is the `listNeighbourPositions()` that makes calls itself recursively to return a list of all cells neighbours and neighbours of neighbours

Here's [code](https://github.com/AtActionPark/odin_js_tictactoe), and the [result](http://htmlpreview.github.io/?https://github.com/AtActionPark/odin_minesweeper/blob/master/index.html). 

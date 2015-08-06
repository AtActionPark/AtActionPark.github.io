---
layout: post
title: Odin Project - JS Calculator
---


>## [Project: Building An On Screen Calculator Using Javascript](http://www.theodinproject.com/javascript-and-jquery/on-screen-calculator)
> <!--more-->
>You've done a lot of Codecademy lately and so it's time to wean yourself off their super-friendly environment and create some Javascript on your own computer.
>
>In this project, you'll be building many simple exercises to drill in your understanding of the language. You can write them in a script file in your text editor and then run them by copy-pasting into JS Fiddle. You can run your functions (e.g. `my_max()` below) by console logging their output with something like `console.log(my_max([1,56,2,3,-1,0]))` (which would output 56).
>
>Save your solutions to Github and submit them for inclusion here when you're finished!



My first approach was recording all user inputs in a buffer, and try to treat it with regex as a whole. It worked, but was not very elegant, and would have needed more work to handle long strings of operations.

As often, while looking at other student solutions, I found out that there was a much more elegant solution. I ended up re implementing [Hutton's](https://github.com/Hutbytheton/js_calculator) solution.

So here's the [result](http://htmlpreview.github.io/?https://github.com/AtActionPark/odin_calculator/blob/master/main.html). I am not a designer.

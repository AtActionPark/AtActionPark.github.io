---
layout: post
title: Using Canvas to build games in the browser
---

># [Project: Validating a Form with jQuery](http://www.theodinproject.com/javascript-and-jquery/validating-a-form-with-jquery)
>
><!--more-->
>We'll take a step away from games and back to the real world for a relatively simple project where you'll get a chance to build and validate a form.  For inspiration, go to [Mint.com's signup form](https://wwws.mint.com/login.event?task=S) and see how it gives you error messages if you fail to enter the proper information.
>
>## Your Task
>
>Build a browser form like Mint.com's which collects Email, Email Confirmation, Country, Zip Code, Password and Passowrd Confirmation fields.  It should use live inline validation to inform the user whether a field is properly filled in or not.  That means highlighting a field red and providing a helpful error message until it has been filled in properly.  
>
>The form doesn't need to actually submit, but you should give an error message if the button is pushed with any active errors or unfilled required fields.  If all is well and the form is "submitted", give the user a high five.  
>
>Do all this using [jQuery's Validate plugin](http://jqueryvalidation.org/) but then try it without any plugins to challenge yourself.  See some of the readings for examples of setting up your own validators.
>
>1. Set up a Github Repo for this project.  Follow the instructions atop the [Google Homepage project](/web-development-101/html-css) if you need help.
>1. Set up a blank HTML document
>1. Think about how you would set up the different form elements and their accompanying validators.  What objects and functions will you need? A few minutes of thought can save you from wasting an hour of coding.  The best thing you can do is whiteboard the entire solution before even touching the computer.
>2. Write the simple form elements.  Don't worry about styling them.
>3. Build the validation logic.
>4. Test out all possible cases.
>5. Push your solution to Github and submit it below.

Although the goal was mostly to understand how to use jQuery Validatie plugin, I spent most of the time recreating the site's appearance. Not a lot of things to say, once you get the syntax, the validation plugin is pretty easy to use.

It took me a while because of a stupid mistake though; I first tested the validate() method by adding it in a script directly on my html page, and forgot to delete it. I couldn't understand why I would get default validation error messages instead of those I entered on my script. Oh well, now I know. 

[html preview](http://htmlpreview.github.io/?https://github.com/AtActionPark/odin_jquery_validating/blob/master/index.html)
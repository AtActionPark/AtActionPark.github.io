---
layout: post
title: Using a Rails Back End to Build "Where's Waldo"
---

># [Project: Using a Rails Back End to Build "Where's Waldo" (a Photo Tagging App)](http://www.theodinproject.com/javascript-and-jquery/where-s-waldo-a-photo-tagging-app)
<!--more-->
>This project will finally give you a chance to tie together everything you've learned so far.  It's a project with a lot of complexity so take it one step at a time.  Working with Rails as a back end is pretty straightforward but you'll be juggling some of the front end functionality too.  This is entirely within the realm of the kind of thing you might be asked to build on the job (though of course not exactly).  Are you ready???
>
>Hopefully you got a chance to play "Where's Waldo" ([see here if unfamiliar](http://en.wikipedia.org/wiki/Where's_Wally%3F)) as a kid.  You're given a photograph with tons of stuff going on and you're meant to locate several familiar characters
>
>## Your Task
>
>Here, you'll build an app that feels a whole lot like a photo tagging app.  You'll start with a large photograph which contains several elements the user is meant to find, e.g. Waldo, The Wizard, Wilma etc... name your own if you'd like to use your own photos.  The user will make selections for each character and they will get feedback on whether they are correct or not.
>
>To start with, you'll need to choose a photo and identify where exactly each person is to start with and save that to your database.  When the user clicks the photo, it should place a targeting box around the portion of the photo where the user has clicked.  That box should contain a list of possible characters.  
>
>When the user selects one of these characters, you should check with your Rails Backend to see if that character is actually within the targeting box.  Provide the user with appropriate feedback (e.g. if wrong, an error message).  If correct, place a marker on the photo in the character's location.  In either case, remove the targeting box until the user clicks again.
>
>Keep track of how long it takes between when the photo is first loaded and when the user finally identifies all characters (do this on the server side otherwise the user could hack their score).  Once a round is complete, ask the user for his/her name and record that time.  This will get a bit tricky since you'll have annonymous users you need to keep track of!
>
>1. Set up a Github Repo for this project.  Follow the instructions atop the [Google Homepage project](/web-development-101/html-css) if you need help.
>1. Think about what you'll need to do to get this all working together.  This is where it's really helpful to think it completely through on paper or whiteboard ahead of time!  A few minutes of thought can save you from wasting an hour of coding. 
>2. Create a new Rails application with just the bare bones required to load your HTML page for now.
>2. Build the front end functionality without actually using any calls to the back end yet.  Specifically, that means creating the Javascript functionality which pops the targeting box and dropdown menu on the screen when the user clicks on the photo and removes it when the user clicks away.
>3. Now hook up the functionality for validating with your back end whether the user has clicked in the right place for that character.
>4. Tie it into your front end so you can seamlessly select characters, validate them, and place the appropriate markers on the map if the selection was correct.
>5. Add in the ability to time the user from when they first load the page and then display their "score" (time) when they successfully identify all characters.  Create a popup that asks them to enter their name for the high scores table if they have earned it.
>6. Play with it!
>3. Push your solution to Github and submit it below.  This is a serious project, congratulations!
>
>### Optional Extensions:
>
>1. Load many images into your database and allow your use to select from among them before starting the game.


Ok, this one is gonna be harder. Mostly because I feel like I forgot everything about rails already, but thats a good way of getting back to it.

Setting up the main image, base html css and javascript is quite easy, rails takes care of a lot. I thought CoffeeScript would understand basic JS but its not the case, I'll have to look at that later.

The only thing that was actually complex was remembering the lesson on API's.
It's working like that:

Create a route to an adress (in my case '/checkAnswer') that returns some json  if it checks out, or nothing

{% highlight ruby %}
def checkCharacter
  answer = checkAnswer(params[:x],params[:y],params[:character]);
  if (@@counter == 3)
    flash[:custom] = "Nice! Finished in #{score} seconds"
    render js: "location.reload()"
  else
    render json: answer
  end
end

def checkAnswer x,y,name
  c = Character.where(name: name.to_s)[0] 
  if(x.to_i> c.left.to_i && x.to_i<c.left.to_i + c.width.to_i && y.to_i>c.top .to_i && y.to_i<c.top.to_i + c.height.to_i)
    @@counter += 1
    return { name: c.name, top: c.top,  left: c.left, width: c.width, height: c.height }
  else
    false
  end
end
{% endhighlight %}

In the javascript of the page, create an AJAX request to that location, sending the name of the character we are looking for, and the coordinates of the click.

If some json is returned, it means we clicked on the right person, and we can css it out.

{% highlight javascript %}
function checkAnswer(name){
  $.ajax({
      url:'/checkCharacter',
      type: "GET",
      data: {character: name, x: mouseX, y: mouseY},
      success: function(json){
        handleAnswer(json);
        $('#targetingBox').hide();
      },
      error: function( xhr, status, errorThrown ) {
          alert( "Error, see log" );
          console.log( "Error: " + errorThrown );
          console.log( "Status: " + status );
          console.dir( xhr );
      },
    })
}

function handleAnswer(data){
  if(data)
    hideCharacter(data.name,data.left,data.top,data.width,data.height);
  else
    alert('nope');
}
{% endhighlight %}

Thats it for the main part, the rest is just pretty straightforward

Here is the [code](https://github.com/AtActionPark/odin_waldo) and [html preview](https://hidden-sierra-6699.herokuapp.com/)
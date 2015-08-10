---
layout: post
title: Infinite Scroll and Submitting a Form with AJAX
---

> [Project: Infinite Scroll and Submitting a Form with AJAX](http://www.theodinproject.com/javascript-and-jquery/infinite-scroll-and-submitting-a-form-with-ajax)
><!--more-->
>Because we haven't yet linked your front end projects to your back end Rails APIs, in this project you'll get a chance to set up a front end form but submit to an existing API on the internet called the [Open Movie Database (OMDB)](http://www.omdbapi.com/))(modeled after [IMDB](http://www.imdb.com)). Read through the OMDB documentation before getting started so you have an idea of how you might submit a new movie entry.
>
>
>## Your Task -- NOTE: The OMDB submission functionality is unavailable, we're figuring out a solution to that.  Until then, go to Part II, which involves pulling from the existing data instead.
>
>Create a form which will submit a new movie to the database.  You should validate that the title and other required attributes are not blank.
>
>1. Set up a Github Repo for this project.  Follow the instructions atop the [Google Homepage project](/web-development-101/html-css) if you need help.
>1. Set up a blank HTML document
>1. Think about how you will need to set up the form and then how that form will submit to the OMDB API.  What objects and functions will you need? A few minutes of thought can save you from wasting an hour of coding.  The best thing you can do is whiteboard the entire solution before even touching the computer.
>2. Write the simple form elements.  Don't worry about styling them.
>3. Build the validation logic.
>4. Make the form actually submit a movie to the database.
>4. Create a "loading..." status while AJAX is processing and which disappears after the AJAX call has finished.
>5. Play around with this form
>
>## Part II: Infinite Scroll
>
>5. Once you have successfully submitted a new movie to the database, let's grab a whole bunch of movies and display them.  Remove the form from the page with jQuery and then make another AJAX call to the database to retrieve and display 10 movies on your page. 
>6. Create an infinite scroll that loads another 10 movies and adds them to the bottom every time you scroll down to the bottom of the page.  Your "loading..." icon should come into play here too while waiting for the next batch of movies to be added.
>7. Play around with this new scroll.  What breaks it?
>8. Push your solution to Github and include it below.


Liked this one a lot:  I had been puzzled by AJAX for a while. A bit afraid of it even. 

First part was understanding how to retrieve info from the omdb with ajax. The core of it was the ajaxSearchId function:

{% highlight javascript %}
function ajaxSearchID(id){
  $.ajax({
 
      // The URL for the request
      url: "http://www.omdbapi.com/?i=" + id + "&r=json",
 
      // Whether this is a POST or GET request
      type: "GET",
 
      // The type of data we expect back
      dataType : "json",
 
      // Code to run if the request succeeds;
      // the response is passed to the function
      success: function(json) {
        displayMovie(json);
      },
 
      // Code to run if the request fails; the raw request and
      // status codes are passed to the function
      error: function( xhr, status, errorThrown ) {
          alert( "Error, see log" );
          console.log( "Error: " + errorThrown );
          console.log( "Status: " + status );
          console.dir( xhr );
      },
 
      // Code to run regardless of success or failure
      complete: function( xhr, status ) {
        console.log( "Complete" );
      }
  });
}
{% endhighlight %}

It takes an IMDB id and runs the displayMovie function that creates the relevant html elements by reading the returned json.

The infinite scroll is quite easy too:

Check if the user is scrolling, and if he is, check if the scroll is at the end of the page

{% highlight javascript %}
$(document).scroll(function(){
    var scrollHeight = $(document).height();
    var scrollPosition = $(window).height() + $(window).scrollTop();
    if ((scrollHeight - scrollPosition) / scrollHeight === 0 && loading == false) {
      // the scroll is at the end of the page and page is not already loading
      loadMovies(10);
  }
})
{% endhighlight %}

Quick and easy, yet still eyes opening. 

Once again (its starting to become a habit), I "borrowed" some of the work from [donald](https://github.com/donaldali/odin-js-jquery/tree/master/ajax_infinite_scroll). He prepared a neat list of a bunch of movies sorted by rating, and I can't resist the opportunity of doing less work than necessary...

[html preview](http://htmlpreview.github.io/?https://github.com/AtActionPark/odin_ajax_scroll/blob/master/index.html)

---
layout: post
title: Building a kickstarter clone with rails part 2
---

So the Users and projects are working. Almost. But there is still a lot to be done.
<!--more-->

We are gonna need to be able to bring in the (fake) donations!

So lets start with adding an amount fiel to the project model and creating a donation model

{% highlight ruby %}
class Donation < ActiveRecord::Base
  belongs_to :user
  belongs_to :project

  validates_presence_of :user, :project, :amount
  validates :amount, :numericality => { :greater_than => 0 }
end
{% endhighlight %}

Nothing fancy here. Just make sure to add has_many :donations to both the user and project model.

Then add a integer field and button to the project show page so that a logged user can decide to donate:

{% highlight ERB %}
# app/views/projects/show.html.erb

...
 <% if current_user %>
  <%= form_for(@project.donations.build, remote: true) do |f| %>
    <%= f.hidden_field :project_id %>
    <div class="field">
      <%= f.number_field :amount, placeholder: "Montant" %>
    </div>
    <%= f.submit "Contribuer", class: "btn btn-primary" %>
  <% end %>
<% else %>
  <div class="asideElement">
  <div class="connect"><%= button_to("Se connecter", new_user_session_path, class: "btn btn-primary",:method => :get) %> </div>
  </div>
<% end %>
{% endhighlight %}


Note the remote: true in the form declaration. It says to rails: "hey, Im gonna use ajax on the submit, because I dont really feel like reloading the whole page everytime I click the button"

That also means the controller will need to know that we will respond to something else that html. In this case js.

The whole code for the controller is :
{% highlight ruby %}
class DonationsController < ApplicationController
  before_action :authenticate_user!

  def create
    @donation = current_user.donations.build(donations_params)
    @amount=0
    if @donation.save
      p = Project.find(donations_params[:project_id])
      @amount = p.amount.to_i + donations_params[:amount].to_i
      p.update_attribute(:amount, @amount)
      respond_to do |format|
        format.html { redirect_to root_path }
        format.js 
      end
    else
      respond_to do |format|
        format.html { redirect_to root_path }
        format.js {}
      end
    end
  end

  private
    def donations_params
      params.require(:donation).permit(:project_id, :amount)
    end
{% endhighlight %}

So what happens?

The form submits the project id and the amount as params. We then create a donation by building directly from the user, so that the user id will get referenced too.

On save, we then update the project amount attribute by adding the current donation, then proceed to execute the js in app/views/donations/create.js.erb:

{% highlight ERB %}
if (<%= @amount %> > 0){
  $('.amount').html(<%= @amount %> + "€");
  $('.amount').fadeIn(100).fadeOut(100).fadeIn(100).fadeOut(100).fadeIn(100);
  $('.alert').remove();
  $('.projectShow').before('<div class="alert alert-success">Votre participation a été prise en compte, merci!</div>');
}
else{
  $('.alert').remove();
  $('.projectShow').before('<div class="alert alert-error">Montant incorrect</div>');
}
{% endhighlight %}

The project new @amount is passed so that we can update directly on the page without reloading. We also add a flah and some extremely fancy animation.
If the @amount is <= 0 (if the donation failed), we just display a flash alert.

I also want to track the number of unique contributors to a project. Thats easy!

I added a method in my project model:

{% highlight ruby %}
def contributors
    Donation.where("project_id = ?", self.id).select('DISTINCT user_id').count()
end
{% endhighlight %}

Each project can check the donations table for all donation to himself, and then count the number of unique users.

The Project show page can then pass a @contributors = @project.contributors  variable to the view for display.


Similiar to that, I added a time limit for project, in days.

My project model has a new method to return the remaining days:

{% highlight ruby %}
def remainingTime
  p = Project.find(self.id)
  (p.timelimit - (Time.now - p.created_at)/1.day).floor
end
{% endhighlight %}

It calculates the difference between the creation date and today, and subtracts it from the time limit in days.
By creating a @remainingdays variable in my project controller, I can also pass it to the project view.

I end up with something like that:

![result](/images/gimmeplz1.png)

Look at all the money I'm gonna make!

I also took some time to fix the projects index page. I redid all the css and added pagination and infinite scroll with ajax. I tried to do it the way I did the odin project ajax scroll tutorial, but after a lot of sweat and tears, I ended up following [this tutorial](http://www.sitepoint.com/infinite-scrolling-rails-basics/).

Its always easier when you dont have to think...

And a message for future me, I hope you know coffeescript by now. Having to translate into js everytime to make sure you understand everything is not a viable option in the long term.




Here is the [code](https://github.com/AtActionPark) and [html preview](https://shielded-taiga-9226.herokuapp.com/)
---
layout: post
title: Building a kickstarter clone with rails part 4
---

Finishing up?
<!--more-->

There is still a few things I'm not happy about: The way I'm handling the selection of projects on the home page, and the way I'm handling the projects "expiration date".

For the expiration date, it would be better if the project model had an 'expired' attribute. So lets do it!

First add a boolean expired field to the model through a new migration:

`rails g migration AddExpiredToProjects expired:boolean` 

After migrating, I only need to change a few stuff:




{% highlight ruby %}
# app/models/project.rb

def remainingTime
    p = Project.find(self.id)
    rem = [(p.timelimit - (Time.now - p.created_at)/1.day).ceil,0].max
    if rem <= 0
      p.update_attribute('expired', true)
    end
    rem

  end
{% endhighlight %}

Here I set my model so that whenever I try to access the remaining time for the first time, I'll update the model if needed.

I can now update my views. For example in my project partial:

{% highlight ERB %}
<div class="remaining">
  <% if !project.expired %>
    <p ><%= project.remainingTime %> Jours restant</p>
  <% else %>
    <p class="finished">Projet terminé</p>
  <% end %>
</div>
{% endhighlight %}

No need to show the remaining time if the time is over.

I can also add a checkbox to filter my projects index for all projects, or only those not expired.

I'll add the checkbox in my view:

{% highlight ERB %}
<div class="filterrific">
   En cours <%= f.check_box :with_not_expired %> 
</div>
{% endhighlight %}

And update the model:

{% highlight ruby %}
filterrific(
  default_filter_params: {},
  available_filters: [
    :search_query,
    :with_category,
    :with_not_expired
  ]
)

scope :with_not_expired, lambda { |flag|
  if flag == '0'
    all
  else
    where( expired: false || nil)
  end
}
{% endhighlight %}

Much better.

After a bit of css, I also restyled my finished projects so its easier to identify them.

![result](/images/gimmeplz4.png)

Oh, yeah, I forgot to tell about it, but I added a neat progress indicator for my projects.

I wont go into details but I used [jQuery-Knob](https://github.com/aterrien/jQuery-Knob)

Now the last important thing was to have an admin panel, to editing/erasing projects and users, and setting the selection of projects on the front page.

I updated my header to add a few more options:

![result](/images/gimmeplz5.png)

{% highlight ERB %}
<% if user_signed_in? %>
  <li class="dropdown">
    <a href="#" class="dropdown-toggle" data-toggle="dropdown">
      <%= current_user.name %> <b class="caret"></b>
    </a>
    <ul class="dropdown-menu">
      <li><%= link_to "Profile", current_user %></li>
      <li><%= link_to "Editer le profile", edit_user_registration_path %></li>
      <% if current_user.admin? %>
        <li><%= link_to "Admin - Users", users_path %></li>
        <li><%= link_to "Admin - Selection", selection_path %></li>
      <% end %>
      <li class="divider"></li>
      <li>
        <%= link_to "Deconnexion", destroy_user_session_path, method: "delete" %>
      </li>
    </ul>
  </li>
  <li class="round-image-50"><%= image_tag(current_user.avatar.url(:thumb)) %></li>
<% else %>
  <li><%= link_to "Se connecter", new_user_session_path%></li>
<% end %>
{% endhighlight %}

users_path is just the users index that I was not using yet. selection_path is new, so I need to add an action to my controller and a new route:

{% highlight ruby %}
# app/controllers/static_pages_controller.rb

def selection

end

# config/routes.rb

get 'selection', :to => "static_pages#selection"
post 'selection', :to => "static_pages#selection"

{% endhighlight %}

I'm gonna put a form on the selection pages, so it needs to respond to post.


and for the view:

{% highlight ERB %}
# app/view/static_pages/selection.html.erb

<div class="adminSelection">
  <div class="currentSelection">
    Selection actuelle : <% for p in @selection do %>
      <%= p.project_id %>,
    <% end %>
  </div>

  <%= form_tag do %>
    <%= label_tag(:add, "Ajouter") %>
    <%= text_field_tag(:add) %>
    <%= submit_tag("Ok") %>
  <% end %>

  <%= form_tag do %>
    <%= label_tag(:remove, "Supprimer") %>
    <%= text_field_tag(:remove) %>
    <%= submit_tag("Ok") %>
  <% end %>
</div>
{% endhighlight %}

This page is gonna need an access to a list of selections (the projects I want to display on the front page). It will have to fields that will allow to add or remove a project to the selection list.

Now I should have planned that in advance, I know. The best way would have been to create a selection model that references a project (has_many projects).

Because I'm lazy and I didnt want to change my associations, i just created a new Selection model, with only one field : project_id. Whats the problem? because there is no real association there is no dependance when a project is destroyed, so I'll have some extra work to do in the long run. Meh, lets do it the easy way...

My controller will need a few changes:

The home action will need a reference to all the projects in the selection table. Because I was lazy, if I erase a project it doesnt get erased in the selection db, so I have to take an extra step and make sure the project referenced still exists:

{% highlight ruby %}
# app/controllers/static_pages_controller.rb

def home
  @projects = []
  for s in Selection.all do
    @projects << Project.find(s.project_id) if Project.exists?(s.project_id)
  end
end
{% endhighlight %}

Not very elegant...

Then for the selection action:

{% highlight ruby %}
# app/controllers/static_pages_controller.rb

def selection
  @selection = Selection.all
  if !selection_params[:add].nil?
    if Project.exists?(selection_params[:add])
      Selection.create(project_id: selection_params[:add])
    end
  end

  if !selection_params[:remove].nil?
    if Selection.where("project_id = ?", selection_params[:remove]).exists?
      Selection.where("project_id = ?", selection_params[:remove]).take().destroy
    end
  end
end
{% endhighlight %}

So what happens here? I get a list of all the selections that i pass to my view. If I receive a post request (if an admin pressed a button on the selection pages), I check the content of the params. 

If it contains an :add, I add the project th the selection db. If its a :remove, I remove it. There's some extra validation needed but it seems to work.

Oh, and I forgot but the users index view was changed also, to allow for admins to delete profiles, and grant admin privileges:

{% highlight ERB %}
# app/view/users/_user.html.erb

<li>
  <%= image_tag user.avatar.url(:thumb)  %>
  <%= link_to user.name, user %>
  <% if current_user.admin? && current_user != user %>
  <span> ---- </span>
    <%= link_to "Delete", user, method: :delete,
                                  data: { confirm: "You sure?" } %>
                                  <span> - </span>
    <% if user.admin? %>
      <%= link_to "Remove Admin Provileges", {:controller => "users", :action => "removeAdmin", :id => user.id } %>
    <% else %>
      <%= link_to "Add Admin Provileges", {:controller => "users", :action => "addAdmin", :id => user.id } %>
    <% end %>
  <% end %>
</li>
{% endhighlight %}

The result is something like that:

![result](/images/gimmeplz6.png)

Well, that looks quite okish if I might say.

I guess that will have to do for now because I wont have anymore time soon.

So if I could do it again, what would I do differently?

two word: tests and planning. 

I took some time at the beginning to write some tests, but I got frustrated very quickly when I couldn't make it work as well as I wanted, and I told myself, meh, its just gonna be a quick app, nothing fancy, maybe I dont reaaaaally need tests?

WRONG.

YOU NEED TESTS.

As for planning, I know I'm supposed to think in advance about all the stuff I want. I know I will lose hours if I dont. Guess what? I didn't, and I lost hours.

Thanks former self.

And by the way, i wrote almost no comments on my code. But surely that will never be a problem, right?

Here is the [code](https://github.com/AtActionPark) and [result](https://shielded-taiga-9226.herokuapp.com/)
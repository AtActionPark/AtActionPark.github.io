---
layout: post
title: Building a kickstarter clone with rails
---

So a friend of mine recently asked if I would be able to build a kickstarter type site for a project of his. 
<!--more-->
So here it goes. After all those tutorials and project, am I able to create something from scratch, without guidances, or other examples to refer to?

I think as long as I can use google, its gonna be all right...

The first step after generating the project was to take care of the User model.
I went with devise, as I didnt need anything fancy, and just added a user name, and an admin bool.


{% highlight ruby %}
class AddFieldsToUsers < ActiveRecord::Migration
  def change
    add_column :users, :admin, :boolean, :default => false
    add_column :users, :name, :string
  end
end

{% endhighlight %}

I chose paperclip for the avatar, and running 'rails g paperclip user avatar' generated the following migration

{% highlight ruby %}
class AddAttachmentAvatarToUsers < ActiveRecord::Migration
  def self.up
    change_table :users do |t|
      t.attachment :avatar
    end
  end

  def self.down
    remove_attachment :users, :avatar
  end
end
{% endhighlight %}

I also needed to white list those fields. The best solution is to create a registration controller that overrides devise's, and tell devise to use this one:

{% highlight ruby %}
class RegistrationsController < Devise::RegistrationsController

  def sign_up_params
    params.require(:user).permit(:name, :avatar, :email, :picture, :password, :password_confirmation)
  end

  def account_update_params
    params.require(:user).permit(:name, :avatar, :picture, :email, :password, :password_confirmation, :current_password)
  end

  def after_update_path_for(resource)
      user_path(resource)
    end
end
{% endhighlight %}

{% highlight ruby %}
# config/routes.rb
devise_for :users, :controllers => { registrations: 'registrations' }
{% endhighlight %}

To finish with the users avatar I needed to declare the attachment in the User model:

{% highlight ruby %}
#app/model/user.rb
 has_attached_file :avatar, :styles => { :medium => "300x300>", :thumb => "100x100#" },
    :default_url => "/images/:style/missing.png"

  validates_attachment_content_type :avatar, :content_type => /\Aimage\/.*\Z/
{% endhighlight %}

It creates 2 templates for the images, medium and thumb

After migrating and unpacking the devise user views, I could add the name field to registration views. Here is the edit one for reference:

{% highlight ERB %}
<h2>Edit <%= resource_name.to_s.humanize %></h2>

<%= form_for(resource, as: resource_name, url: registration_path(resource_name),html: { method: :put }) do |f| %>
  <%= devise_error_messages! %>


  <span class="avatar field">
    <%= f.label :avatar %><br />
    <%= f.file_field :avatar %>
  </span>

  <div class="field">
    <%= f.label :name %><br />
    <%= f.text_field :name, autofocus: true %>
  </div>

  <div class="field">
    <%= f.label :email %><br />
    <%= f.email_field :email, autofocus: true %>
  </div>

  <% if devise_mapping.confirmable? && resource.pending_reconfirmation? %>
    <div>Currently waiting confirmation for: <%= resource.unconfirmed_email %></div>
  <% end %>

  <div class="field">
    <%= f.label :password %> <i>(leave blank if you don't want to change it)</i><br />
    <%= f.password_field :password, autocomplete: "off" %>
  </div>

  <div class="field">
    <%= f.label :password_confirmation %><br />
    <%= f.password_field :password_confirmation, autocomplete: "off" %>
  </div>

  <div class="field">
    <%= f.label :current_password %> <i>(we need your current password to confirm your changes)</i><br />
    <%= f.password_field :current_password, autocomplete: "off" %>
  </div>

  <div class="actions">
    <%= f.submit "Update" %>
  </div>
<% end %>

<h3>Cancel my account</h3>

<p>Unhappy? <%= button_to "Cancel my account", registration_path(resource_name), data: { confirm: "Are you sure?" }, method: :delete %></p>

<%= link_to "Back", :back %>
{% endhighlight %}




After a bit of tinkering, the users are now functional.

However, and I didnt know that of course, you cant host your images on heroku. Well you can but they get wiped out after each dyno restart. So thats not good...

The solution was to create an amazon web service S3 account and store all files on it. There are a lot of tutorials for that, and when you know what to look, the code is pretty straightforward.
It includes aws-sdk, figaro, and a lot of swearing because you misspelled a variable somewhere, but it ended up working.

So that went well. Now for the projects:


Nothing very complicated went into creating the Project model. I went with more attributes though (name, mainpicture, presentation, objective, descrition and user_id)

Im a bit lazy so I wont go through the whole process, but this is textbook rails generation. Project belongs to users, so you can create them with something like that:
{% highlight ruby %}
def create
    @project = current_user.projects.build(project_params)
    if @project.save
      flash[:success] = "project created!"
      redirect_to root_url
    else
      render :new
    end
  end
{% endhighlight %}


I wanted to be able to user markdown in the project description, so I found the redcarpet gem that allows just that.

You just need to create a helper method like this:
{% highlight ruby %}
 def markdown(text)
  markdown = Redcarpet::Markdown.new(Redcarpet::Render::HTML,
      no_intra_emphasis: true, 
      fenced_code_blocks: true,   
      disable_indented_code_blocks: true)
    return markdown.render(text).html_safe
  end
{% endhighlight %}

that allows you to render stuff like this in your views:
{% highlight ruby %}
<%= markdown(@project.description) %>
{% endhighlight %}

I added disqus comments to my project show pages, after registering my site, and called it a day.

For now.


Here is the [code](https://github.com/AtActionPark) and [result](https://shielded-taiga-9226.herokuapp.com/)
---
layout: post
title: Building a kickstarter clone with rails part 3
---

Categories and search engine
<!--more-->

I decided that I wanted to add categories to my project, and that I wanted to be able to sort the project index by categories. Oh, and while I'm at it, why not add a search bar! I'm sure I wont get depressed if I fail!

So adding the category is easy, its just a matter of adding a field to my db.

Just run `rails g migration  AddCategoryToProjects category:string` and `rake db:migrate`, and as usual, add the field in the new and edit view


{% highlight ERB %}
# _project_form.html.erb

 <%= f.label :'Categorie' %><br>
  <%= f.select :category, Project.options_for_category %>
{% endhighlight %}

I added the options in my projects model:

{% highlight ruby %}
def self.options_for_category
    [
      'Art',
      'BD',
      'Artisanat',
      'Danse',
      'Design',
      'Mode',
      'Cinema & video',
      'Gastronomie',
      'Jeux',
      'Journalisme',
      'Musique',
      'Photographie',
      'Edition',
      'Technologie',
      'Theatre'
    ]
{% endhighlight %}

Done!

Ok so now in order to sort and search, I'm gonna have to rebuild the @projects variable my projects controller passes to my views. 

Should I ... should I try to ... find a gem that does it for me?

YES!

The first one I found was [elasticsearch](https://github.com/elastic/elasticsearch-rails). And I loved it. Easy to use, extremely fast and powerful, what's not to love?

Well, the deployment to heroku wasn't exactly smooth : You need to add the Heroku bonsai add-on (that means having a verified heroku account), set it up, add a elasticsearch rake task, import the records...

I'll be honest, when I saw that it wasn't working immediately, I decided to find an other solution. If I was building a huge site, with a huge database, I would consider it, but for now I dont really need that much power.

So I found [filterrific](https://github.com/jhund/filterrific).

The documentation is pretty much self-explanatory, but lets go through it anyway:

The first step after installing the gem is to make sure I update my model:

{% highlight ruby %}
filterrific(
    default_filter_params: {},
    available_filters: [
      :search_query,
      :with_category
    ]
  )
{% endhighlight %}

I only need 2 filters for now: a search box, and a category filter. 

{% highlight ruby %}
scope :with_category, lambda { |categories|
    where(category: [*categories])
  }

scope :search_query, lambda { |query|
    # Searches query on the 'title' and 'description' columns.
    # Matches using LIKE, automatically appends '%' to each term.
    # LIKE is case INsensitive with MySQL, however it is case
    # sensitive with PostGreSQL. To make it work in both worlds,
    # we downcase everything.
    return nil  if query.blank?

    # condition query, parse into individual keywords
    terms = query.downcase.split(/\s+/)

    # replace "*" with "%" for wildcard searches,
    # append '%', remove duplicate '%'s
    terms = terms.map { |e|
      ('%' + e.gsub('*', '%') + '%').gsub(/%+/, '%')
    }
    # configure number of OR conditions for provision
    # of interpolation arguments. Adjust this if you
    # change the number of OR conditions.
    num_or_conds = 2
    where(
      terms.map { |term|
        "(LOWER(projects.title) LIKE ? OR LOWER(projects.presentation) LIKE ?)"
      }.join(' AND '),
      *terms.map { |e| [e] * num_or_conds }.flatten
    )
  }
{% endhighlight %}

And here is the filter declaration: I followed the examples in the doc. I just changed the second one slightly to search in the whole field.

Im sure the search query is not very efficient with a lot of data, but that will be enough for now I guess

I then added a few line in my project controller:

{% highlight ruby %}
def index
    @filterrific = initialize_filterrific(
    Project,
    params[:filterrific],
    select_options:{
      with_category: Project.options_for_category
    }
    ) or return
    
    @projects = @filterrific.find.page(params[:page])

    respond_to do |format|
      format.html
      format.js
    end
  end
{% endhighlight %}

Here what happens is filterrific is gonna take care of building the new indew based on my query. I can still paginate the results without changing anything so thats pretty cool.

I then only need to change my index view: 

{% highlight ERB %}
<div id="projects" >
<%= form_for_filterrific @filterrific do |f| %>
  <div class="filterrific">
    Search
    <%# give the search field the 'filterrific-periodically-observed' class for live updates %>
    <%= f.text_field(
      :search_query,
      class: 'filterrific-periodically-observed'
    ) %>
  </div>
  <div class="filterrific">
    Category
    <%= f.select(
      :with_category,
      @filterrific.select_options[:with_category],
      { include_blank: '- Any -' }
    ) %>
  </div>

  <div class="filterrific">
    <%= link_to(
      'Reset filters',
      reset_filterrific_url,
    ) %>
  </div>
  <%# add an automated spinner to your form when the list is refreshed %>
  <%= render_filterrific_spinner %>
<% end %>

  <%= render @projects %>
</div>
{% endhighlight %}

Once again the documentation is very good, not a lot to add there.

After a bit of styling, the result is something like that:

![result](/images/gimmeplzSearch.png)

It still needs a bit of polish but it works well enough for now.



Here is the [code](https://github.com/AtActionPark) and [result](https://shielded-taiga-9226.herokuapp.com/)
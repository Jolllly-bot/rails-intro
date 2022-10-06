# Part 1: Filter the list of movies by rating

_Suggestion: Read this whole page before starting to code! There are hints and an overview of the big picture here!_

### Overview
You will need to enhance RottenPotatoes as follows:

1. At the top of the All Movies listing, add some checkboxes that allow the user to filter the list to show only movies with certain MPAA ratings.
2. When the Refresh button is pressed, the list of movies is redisplayed showing only those movies whose ratings were checked. 
3. If _no_ boxes are checked, after a user hits the Refresh button, all movies should be listed, and all checkboxes should be checked. It also applies to the first time when the user visits the page (i.e. when the user visits the page, all checkboxes should be checked).

~~![Screenshot. The filter should be included somewhere below the page heading. It should have a checkbox for each rating, followed by a "Refresh" button.]
(https://apollo-media.codio.com/media%2F1%2F222744519f1bdd7b747b690d8d2ce76b-ecf42aa98fcb72b9.webp)~~
**TODO: A (or link to the) video demo of how RottenPotatoes will respond after these  enhancements are done will be more helpful**

This will require a couple of pieces of code in your Model, View and Controller.

### View
We have provided the code that generates the checkboxes form, which you can include in the `index.html.erb` template.

```erb
<%= form_tag movies_path, method: :get, id: 'ratings_form' do %>
  Include:
  <% @all_ratings.each do |rating| %>
    <div class="form-check  form-check-inline">
      <%= label_tag "ratings[#{rating}]", rating, class: 'form-check-label' %>
      <%= check_box_tag "ratings[#{rating}]", "1",  @ratings_to_show.include?(rating), class: 'form-check-input' %>
    </div>
  <% end %>
  <%= submit_tag 'Refresh', id: 'ratings_submit', class: 'btn btn-primary' %>
<% end %>
```

#### IMPORTANT for grading purposes
- The `id` attributes on some of the elements are required for the autograder to work, so don’t change/delete them.
- As in the above code, your form tag should have the id `ratings_form` and the form submit button for filtering by ratings should have the id `ratings_submit`.
- Each checkbox should have an HTML element id of `ratings_#{rating}`, where the interpolated rating should be the rating itself, such as `ratings_PG-13`, `ratings_G`, and so on.


### Controller
You have to do a bit of work to use the above code:

#### 1. Set up all possible rating values
As you can see, the code in view expects the variable `@all_ratings` to be an enumerable collection of all possible values of a movie rating, such as `['G','PG','PG-13','R']`. The controller action needs to set up this variable. And since the possible values of movie ratings are really the responsibility of the Movie model, it’s best if the controller sets this variable by consulting the Model. 

How to separate the responsibility of Model and Controller?

> A good form would be to create a class method of `Movie` that returns an appropriate value for this collection, say `Movie.all_ratings`, and have the controller assign that to the appropriate instance variable for the view to pick up.

#### 2. Which ratings should be checked?
In the code above, `@ratings_to_show` is assumed to be a collection of which ratings should be checked. The [documentation](https://api.rubyonrails.org/v4.2.11/classes/ActionView/Helpers/FormTagHelper.html#method-i-check_box_tag)for `check_box_tag` says that the third value, evaluated as a Boolean, tells whether the checkbox should be displayed as checked or not.  So `@ratings_to_show.include?('G')` would be true if `'G'` was a member of the collection. The controller action must also set up this array, _even if no check boxes are checked._

 Why must the controller set up a default value for`@ratings_to_show` even if nothing is checked?

> If it doesn't, then @ratings_to_show will have a nil value in the view, and trying to call nil.include? will cause an exception.


You will also need code in the controller that knows:

#### 3. How to figure out which boxes the user checked?
Try viewing the source (right click and click View Page Source) of the movie listings with the checkbox form, and you’ll see that the checkboxes have field names like `ratings[G]`, `ratings[PG]`, etc. This trick will cause Rails to aggregate the values into a single hash called `ratings`, whose keys will be the names of the checked boxes only, and whose values will be the value attribute of the checkbox (which is “1” by default, since we didn’t specify another value when calling the `check_box_tag` helper). 

Using the debugger, take a look at what is in `params[]` when the form is submitted with various checkboxes checked. Particularly, what is in `params[:ratings]` (or `params['ratings']` - special hashes in Rails such as `params` and `session` can be accessed by either strings or symbols, even though hashes in Ruby do not generally behave this way).

If the user checks the G and R boxes, what will the `params[]` be like?

>  `params[]` will include as one of its values 
>  `:ratings=>{"G"=>"1", "R"=>"1"}`

**Hint:** Check out the `Hash` documentation for an easy way to grab just the keys of a hash, since we don’t care about the values in this case. Checkboxes that weren’t checked don’t appear in the `params[]` hash at all.


#### 4.  How to restrict the database query based on that result?
You’ll probably end up replacing `Movie.all` in the controller method. Since most interesting code should go in the model rather than exposing details of the schema to the controller, consider defining a class-level method in the model such as `Movie.with_ratings(ratings)` that takes an array of ratings (e.g. `["r", "pg-13"]`) and returns an ActiveRecord relation of movies whose rating matches (case-insensitively) anything in that array. 
To do its job, this method can make use of `Movie.where`, which has various options to help you restrict the database query.

**Hint:** Read the [Guide](https://guides.rubyonrails.org/active_record_basics.html)and [API](https://api.rubyonrails.org/v4.2.11/classes/ActiveRecord/Base.html)about `ActiveRecord::Base` for examples of how to use `where` to do queries like this. You may also find `.present?` convenient. The [ActiveRecord Intro CHIPS assignment](https://parlorpolo-macroexotic.codio.io/saasbook/hw-activerecord-intro) may be helpful too. 

_We suggest_ adding a class method in `Movie.rb` as follows:
```ruby
class Movie
  def self.with_ratings(ratings_list)
  # if ratings_list is an array such as ['G', 'PG', 'R'], retrieve all
  #  movies with those ratings
  # if ratings_list is nil, retrieve ALL movies
  end
 end
```


Notice that you will need to use the `params[:ratings]` values in two ways in the controller:
1. To determine what values to pass to `Movie.with_ratings`.
2. To set a variable that can be used by the view so that the appropriate checkboxes show up as checked when the filtered view is loaded.

### To pay attention to…

**Labels** You may notice that we have included a HTML `label` alongside the checkbox. These labels are critical for a properly functioning form! Primarily, they tell users what checkbox they are about to select. However, labels also provide built-in accessibility features (such as for blind users, so they too know what each checkbox does) or handy shortcuts like clicking “G” to apply the “G” checkbox. (On a phone, for example, this means users are less likely to tap on the wrong action.)

**Styling** We’ve included some default Bootstrap styling. This is not required to make your form work, but it’s a comming pattern. You are welcome to tweak the CSS classes applied to the form as long as the application works correctly.

**Reminder** Don’t put code in your views! If you find yourself doing computation in your views, set up an instance variable in the controller and do the computation there instead. And if the computation is anything more than trivial, it probably belongs in a model, not in the controller.

### After finishing part1...
You’ll submit this part after you deploy on Heroku and when you supply your Heroku deployment URL in part 3. But you can commit all the changes you have made so far to git, deploy them to Heroku and check that they work on Heroku before moving on to the next section:

```sh
git commit -am "part 1 complete"
git push heroku master
```

**NOTE!** Be sure that you have used `git add` to add all the new files you’ve created! 
If in doubt, use `git status` to show which files Git thinks it does not know about. 

A common pitfall is forgetting to add some files, and then when the app is deployed to Heroku, it fails because some files are missing.
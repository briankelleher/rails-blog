## How this was developed.

Isn't this nice?? The developer giving you his exact step by step instructions for creating this blog from scratch!

## First Steps - Scaffolding the project

You will need ruby installed on your machine.  You will want to install the bundler gem and the rails gem.

I recommend using [RVM](https://rvm.io/).

```bash
# to install bundler and rails
gem install bundler
gem install rails
```

Running the following command will create a rails project in a new directory named [project-name].

```ruby
rails new [project-name]
cd [project-name]
```

## Users + Authentication

The main portion of the blog revolves around posts, but user authentication in some cases can be rather complex and be involved in most models in your application.  I usually like to start with authentication in this case, so you can build proper user-model relationships during development.

Add `gem 'devise'` to your Gemfile.  Then you will want to install it:

```bash
bundle install
rails generate devise:install
```

In a production-level application you will want to take a look at `config/initializers/devise.rb` for settings that suit the needs of your application.

```bash
# This will create a User model, already set up with devise default routes.
rails generate devise User
```

Now is the time to decide on some options for your blog users.  You can make changes to the user model by editing `app/models/user.rb`.  Read the [devise github](https://github.com/plataformatec/devise) to add/remove options like `:registerable`.

For the purposes of this app, we will want to remove `:registerable`, and keep the rest of the default options.

Next, run `rails db:migrate`.  This will add users to the database.

## Creating the posts

We are going to operate under the assumption that the basic post model will need a title, and body text.  All other information about the post should be relational in our simple example.

```
rails g scaffold Post title:string body:text
```

The above will create a Post controller, Post routes for showing, creating, editing, and deleting, routes for showing all and single posts in JSON, a coffee script file and a scss file that are automatically added to the build pipeline.  Personally I do not use CoffeeScript, so I safely delete this file.

The scaffold also adds a migration file for us, with timestamps added for `created_at` and `updated_at`.

Now, we run `rails db:migrate`, and our model is looking good!

Lastly, add to your `config/routes.rb` the line `root "posts#index"`.  This will route the home route of your app to the posts controller, index command, and thus render the posts index homepage.

Lets test this thing.  Run `rails s` to start a rails local server, and open `http://localhost:3000` in your browser to view your app.  The basic posts index should render.

## Seed our users, and add a post-user relationship.

Our users are the only ones that are able to create posts.  Users cannot sign up to be on this blog, as this blog will be admin-only.  As a result, we just need to lock down the update, create, destroy, and new commands to be admin-only.  With devise, this is easy.  Simply add this to the top of  `app/controllers/posts_controller.rb`:

```ruby
before_action :authenticate_user!, except: [:index, :show]
```

We also need users in our database to test this, as you cannot sign up.  Open up `db/seeds.rb`.  Add the following:

```ruby
User.create(
    email: "test@example.com",
    password: "password"
)
```

Running `db:seed` will create a basic test user, so you can log in and test user relationships.

Next, we will want to add the user-post author relationship.  We first have to set up a database migration to add this relationship:

```bash
rails g migration AddUserToPosts user:references
```

This will automatically add the foreign key relationship between posts and users in a migration file.  Just run `rails db:migrate`, and this will be reflected in the database.

We will want to make a slight adjustment to the posts controller in order for this to work well.  Swap out the first line of the `create` function to read:

```ruby
@post = current_user.posts.build(post_params)
```

Instead of flat-out creating a post, this will create a post in the context of the current user (`current_user` is a built in function referencing the user currently signed in.  Since we have already set authentication up on our posts, we have guaranteed a user is logged in when they are creating a post).

In order for the above to work, you will need to add a line to the User model, at `app/models/user.rb`.  Add `has_many :posts`.  This will add a backwards user-post relationship, so one could easily query for a user's posts, as well as create them for a user.

## Categories!

Once this blog gets huge and successful, you will need some categories so users can easily see relevant posts!  This will also help your sanity in post management.

Lets scaffold out some categories.

```bash
rails g scaffold category name:string description:text
```

Before you run a migration, add the following to the most recently added migration file (in `db/migrate`):

```ruby
add_reference :posts, :category, foreign_key: true
```

Then, a simple `rails db:migrate` will update the database schema to support categories.

To finish the category-post relationship add `has_many :posts` to `app/models/category.rb`.

Run a `rails s` and open `http://localhost:3000/categories` to see the category index.  Go ahead and create a category, and we will edit the post form to have a category (`app/views/posts/_form.html.erb`).  Add the following to the post form view before the submit button:

```html
<div class="field">
    <%= f.label :category_id %>
    <%= f.collection_select :category_id, Category.all, :id, :name %>
</div>
```



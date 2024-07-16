# Task Manager 7

## CRUD in Rails

What does CRUD (in the the programming world) stand for?

- C: Create
- R: Read
- U: Update
- D: Delete

The apps that you will be creating in Mod 2 will make heavy use of these four actions.

Let's use Rails to build an application where we can manage some tasks.

TODO: Add an image very similar to this but without the view portion: https://backend.turing.edu/module2/lessons/images/mvc_rails.png

TODO: Update MVC wording for an API returning JSON data. Maybe use a page like this to learn more about JSON? https://backend.turing.edu/module3/notes/json_for_api_development.html

Throughout the module, we'll talk through some conventions and best practices, but for now - we'd like for you to follow along with this tutorial. We highly recommend **not** copying and pasting the code in this tutorial. It's to your advantage to type each line of code on your own.

## Getting Configured

Before creating our new Task Manager app, let's make sure we are all on the same version of Rails. For this tutorial, you will want to be running Rails 7.1.2. To check which version of rails you have installed, run `$ rails -v`. If you see any version other than 7.1.2, you will need to follow [these instructions](./rails_uninstall.md) to get the correct version installed.

After confirming that you are running the correct version of rails, we are ready to get started!

To create your rails app, navigate to your 2module directory and run the following command:

`$ rails new task_manager -T -d="postgresql" --api`

Let's break down this command to better understand what is happening. `rails new` is the command to create a new Rails app - this will create *a lot* of directories and files; we will explore this file structure in a moment. `task_manager` is going to be the name of the directory in which our Rails app lives and is, essentially, the name of our application. `-T` tells Rails that we are not going to use its default testing suite. We will be using RSpec as our testing framework. `-d="postgresql"` tells Rails that we will be using a postgresql database; Rails can handle many different databases, but this is the one we will be using in Mod 2. `--api` tells rails that we want to build an API instead of an application that also has a frontend.

Now that we have created our rails app, let's `cd task_manager` and explore some of the structure that Rails builds out for us.

## Project Folder Structure

![Untitled](./images/rails_7_file_structure.png)

Taking a look at the files that rails creates can be a bit overwhelming at first, but don't worry - this tutorial will only touch on a handful of directories! The top level directories that we are concered with are:

- *app* - This is where we configure Models and Controllers.
- *config* - Inside this directory, in the routes.rb file is where we will tell our Rails app which HTTP requests to respond to.
- *db* - Where our database structure will be set up.

In addition to these directories, we will also be dealing with our Gemfile, which is where we will tell Rails about any other gems we might need to run our app. For our task manager we will be adding just one gem to our Gemfile. Open your gemfile and add `pry` to the `:development, :test` group - your Gemfile should now look like this:

```ruby
source "https://rubygems.org"
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby "3.2.2"

# Bundle edge Rails instead: gem "rails", github: "rails/rails", branch: "main"
gem "rails", "~> 7.1.2"

# The original asset pipeline for Rails [https://github.com/rails/sprockets-rails]
gem "sprockets-rails"

# Use postgresql as the database for Active Record
gem "pg", "~> 1.1"

# Use the Puma web server [https://github.com/puma/puma]
gem "puma", "~> 6.0"

# Use JavaScript with ESM import maps [https://github.com/rails/importmap-rails]
gem "importmap-rails"

# Hotwire's SPA-like page accelerator [https://turbo.hotwired.dev]
gem "turbo-rails"

# Hotwire's modest JavaScript framework [https://stimulus.hotwired.dev]
gem "stimulus-rails"

# Build JSON APIs with ease [https://github.com/rails/jbuilder]
gem "jbuilder"

# Use Redis adapter to run Action Cable in production
# gem "redis", "~> 4.0"

# Use Kredis to get higher-level data types in Redis [https://github.com/rails/kredis]
# gem "kredis"

# Use Active Model has_secure_password [https://guides.rubyonrails.org/active_model_basics.html#securepassword]
# gem "bcrypt", "~> 3.1.7"

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem "tzinfo-data", platforms: %i[ mingw mswin x64_mingw jruby ]

# Reduces boot times through caching; required in config/boot.rb
gem "bootsnap", require: false

# Use Sass to process CSS
# gem "sassc-rails"

# Use Active Storage variants [https://guides.rubyonrails.org/active_storage_overview.html#transforming-images]
# gem "image_processing", "~> 1.2"

group :development, :test do
  # See https://guides.rubyonrails.org/debugging_rails_applications.html#debugging-with-the-debug-gem
  gem "debug", platforms: %i[ mri mingw x64_mingw ]
  gem "pry"
end

group :development do
  # Use console on exceptions pages [https://github.com/rails/web-console]
  gem "web-console"

  # Add speed badges [https://github.com/MiniProfiler/rack-mini-profiler]
  # gem "rack-mini-profiler"

  # Speed up commands on slow machines / big apps [https://github.com/rails/spring]
  # gem "spring"
end
```

Any time we update our Gemfile, we will need to tell our application to install or update the additional gems. In your terminal, run the command:

```bash
$ bundle install
```

Great - now we can use `binding.pry` anywhere in our app to debug as we go!

## Database Set-Up

Before we can see what our new rails app can do, we need to do some more set up. First, let's create our app's database. Do this from the command line with:

```bash
$ rails db:create
```

You should see some output like this:

```bash
Created database 'task_manager_development'
Created database 'task_manager_test'
```

We have created two databases, a "development" and a "test" database. In this tutorial, we won't be writing any tests so we won't touch our test database. We are working in the "development" environment.

Great, now our database is created! But, we don't have any tables yet.

### Migrating to our Database

We could potentially create a tasks table directly from our terminal, but that's no fun, and it makes it pretty difficult to work with other people. Instead, let's create some migrations.

Migrations allow you to evolve your database structure over time. Each migration includes instructions to make some change to the database (e.g. adding a table, adding columns to a table, dropping a table, etc.). One advantage to this approach is that it will allow you to transfer the application to different computers without transferring the whole database. This isn't a big problem for us now, but as your database grows it will be advantageous to be able to transfer the instructions to create the database instead of the database itself.

To create a migration that will send instructions to create a tasks table to our database, run the following command from your terminal:

```bash
$ rails generate migration CreateTask title:string description:string
```

In this command, we are telling rails to generate a migration file that will create a tasks table in our database with two columns - title and description. To see the migration that rails created, open your `db/migrate` directory, and you should have a file in there that is called something like `db/migrate/20190414173402_create_task.rb`. Open that file and you will see the following:

```ruby
class CreateTask < ActiveRecord::Migration[7.0]
  def change
    create_table :tasks do |t|
      t.string :title
      t.string :description

      t.timestamps
    end
  end
end
```

So, now we have a migration with some instructions to tell our database to create a tasks table, but how do we actually get the table created? Run the following in your terminal:

```bash
$ rails db:migrate
```

And you should see something like this:

```bash
== 20221130062449 CreateTask: migrating =======================================
-- create_table(:tasks)
   -> 0.0052s
== 20221130062449 CreateTask: migrated (0.0053s) ==============================
```

Great! How can we verify that worked?

In your terminal, connect to the database that we just created, and see if we can select some information from our tasks table:

```bash
$ rails dbconsole
```

```bash
psql (14.2)
Type "help" for help.

task_manager_development=# SELECT * FROM tasks;
 id | title | description | created_at | updated_at
----+-------+-------------+------------+------------
(0 rows)

task_manager_development=#
```

Awesome - we have a database with a table for tasks! In order to test your API, you'll want to add at least two tasks to your database. Review your SQL practice from intermission to learn how to add some tasks. 

If you're still stuck, this is a great time to use ChatGPT to help you generate some data. Try asking something like 
```
How can I write SQL to insert data into this table?

id | title | description | created_at | updated_at
```

To exit the psql session, enter the command `exit`

## Getting the App Running

Now that we've set up our database, it's time to get our server up and running and ready for HTTP requests. To do this, run either

`rails server` or `rails s`

You should see something like this:

```bash
=> Booting Puma
=> Rails 7.1.2 application starting in development
=> Run `bin/rails server --help` for more startup options
Puma starting in single mode...
* Puma version: 5.6.5 (ruby 3.2.2-p18) ("Birdie's Version")
*  Min threads: 5
*  Max threads: 5
*  Environment: development
*          PID: 66200
* Listening on http://127.0.0.1:3000
* Listening on http://[::1]:3000
Use Ctrl-C to stop
```

Navigate to [http://localhost:3000/](http://localhost:3000/) and you should see some Rails magic!

Now, let's take a look back at your terminal and walk through what just happened.

```bash
Started GET "/" for ::1 at 2022-11-29 18:51:51 -0700
Processing by Rails::WelcomeController#index as HTML
  Rendering /Users/mdao/.rbenv/versions/3.2.2/lib/ruby/gems/3.1.0/gems/railties-7.0.5/lib/rails/templates/rails/welcome/index.html.erb
  Rendered /Users/mdao/.rbenv/versions/3.2.2/lib/ruby/gems/3.1.0/gems/railties-7.0.5/lib/rails/templates/rails/welcome/index.html.erb (Duration: 1.4ms | Allocations: 635)
Completed 200 OK in 13ms (Views: 4.9ms | ActiveRecord: 0.0ms | Allocations: 5094)
```

On the first line, you are seeing a snapshot of the HTTP request that was received by our server when we navigated to localhost:3000 - `GET "/"`. This is basically telling us that our application received a request for the information that lives at a certain address.

On the last line, we can see a snapshot of the response that our server sent back to the browser `Completed 200 OK`. Meaning, we received a request, were able to process that request and succesfully send back a response. The response is what contains the information that allows the browser to render the default page for your Rails app!

## Returning All Tasks

### Adding the Route

Let's update our app to return something other than the default Rails welcome page - remember our goal is to send back JSON data that a frontend application could use.

Here's our goal. When our browser makes a GET request to `localhost:3000/tasks`, we want our API to return all of the tasks currently in the database.

We want to see something like this:
<!-- Insert screenshot here -->

Try navigating to `localhost:3000/tasks` in your own browser. You should see an error telling you that `No route matches [GET] "/tasks"`. Our app received a request that it was not set up to handle, so let's change that!

First we need to update our `config/routes.rb` to handle a `GET '/tasks'` request. Update your routes.rb file to include the following:

**config/routes.rb**

```ruby
Rails.application.routes.draw do
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  # Defines the root path route ("/")
  # root "articles#index"

  get "/tasks", to: "tasks#index"
end
```

This line that we added is telling our application that anytime we receive an HTTP GET request for the URI '/tasks', we should perform the index action within our tasks controller.

Go back to your browser and refresh the page - you should now be seeing an error telling you that you have a `Routing Error`; specifically, that you have an `uninitialized constant TasksController`. This is a good thing; at this point, we have told our application which controller action to perform when this request is received, but we haven't created that controller (or the action) yet. So, let's go do that.

### Adding the Controller

Open your `app/controllers` directory and create a file inside called `tasks_controller.rb`. In that file, add the following code:

**app/controllers/tasks_controller.rb**

```ruby
class TasksController < ApplicationController
  def index
    render json: {
      example_key: {
        example_nested_key: "It worked!"
      },
      example_array_key: [
        "Array item 1", "Array item 2"
      ]
    }
  end
end
```

Go back to your browser and refresh the page. You should now see the json data you added show up in your browser! Try changing the keys and values of the data that's returned.

We're getting close! The last thing we need to do is return the tasks stored in our database instead of this random json. Following MVC conventions, we're going to create a Model to help us do this.

### Creating our Model

In order to follow MVC conventions, we are going to create this Task class in a `app/models/task.rb` file. In that file, create a Task class that looks like this:

**app/models/task.rb**

```ruby
class Task < ApplicationRecord

end
```

Why inherit from `ApplicationRecord`? This Task class that we are creating is meant to be a very specific and specialized class that we refer to as a Model. The purpose of Models is to create objects based on records that exist in a database. Rails give us some methods that we can use to help in this creation and we inherit those methods from `ApplicationRecord`. Some of the methods that we inherit are `#all`, `::new` and `#save`. These methods are inherited even when our class is totally empty. We're going to make use of the `all` method first.

### Returning Tasks from the Database

Update your tasks_controller to match the following. Rails will pull all of the Tasks from our database and then `render json:` will return that data as json.

**app/controllers/tasks_controller.rb**

```ruby
class TasksController < ApplicationController
  def index
    render json: Task.all
  end
end
```

Refresh your browser and you should now see when you make a request to `localhost:3000/tasks` you are returned all of the tasks as json. Success!

## Returning One Task

The next endpoint we need to create is a `GET` request to https://localhost:3000/tasks/:id, which should return the task with the ID specified. For example, a `GET` request to `localhost:3000/tasks/2` would return the task with id 2.

If you make the request https://localhost:3000/tasks/2 in your browser. You should see the error `No route matches [GET] "/tasks/2"`. This is because we haven't added this route. Update your routes.rb file to include the following:

**config/routes.rb**

```ruby
Rails.application.routes.draw do
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  # Defines the root path route ("/")
  # root "articles#index"

  get "/tasks", to: "tasks#index"
  get "/tasks/:id", to: "tasks#show"
end
```

Now let's add that show route into our tasks_controller:

**app/controllers/tasks_controller.rb**

```ruby
class TasksController < ApplicationController
  def index
    render json: Task.all
  end

  def show

  end
end
```

Remember when we talked about how we are inheriting some functionality from ActiveRecord that helps us interact with our database? Well, now's a great place to take advantage of the ActiveRecord method find, which will retrieve a record from our database based on that record's id. In this case, we can use find like this:

**app/controllers/tasks_controller.rb**

```ruby
def show
  render json: Task.find(params[:id])
end
```

Navigate to http://localhost:3000/tasks/1 and you should now see the json for that particular task!


What is this `params` that we are passing into the find method? Let's throw a binding.pry at the top of our show action and see what we can find:

```ruby
def show
  binding.pry
  render json: Task.find(params[:id])
end
```

Refresh your browser and take a look at your terminal. In your pry session, call params and see what is returned.

```
 13: def show
 => 14:   binding.pry
    15:   render json: Task.find(params[:id])
    16: end

[1] pry(#<TasksController>)> params
=> #<ActionController::Parameters {"controller"=>"tasks", "action"=>"show", "id"=>"2"} permitted: false>
```

We are getting a params object that includes an :id that matches with the very end of the uri we visited (/tasks/2). Looking at our routes, we see that we set up our URI pattern to accept :id, but when we visited this site, we typed in 2 which is an actual id that exists in our database. When we need to get some information, like an id, from our route in the form of parameters, we can include a symbol of the thing we are expecting when we set up our route - in this case, we are expecting an :id. So, based on how we set up our routes, we can manipulate and dictate what parameters we want; and what information we will need access to in our controllers.

Remember that you will need to exit your pry session to continue interacting with your site!

## Requests in Postman

Open up [Postman](https://www.postman.com/downloads/) which you should have installed over intermission.

In postman, make the same two requests you have been making in your browser.

GET /localhost:3000/tasks
GET /localhost:3000/tasks/2

<!-- ADD postman screenshot here -->

While you can make GET requests in your browser, it's often easier to see what's happening in Postman. 

Additionally, we're now going to build POST, PUT and DELETE endpoints and we need an additional tool such as Postman to make those types of requests.

## Adding New Tasks

Our goal is to support all of the CRUD operations in our API. We have successfully build out the Read portion, now it's time to build a route for Creating a new task.

Let's set up the request in Postman first. To create a new task, you're making a `POST` request to the `/tasks` endpoint and passing in the data for that new task in your request body. 

<!-- TODO, insert image of request in postman here -->

To build out this endpoint, we will follow the same steps as we have been. 1. Add the route. 2. Update the controller.

**config/routes.rb**

```ruby
Rails.application.routes.draw do
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  # Defines the root path route ("/")
  # root "articles#index"

  get "/tasks", to: "tasks#index"
  get "/tasks/:id", to: "tasks#show"
  post "/tasks", to: "tasks#create"
end
```

And then in our tasks controller:

**app/controllers/tasks_controller.rb**

```ruby
class TasksController < ApplicationController
  def index
    render json: Task.All
  end

  def show
    render json: Task.find(params[:id])
  end

  def create
    task = Task.new({
      title: params[:title],
      description: params[:description]
    })
  
    if task.save
      render json: task, status: :created
    else
      render json: task.errors, status: :unprocessable_entity
    end
  end
end
```
<!-- TODO: do we want this error handling. Sort of seems like we want error handling of some sort here? -->

Now that we have our create endpoint set up, let's put three binding.pry's in our tasks controller's create action so that it looks like this:

**app/controllers/tasks_controller.rb**

```ruby
def create
    binding.pry
    task = Task.new({
      title: params[:title],
      description: params[:description]
    })
    binding.pry
    if task.save
      binding.pry
      render json: task, status: :created
    else
      render json: task.errors, status: :unprocessable_entity
    end
  end
```
Make a POST request to http://localhost:3000/tasks/create in Postman. In our terminal, we should have hit our first pry - let's make sure we have access to the information we need to create a new task:

```
 10: def create
 => 11:   binding.pry
    12:   task = Task.new({
    13:     title: params[:title],
    14:     description: params[:description]
    15:   })
    16:   binding.pry
    17:   if task.save
    18:     binding.pry
    19:     render json: task, status: :created
    20:   else
    21:     render json: task.errors, status: :unprocessable_entity
    22:   end
    23: end

[1] pry(#<TasksController>)> params
=> #<ActionController::Parameters {"title"=>"Attend Meeting", "description"=>"Participate in the team meeting at 10 AM", "controller"=>"tasks", "action"=>"create", "task"=>{"title"=>"Attend Meeting", "description"=>"Participate in the team meeting at 10 AM"}} permitted: false>
```
Looks good, let's `exit` to hit our next pry and see what `task` looks like at this point:

```

    10: def create
    11:   binding.pry
    12:   task = Task.new({
    13:     title: params[:title],
    14:     description: params[:description]
    15:   })
 => 16:   binding.pry
    17:   if task.save
    18:     binding.pry
    19:     render json: task, status: :created
    20:   else
    21:     render json: task.errors, status: :unprocessable_entity
    22:   end
    23: end

[1] pry(#<TasksController>)> task
=> #<Task:0x000000010b8b07e8
 id: nil,
 title: "Attend Meeting",
 description: "Participate in the team meeting at 10 AM",
 created_at: nil,
 updated_at: nil>
```

We have a Task object! We can see that it has a title and a description, but no id; this is telling us that Rails was able to create the object, but has not yet saved it in the database. Let's exit to hit our next pry and see if anything changes.

```
    10: def create
    11:   binding.pry
    12:   task = Task.new({
    13:     title: params[:title],
    14:     description: params[:description]
    15:   })
    16:   binding.pry
    17:   if task.save
 => 18:     binding.pry
    19:     render json: task, status: :created
    20:   else
    21:     render json: task.errors, status: :unprocessable_entity
    22:   end
    23: end

[1] pry(#<TasksController>)> task
=> #<Task:0x000000010b8b07e8
 id: 5,
 title: "Attend Meeting",
 description: "Participate in the team meeting at 10 AM",
 created_at: Tue, 16 Jul 2024 16:08:03.234819000 UTC +00:00,
 updated_at: Tue, 16 Jul 2024 16:08:03.234819000 UTC +00:00>
 ```

Now, our task has an id, indicating that it was saved in our database! Great, it looks like everything is working, so let's exit through this last pry and go check out the response in Postman.

The response body should look like the following and contain the newly created task:

```json
{
    "id": 5,
    "title": "Attend Meeting",
    "description": "Participate in the team meeting at 10 AM",
    "created_at": "2024-07-16T16:08:03.234Z",
    "updated_at": "2024-07-16T16:08:03.234Z"
}
```

## Editing a Task

At this point, we have an API that will allow us to fetch a list of tasks, fetch a specific task, and add a task. We have covered the Create and Read portions of CRUD. Now, let's add some functionality to be able to Update existing tasks.

We want to support a PATCH request to `/tasks/:id` where the user sends the fields to update in the request body.

### Creating the Route and Controller Action 

In our `config/routes.rb` file, add the following route:

**config/routes.rb**

```ruby
patch '/tasks/:id', to: 'tasks#update'
```

And in our tasks controller, add the following action:

**app/controllers/tasks_controller.rb**

<!-- TODO, this does not work well if you only include title or description it wipes the other out. Not a true patch. -->
```ruby
def update
  task = Task.find(params[:id])
  task.update({
    title: params[:title],
    description: params[:description]
    })
  if task.save
    binding.pry
    render json: task, status: :updated
  else
    render json: task.errors, status: :unprocessable_entity
  end
end
```
<!-- TODO, again do we want error handling and is this the way we want to do it? -->

This is how we do the API update endpoint in the update lesson. Could also do this with no error handling

```ruby
def update
  render json: Book.update(params[:id], book_params)
end
```

There's a lot going on here - let's break it down.

Before we can make any updates to a record in our database, we first need to find that information, which is why we are using the find method that we used on our show action. Then, we can use an ActiveRecord update to change the object that ActiveRecord found and created for us. Finally, we need to save the changes we made to that object in our database. And, thinking way back to when we were creating an object, we want to return the full object to the user as json.

Now that we have this edit functionality implemented, make sure you have your server running, and play around with sending requests in postman to update tasks.

## Deleting a Task

As of now, we have the Create, Read, and Update functions done - all we are missing is Delete!

### Add a Delete Route and Controller Action

Let's go add that route to our `config/routes.rb`:

**config/routes.rb**
```ruby
delete '/tasks/:id', to: 'tasks#destroy'
```

And, with this route, we will need to add a destroy action to our tasks controller:

```ruby
def destroy
    render json: Task.delete(params[:id])
    # TODO: what is the difference between delete and destroy? We do delete in the api tutorial and distroy in the old task manager.
    # I think that delete returns 200 and no respose body, and destroy returns the deleted object, which is weird.
    # I don't like that if you call delete on an id that doesn't exist you still get 200 ok and not an error...
end
```

Another new method! We have used new, save, find, and update so far - what is delete doing? delete is another method we are inheriting from ActiveRecord that deletes records in our database based on an id that you send to the method.

Now, we should be able to send a `DELETE` request to http://localhost:3000/tasks/:id and delete the task for the provided id!


### One Step Further - Serializers

The goal for this tutorial is to walk you through the full process of building an API that supports the fully CRUD actions. We have successfully done that and we could stop here!

But often times we will want to customize our response a little bit more instead of sending back the full object. For us to accomplish this, we are going to use something called a Serializer.

```
$ mkdir -p app/serializers
$ touch app/serializers/task_serializer.rb
```

<!-- TODO, any ideas for something more real world to do with this serializer besides not return the description? -->

**app/serializers/book_serializer.rb**

```ruby
class TaskSerializer
  def self.format_tasks(tasks)
    tasks.map do |task|
      {
        id: task.id,
        title: task.title,
        created_at: task.created_at,
        updated_at: task.updated_at,
      }
    end
  end
end
```

Now that we have a serializer that formats our tasks for our json response we can use it in our controller.

**app/controllers/api/v2/books_controller.rb**

```ruby
def index
  tasks = Task.all
  render json: TaskSerializer.format_tasks(tasks)
end
```
Make a GET request to retrieve all tasks again and we should see the new formatting! If you are still curious about serializers look ahead to the serializers lesson and do a little research.

## Finished!

Congrats! You have finished your first Rails api that can handle full CRUD functionality for a database resource! We can now Create, Read, Update, and Delete tasks! And you are now familiar with the concept of a serializer to customize how your format your Json response.

## Checks for Understanding

1. Define CRUD.
1. Define MVC.
1. What two files would you need to create/modify for a Rails application to respond to a GET request to /tasks, assuming you have a Task model.
1. What are params? Where do they come from?
1. What is the purpose of a serializer?
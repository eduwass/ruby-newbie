# :gem: Ruby newbie :gem:
My own collection of TL;DR quickstart notes for starting Ruby on Rails + Heroku development.

Disclaimer: I made this notes while using a  MacBook Pro (Retina, 13-inch, Mid 2014) with Yosemite,
and my software background is mainly being a LAMP backend dev. This is a quickstart guide I made for my own learning purposes, so this might not work for you, but if it does, buy me a beer or something.

Initial Setup
------

This assumes you already have either a Heroku app, or some other app in a Bitbucket/Github repo.

##### a) Cloning a Heroku app repo to your local dev env:

Heroku will provide its own git servers, so you don't necesarily need another git repo.

1. Install the [Heroku Mac Toolbelt](https://toolbelt.heroku.com/download/osx)
2. Login wiht your Heroku creds: `heroku login`
3. Go ahead and clone your app: `heroku git:clone -a APPNAME`
4. You can now deploy to Heroku like this: `git push heroku master`

##### b) Import your existing Bitbucket/Github app to Heroku

If you already have your app in another git repo, this is probably what you want.

1. Create an entry for the app in heroku (assuming its not there already)
2. Add the 'heroku' remote to your git repo: `heroku git:remote -a APPNAME` (you can check if the new remote was added succesfully using `git remote -v)
3. You can now deploy to Heroku like this: `git push heroku master`

#### Install dependencies, setup the DB and run it!

###### Install those deps
5. cd inside the app and bundle install its dependencies: `bundle install` ( uh-oh... did your [pg gem fail](#pg-gem-fail)? )

###### Using a DB? Set it up to date!

Initially you can just run: `rake db:setup` (this will trigger db:create, db:schema:load, db:seed)

But if you want it to be just like Production, it's likely you'd rather [get a real DB dump from Production](#get-prod-db), though.

###### Run it

From inside the apps dir run: `rails server`
If nothing fails you'll have a running webapp in http://localhost:3000
That's just the default URL and port, could be different depending on your app.

DB Ops
------
#### Get a Heroku/Production DB dump
$ heroku pg:backups capture
$ curl -o latest.dump `heroku pg:backups public-url`
Import Prod DB to Dev DB:

#### Syncing DBs between Development & Heroku/Production
##### From Production to Development
😊 This is the standard procedure to get your dev DB up to date.

-- to do --

##### From Development to Production
😱This will *upload your local DB to Production*, you sure you wanna do this?

-- to do --


Directory Structure
------

path | what it is
------------- | -------------
`./app` | 90% of your application code goes here. Your models, views, controllers, mailers, and workers (if you're using Resque) are organized inside app. Feel free to make your own subdirectories.
`./bin` | Where binstubs -- gem executables wrapped for use in your app's environment -- live
`./config` |  Set up your database, your routes, deployment environments, i18n localization here. For all things related to configuring your app.
`./config.ru` | This is the file that links Rack with Rails.
`./db` |  Your migrations, your schema.rb, and, if you're using SQLite, your development.sqlite database
`./Gemfile` and `./Gemfile.lock` |  Used by bundler to know what gems your app needs
`./lib` | A wasteland that's appended onto the load path. Except for custom rake tasks in `./lib`/tasks, I've never seen code in lib that shouldn't be somewhere else. Don't put code here.
`./log` | Where your log files go
`./public` |  Anything put here will be available to the world as http://www.myapp.com/filename. Things like 500.html go here, because this directory still works when your app crashes. You can also serve out static assets, like CSS files or images. Feel free to make subdirectories.
`./Rakefile` |  The Rakefile is responsible for defining all of those basic Rails tasks, like db:migrate, when you call rake from the command line. The only justified reason I've seen for munging this file is when you want to load in a limited subset of Rails (such as for Resque).
`./README.rdoc` | The most important file in a Rails app. Use it to document how your app works, how to run it in development, etc.
`./test` |  The most useless folder in Rails. 99% of the time, testing is for chumps. Use your limited time in life to do something meaningful.
`./tmp` | A toiletbowl for precompiled assets, pid files, and session files. Don't put anything permanent here, because the toilet flushes.
`./vendor` |  3rd party code, like gems not managed by bundle or CSS libraries

Models - Active Record
------
Active Record is Ruby's Model definition system.
It abstracts all database operations for us.

This is a very short intro, check the official [ActiveRecord Manual](http://guides.rubyonrails.org/active_record_basics.html) for more info

Also check out this [great TL;DR gist about Rails Models by rstacruz](https://gist.github.com/rstacruz/1569572) 

#### Data types:
* `:binary`
* `:boolean`
* `:date`
* `:datetime`
* `:decimal`
* `:float`
* `:integer`
* `:primary_key`
* `:references`
* `:string`
* `:text`
* `:time`
* `:timestamp`

#### Associations

Associations are the way in which we relate Models in Active Record.
How exactly does this look? Well, when you define your Models, you'd do something like this:
```ruby
class Customer < ActiveRecord::Base
  has_many :orders, dependent: :destroy
end
 
class Order < ActiveRecord::Base
  belongs_to :customer
end
```

These are the possible **Association Types**:
* `belongs_to`
* `has_one`
* `has_many`
* `has_many :through`
* `has_one :through`
* `has_and_belongs_to_many`

This is very well explained in [The Types of Associations (Docs)](http://guides.rubyonrails.org/association_basics.html#the-has-many-through-association), you should read it before you start modelling.

#### Migrations
Migrations are a convenient way to alter your database schema over time in a consistent and easy way.
You usually generate them using `rails g migration` and run them with `rake db:migrate`, take a look at the next two sections ( [Useful command](#useful-commands) and [Generators](#generators) ) to see some simple examples.

#### Query Interface
So if we don't have access to SQL, how do we query stuff? ActiveRecord provides methods for this, check the [docs](http://guides.rubyonrails.org/active_record_querying.html) for details.

But here are some simple examples to get an overview:

Ruby Expression | SQL/Human equivalent
------------- | -------------
`client = Client.find(10)` | `SELECT * FROM clients WHERE (clients.id = 10) LIMIT 1`
`client = Client.take` | `SELECT * FROM clients LIMIT 1`
`client = Client.first` | `SELECT * FROM clients ORDER BY clients.id ASC LIMIT 1`
`client = Client.last` | `SELECT * FROM clients ORDER BY clients.id DESC LIMIT 1`
`Client.find_by first_name: 'Lifo'   or   Client.where(first_name: 'Lifo').take` | `SELECT * FROM clients WHERE first_name='Lifo'`
`client = Client.find([1, 10])` | ` SELECT * FROM clients WHERE (clients.id IN (1,10)) `
`Client.last(2)` | ` SELECT * FROM clients ORDER BY clients.id DESC LIMIT 2 `
`Client.where("orders_count = ?", params[:orders])` | `SELECT * from clients WHERE orders_count = params[:orders]`
`Client.where("last_name LIKE :l_name", {:l_name => "%#{l_name_var}%"})` | ` SELECT * FROM client WHERE last_name LIKE '%l_name_var%'`
`Client.order(created_at: :desc)` | ` SELECT * FROM clients ORDER BY created_at DESC`
`Client.where('id > 10').limit(20).order('id desc')` | ` SELECT * FROM articles WHERE id > 10 ORDER BY id asc LIMIT 20` 
`Client.joins('LEFT OUTER JOIN addresses ON addresses.client_id = clients.id')` | ` SELECT clients.* FROM clients LEFT OUTER JOIN addresses ON addresses.client_id = clients.id` 
`Client.count` | `SELECT count(*) AS count_all FROM clients`
`Client.average("orders_count")` | `SELECT AVG(orders_count) FROM clients`

**Loop over query**

```ruby
Client.find_each do |client|
  # ...
end
```


Useful commands
------
* `rails new foo` Creates a new Rails applications in subdirectory foo.
* `rails server` Starts up a Web server on port 3000 running the current application; log messages from the server will appear on standard output.
* `rails console` Starts up a Rails application in interactive mode (you can type commands on the console); doesn't actually start a Web server. This command is convenient if you want to type commands interactively to test their behavior.
* `rails destroy model foo` Deletes all of the files created by the rails generate model foo command above. The command has similar forms to match all of the other rails generate commands. This command is useful if you mistype a rails generate command.
* `rake db:migrate` Runs all migrations to bring the database up to date.
* `rake db:reset` Drops the database and creates a new one (does not run any migrations on the new database).
* `rake db:migrate:reset` Drops the database, creates a new one, and runs all migrations to bring the database up to date.
* `rake db:migrate VERSION=20090909201633_create_photos.rb` Runs migrations (either forward or backward) to restore the database to match the state just after the given migration.
* `bundle update` If you modify the Gemfile in a project in order to include new or different Ruby Gems, this command will update all of your Gems to match the Gemfile.
* `gem update` Updates all Ruby Gems to their latest versions.
* `gem update rails--include-dependencies` Updates Rails to the latest version, including all Gems that Rails depends upon.
* `gem update --system` Updates Ruby to the latest version. This command does not always appear to work on Macintoshes.


Generators
------
Ref: http://guides.rubyonrails.org/command_line.html#rails-generate

These are basically CLI commands that will help you boost your productiveness in Rails.

* `rails g` with no generator name will output a list of all available generators and some information about global options.
* `rails g GENERATOR --help` will list the options that can be passed to the specified generator.

Here's a brief list of someones I found to be useful:

#### Initial Design / Scaffolding
* `rails g model foo` Creates a new model class Foo with a skeleton class definition in app/models/foo.rb and a skeletal migration in db/migrate/*_create_foos.rb.
* `rails generate model ad name:string description:text price:decimal seller_id:integer email:string img_url:string` A more specific new model Generator
* `rails generate model Assemblies_parts assembly:references part:references` This will create a Joined table with the migration with all the foreign keys already set upm 
* `rails g controller foo a b` Creates a new controller class FooController with a skeleton class definition in app/controllers/foo_controller.rb. It also creates skeleton action methods a and b in the controller, plus skeleton views in the files app/views/foo/a.html.erb and app/views/foo/a.html.erb. If a and b are omitted then the controller is created with no actions or views.

#### Migrations
Migrations allow you to update the model structure in a secure way.
Remember to run migrations after you create them with `rake db:migrate`
* `rails g migration foo` Creates a new migration in the file db/migrate/*_foo.rb
* `rails g migration AddFieldsToRoom title:string body:text published:boolean` Adds fields to existing Model

#### ActiveAdmin
Note: [ActiveAdmin](http://activeadmin.info/) is a ruby gem that auto-generates admin panels for your Ruby Models
* `rails generate active_admin:page Thing` Registers pages with Active Admin (will create app/admin/thing.rb)

Troubleshooting
------
#### PG Gem Fail
IMO the best solution I found for this problem:

1. Install the Postgress App (from http://postgresapp.com/ ), make it start automatically at login (preferences).
2. Add the Postgress App path to /etc/paths (my path was: /Applications/Postgres.app/Contents/Versions/9.4/bin/ )
3. Install the pg gem using ARCHFLAGS: `env ARCHFLAGS="-arch x86_64" gem install pg`
4. Tears of [happiness](http://41.media.tumblr.com/08115d4b0b7b5ca2a221003c6b37c8f9/tumblr_inline_nv213bdyj91ryyjv7_540.jpg)

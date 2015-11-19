💎
# Ruby newbie
My own collection of TL;DR notes for starting Ruby on Rails + Heroku development.

Note: I'm using a MacBook Pro (Retina, 13-inch, Mid 2014) with Yosemite,
and my software background is mainly being a LAMP backend dev

Initial Setup
------

This assumes you already have either a Heroku app, or some other app in a Bitbucket/Github repo.

##### a) Cloning a Heroku app repo to your local dev env:
1. Install the [Heroku Mac Toolbelt](https://toolbelt.heroku.com/download/osx)
2. Login wiht your Heroku creds: `heroku login`
3. Go ahead and clone your app: `heroku git:clone -a APPNAME`
4. You can now deploy to Heroku like this: `git push heroku master`

##### b) Deploying your app from Bitbucket/Github to Heroku
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
* `rails g controller foo a b` Creates a new controller class FooController with a skeleton class definition in app/controllers/foo_controller.rb. It also creates skeleton action methods a and b in the controller, plus skeleton views in the files app/views/foo/a.html.erb and app/views/foo/a.html.erb. If a and b are omitted then the controller is created with no actions or views.

#### Migrations
Remember to run migrations after you create them with `rake db:migrate`
* `rails g migration foo` Creates a new migration in the file db/migrate/*_foo.rb
* `rails g migration AddFieldsToRoom title:string body:text published:boolean` Adds fields to existing Model

#### ActiveAdmin
Note: [ActiveAdmin](http://activeadmin.info/) is a ruby gem that auto-generates admin panels for you
* `rails generate active_admin:page Thing` Registers pages with Active Admin (will create app/admin/thing.rb)

Troubleshooting
------
#### Succesfully installing pg gem if it failed
IMO the best solution I found for this problem:

1. Install the Postgress App (from http://postgresapp.com/ ), make it start automatically at login (preferences).
2. Add the Postgress App path to /etc/paths (my path was: /Applications/Postgres.app/Contents/Versions/9.4/bin/ )
3. Install the pg gem using ARCHFLAGS: `env ARCHFLAGS="-arch x86_64" gem install pg`
4. Tears of [happiness](http://41.media.tumblr.com/08115d4b0b7b5ca2a221003c6b37c8f9/tumblr_inline_nv213bdyj91ryyjv7_540.jpg)

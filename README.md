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

6. Builds db schema+migrations: `rake db:migrate`
7. Inputs db seeds (default values): `rake db:seeds`

It could be that you'd rather [get a real DB dump from Production](#get-prod-db), though.

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
##### 😊 From Production to Development
😊 This is the standard procedure to get your dev DB up to date.

##### 😱 From Development to Production
😱This will *upload your local DB to Production*, you sure you wanna do this?

Troubleshooting
------
#### Succesfully installing pg gem if it failed
IMO the best solution I found for this problem:

1. Install the Postgress App (from http://postgresapp.com/ ), make it start automatically at login (preferences).
2. Add the Postgress App path to /etc/paths (my path was: /Applications/Postgres.app/Contents/Versions/9.4/bin/ )
3. Install the pg gem using ARCHFLAGS: `env ARCHFLAGS="-arch x86_64" gem install pg`
4. Tears of [happiness](http://41.media.tumblr.com/08115d4b0b7b5ca2a221003c6b37c8f9/tumblr_inline_nv213bdyj91ryyjv7_540.jpg)

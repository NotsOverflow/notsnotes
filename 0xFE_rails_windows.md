![](images/diverse/ruby.svg)

# Windows RoR

> A basic setup of ruby on rails on windows.
> You will be up and runny for coding your rails app.
> Be carefull as some library use a different line return char and my break

## Ruby on Rails:

- go to [rubyinstaller.org](https://rubyinstaller.org/downloads/) and download the recomanded file.
- install ruby with toochain to a nice place where you have user access ( eg: "D:\\" ) and don't autorun `ridk install` yet
- execute ( ❖ + r ) `"C:\Windows\system32\rundll32.exe" sysdm.cpl,EditEnvironmentVariables` to add the binaries folder to your PATH environement variable if you need to
- run an administrator powershell window and remove aslr to mysys64 executables

```powershell
$files = (Get-ChildItem 'D:\Ruby\msys64\usr\bin\*.exe').FullName
$files.ForEach({Set-ProcessMitigation $_ -Disable ForceRelocateImages})
```

- start prompt with ruby

```powershell
# Select option 3
ridk install
gem install bundler
gem install rails
```

## Git and Heroku

- go to [git-scm.com](https://git-scm.com/download/win) , download and install git ( make sure to redo the powershell trick above if you use integrated mysys64 )
- go to [heroku.com](https://devcenter.heroku.com/articles/heroku-cli#download-and-install) and download and install the Heroku client.
- Add the binaries folder to your PATH as above

## Hello World App

### Create your first app

In a command prompt enter the following commands

```powershell
#create an app ( action table is skiped here cause of a bug )
rails new herokuapp --database=postgresql  --skip-action-cable
cd herokuapp
#create a hello world
rails g controller home
```

Add the route `root "home#index"` to `config/routes.rb`
Add the following html to `views/home/index.html.erb`

```html
<h2>Hello World</h2>
<p>The time is now: <%= Time.now %></p>
```

Add `web: bundle exec puma -C config/puma.rb` to `/Procfile`
change `config/puma.rb` to :

```ruby
workers Integer(ENV['WEB_CONCURRENCY'] || 2)
threads_count = Integer(ENV['RAILS_MAX_THREADS'] || 5)
threads threads_count, threads_count

preload_app!

rackup      DefaultRackup
port        ENV['PORT']     || 3000
environment ENV['RACK_ENV'] || 'development'

on_worker_boot do
  # Worker specific setup for Rails 4.1+
  # See: https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server#on-worker-boot
  ActiveRecord::Base.establish_connection
end
```

### Deploy :

in the same terminal enter the following

```powershell
#initialise git
git init
git add .
git commit -m "init"
# open your browser and login
heroku login
heroku create
# set builder pack to version 2
heroku buildpacks:set https://github.com/bundler/heroku-buildpack-bundler2
git push heroku master
#start a dyno
heroku ps:scale web=1
#open the hello world ;D
heroku open
```

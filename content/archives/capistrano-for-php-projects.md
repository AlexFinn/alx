+++
title = "Capistrano for PHP projects"
tags = ["capistrano", "ruby", "php", "deployment"]
date = "2014-04-28"
type = "post"
+++

Some time ago I needed to find a solution for deploying PHP applications that our programmers wrote. Before the start of the project they used ftp to deploy. But it took too much unnecessary work and a long time. After some thought I decided to try capistrano. Alternatively I look at fabric but I was more familiar with capistrano earlier. And now I want to talk about using capistrano a little bit.

As I wrote earlier we use Gentoo on our servers. For capistrano tasks we need installed ruby. It can be done via portage or rvm or rbenv. We use portage.

# Installation

After that we will need bundler to manage gems.

    $ gem install bundler

Futher, we need to create a directory that will be basic. For example, `/opt/capistrano`.

    $ mkdir /opt/capistrano
    $ cd /opt/capistrano
    $ touch Gemfile

Open Gemfile and add following lines:

    source 'https://rubygems.org'

    gem 'railsless-deploy'
    gem 'capistrano', '2.15.4'
    gem 'capistrano-tags'

**Note**: we use capistrano 2.

Install gems

    $ bundle install

After that in current directory run

    $ capify .

Capitrano will create necessary files and directories.

    $ tree
    .
    ├── Capfile
    ├── config
    │   ├── deploy
    │   │   ├── production.rb
    │   │   └── testing.rb
    │   └── deploy.rb
    ├── Gemfile
    └── Gemfile.lock

# Settings

Now I briefly describe the purpose of each of the files and directories, and then I tell what is inside each file.

## Capfile

It's base configuration file for capistrano. Basically, it prescribes loading gems and other configs. Often there is not even have to change anything after the initial setup. In our case we will have to adjust periodically to add new environments if required.

    require 'railsless-deploy'

Using railsless-deploy because we haven’t Ruby on Rails.

## Multistage

    require 'capistrano/ext/multistage'

Settings for multiple environments. Instructed to use the gem to manage multiple environments.

    set :stages, [ 'testing', 'production' ]

## Environments list.

    set :default_stage, 'www_testing'

Default environment. This value will be used to run without specifying environment.

    # load deploy.rb in config-Folder
    load 'config/deploy'

Loading configs from config/deploy.rb.

In general this file rarely changes. Is that the new environment will appear, which we will need to add it.

## config/deploy.rb

The main configuration file. It describes settings that are common to all environments. For example, there may be paths to repositories, if capistrano use only for one project, a login for a repository, and other global settings.

    require 'capistrano-tags'

Gem for specify deploying tag.

    set :application, "my_project"

## Application name.

    set :scm, :subversion

We use SVN for PHP projects. Capistrano support many CVSs.

    set :scm_username, "svnuser"
    set :scm_password, "secret"
    set :svn_username, "#{scm_username}"
    set :svn_password, "#{scm_password}"

Login and password for access to repo.

    set :deploy_via, :export

Specify how we will to deploy files to target host.

    set :keep_releases, 5

A number of saved project versions.

    set :use_sudo, false

To use or not to use sudo.

    task :fix_file_permissions do
      run "sudo chown -R nginx:nginx #{deploy_to}"
      run "sudo chmod -R ug=rwX,o= #{deploy_to}"
    end

Task to set correct permissions on directories and files.

    after "deploy:update", :fix_file_permissions

Run task after deploy.

    set(:tag) { Capistrano::CLI.ui.ask("Enter SVN tag to deploy (or type 'trunk' to deploy from trunk): ") }
    set(:repository) { (tag == "trunk") ? "#{repository_root}/trunk" : "#{repository_root}/tags/#{tag}" }

Set SVN tag by default. Straight run a tag will be requested . If the tag will not specify then will be used default tag.

## config/deploy/testing.rb and config/deploy/production.rb

Both of these files are responsible for the specific parameters for a particular environment, in this case, testing and production. According to the content of each other they do not differ, differ only in specific environment parameters, such a host which the files will deploy on.

    set :deploy_to, '/var/my_project/www/'

A deploying path on destination server.

    set :user, 'deployer'

Username on remote host.

    role :app, "ngx-c1.domain.com", "ngx-c2.domain.com"

Set servers role.

    ssh_options[:port] = 21222

Set non-standart SSH port.

    set :repository_root, "svn://svn.domain.com/projects_php/my_project"

SVN repository.

Environment files differ only server names onto which the deployment, paths where it will be produced, or, for example, users, from which actions will be performed.

# Deployment

Before deploying we should to check whether all the required packages are, and to prepare the directory structure on a remote host. For
creating a structure exists following command - `cap <name environment> deploy:setup`.

    $ cap testing deploy:setup

We should ensure that the user on the remote host has all necessary permissions to create directories.

    $ cap testing deploy:check

And if we haven’t any errors we can deploy our application:

    $ cap testing deploy

testing in that case is our environment.

We can use `cap -T` command to view list of capistrano tasks. For example:

    $ cap -T
    cap production            # Set the target stage to `adm_production'.
    cap testing               # Set the target stage to `adm_testing'.
    cap deploy                # Deploys your project.
    cap deploy:check          # Test deployment dependencies.
    cap deploy:cleanup        # Clean up old releases.
    cap deploy:cold           # Deploys and starts a `cold' application.
    cap deploy:create_symlink # Updates the symlink to the most recently deployed version.
    cap deploy:pending        # Displays the commits since your last deploy.
    cap deploy:pending:diff   # Displays the `diff' since your last deploy.
    cap deploy:rollback       # Rolls back to a previous version and restarts.
    cap deploy:rollback:code  # Rolls back to the previously deployed version.
    cap deploy:setup          # Prepares one or more servers for deployment.
    cap deploy:symlink        # Deprecated.
    cap deploy:update         # Copies your project and updates the symlink.
    cap deploy:update_code    # Copies your project to the remote servers.
    cap deploy:upload         # Copy files to the currently deployed version.
    cap invoke                # Invoke a single command on the remote servers.
    cap multistage:prepare    # Stub out the staging config files.
    cap shell                 # Begin an interactive Capistrano session.

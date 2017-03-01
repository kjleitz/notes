# Cloud9 Environment Setup

Adapted from the [Ubuntu Environment Setup](https://github.com/learn-co-curriculum/linux-env-setup/blob/master/README.md).

## Setting up your Cloud9 Developer Environment

In this readme we are going to go over the steps for setting up your development environment on Cloud9 (which runs Ubuntu, a Linux distribution).

## Initialize a workspace

Select "Create a Workspace". Name it however you'd like, and select "Ruby" as the template environment you'd like to set up. Once Cloud9 has finished setting up the workspace, you will be able to type the following commands in the terminal at the bottom.

## Set up group

He we are setting up a group with the name "npm". This is so that we can set reasonable permissions on packages that get installed via npm (more on permissions and npm later). These permissions ensure that we can globally install npm packages.

 - `sudo groupadd npm`
 - `sudo usermod -a -G npm,staff $USER`

## Make sure everything is up to date

Before we start installing all of our developer tools, we want to make sure that everything is up to date. For this, we're going to use the `apt-get` Linux package manager to update everything using the following commands:

 - `sudo apt-get update`

## Installing Dev tools

### The essentials

Now we can install some essential dev tools (postgres, node...) using curl:

 - `sudo apt-get -y install curl postgresql libpq-dev default-jre build-essential phantomjs`
 - `curl -sL https://deb.nodesource.com/setup_5.x | sudo -E bash -`
 - `sudo apt-get install nodejs`

## Setting file permissions

To be able to properly use some of these tools, we need to manually set some of the permissions and owners of some of the recently installed files:

 - `sudo chown root:staff /usr/bin`
 - `sudo chmod 0775 /usr/bin`
 - `sudo chown -R root:npm /usr/lib/node_modules`
 - `sudo chmod 0775 /usr/lib/node_modules`

If you're curious, you can read more about `chmod` (setting permissions) and `chown` (setting the owner) [here](http://www.unixtutorial.org/2014/07/difference-between-chmod-and-chown/).
Take a look at [this page](http://www.perlfect.com/articles/chmod.shtml) if you want to understand the permission values (0755, 0600, ...)

## Set up .netrc file for the learn gem

The learn gem needs this netrc file to work. The `.netrc` file is a standard location to store login/token info, so it is we store information needed for Learn.

 - `touch ~/.netrc && chmod 0600 ~/.netrc`

## Using RVM to install Ruby

[RVM](https://en.wikipedia.org/wiki/Ruby_Version_Manager) is a great tool that lets you run different versions of Ruby on your computer. This is really useful because if you know one project your working on works with Ruby version 2.1.0 and another needs 2.3.0, you can easily switch between the two versions when you switch between projects. It's already installed in your Cloud9 workspace, but you'll want to use it to install a slightly newer version of Ruby than the one installed by default. You can install it and set it up with the following commands:

 - `rvm install 2.3.1`
 - `rvm use 2.3.1 --default`

You can easily install different Ruby verions with `rvm instal <version number>`, switch between versions with `rvm use <version number>` and check to see which you've already installed with `rvm list`. You can always check what version your current terminal window is with `ruby -v`. Later, you may want to also install the recent Ruby versions `2.3.3` and/or `2.4.0`, but that's up to you. You can always switch the one used by default with `rvm use <version number> --default`, or use one temporarily with `rvm use <version number>`. I'd come back to this after you've completed the environment setup, just for consistency.

## Setting up (and getting) Ruby gems

If your familiar with any other program language, [Ruby gems](https://en.wikipedia.org/wiki/RubyGems) are like libraries. If you're not familiar with programing libraries, it's basically isolated chunks of code that you can easily add to your project. For example, the `learn-co` gem allows you to easily interface with Learn from your command line (opening and submitting labs among many other things).

To set up your gems and install some necessary ones, run the following commands:

 - `echo "gem: --no-ri --no-rdoc" > $HOME/.gemrc`
 - `gem update --system 2.4.8`
 - `gem install learn-co phantomjs pg sqlite3 bundler rails`

## Installing a Node Pacakge

One thing you'll see more of later in the course is npm, or [Node Package Manager](https://en.wikipedia.org/wiki/Npm_(software)). This is very similar to Ruby gems, but it's for JavaScript. We're also going to do a global install for an npm testing package Protractor with the following command:

 - `sudo npm install -g n`
 - `sudo npm install -g protractor`

## Setting up your computer up with Github

All of this courses content is stored on Github so you're going to have to do a lot of cloning (copying files from your github account to your computer) and pushing (taking files you've saved on your computer, and updating your github files with them).

It would be a real pain if you have to type your password in every time you wanted to perform one of these actions. Luckily you don't have to with an SSH key. You can set this up by following the following steps:

 - `ssh-keygen` (ONLY DO THIS IF YOU DON'T ALREADY HAVE A GENERATED SSH KEY: just press return for everything, and don't enter a passphrase)
 - `cat ~/.ssh/id_rsa.pub`. This will disply the output of your SSH key to your terminal. You can then copy that, and add as ssh key on github by following the [instructions posted on github](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/). Note: The first step in these instructions is _the same_ as doing the `cat ~/.ssh/id_rsa.pub` command and manually copying the text.
 - You're also going to want to let the git that is running on your machine to know you you are. You can set this up by running: `git config --global user.email "you@example.com"` and `git config --global user.name "Your Name"`

## Configure the Learn-Co gem

Now that you have most of your tools installed, you're going to want to configure your learn-co gem. As previously mentioned, this gem lets you run tests, open labs, submit them, and save them to finish later. You can configure the gem to your account by using the following command:

 - `learn whoami` and enter oauth token when asked (from https://learn.co/<github_username>)

## Optional Dotfiles

These dotfiles do a variety of different things and I highly recomend you download them.

 - `curl "https://raw.githubusercontent.com/flatiron-school/dotfiles/master/irbrc" -o "$HOME/.irbrc"` - This file gives you some nice formatting for when you're in IRB (IRB lets you write ruby code in your terminal)
 - `curl "https://raw.githubusercontent.com/flatiron-school/dotfiles/master/ubuntu-gitignore" -o "$HOME/.gitignore"` - Global .gitignore rules. When you add a .gitignore file to a project, it let's you specify certain files that you DO NOT want pushed up to github (like API keys...)
 - `curl "https://raw.githubusercontent.com/flatiron-school/dotfiles/master/linux_bash_profile" -o "$HOME/.bash_profile"` - Your bash profile loads up every time you open a terminal window. The Learn bash_profile is designed to load up a bunch of shortcuts for you as well as make sure that RVM loads up every time you open the terminal. I recommend you take a look at this file and even see if there are any shortcuts of your own that you'd like to add! Note: this will overwrite existing bash profile, so back up if you want to.
 - `curl "https://raw.githubusercontent.com/flatiron-school/dotfiles/master/linux_gitconfig" -o "$HOME/.gitconfig"` then `nano $HOME/.gitconfig` and edit what needs to be edited (github username and github email in a few places)

## Using `rackup` and `shotgun`

Normally, when you use `rackup` or `shotgun` to serve a site locally (e.g. with Sinatra), a local server is set up so that you can access it from _only_ your computer. However, Cloud9 is not your computer! You need to tell `rackup` and `shotgun` which IP address and port to serve your site on, so you can use its built-in browser to access it (as opposed to opening a new tab and navigating to `http://localhost:9292` like normal). You can use the following commands to do the equivalent of `rackup` and `shotgun` in your Cloud9 workspace (you don't need to run these right now):

- `rackup -p $PORT -o $IP`
- `shotgun -p $PORT -o $IP`

But those are long and annoying to type out. Instead, we're going to make aliases so you can use them more easily. Run the following commands:

- `echo "alias ru='rackup -p $PORT -o $IP'"`
- `echo "alias sgun='shotgun -p $PORT -o $IP'"`

Now, if you want to use the `rackup` command, just do `ru` instead. If you want to use the `shotgun` command, do `sgun` instead.
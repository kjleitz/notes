# Cloud9 Environment Setup

Adapted from the [Ubuntu Environment Setup](https://GitHub.com/learn-co-curriculum/linux-env-setup/blob/master/README.md).

## Setting up your Cloud9 Developer Environment

In this README, we will go over the steps for setting up your development environment in a Cloud9 workspace (which runs Ubuntu, a Linux distribution).

## Initialize a workspace

Select "Create a Workspace". Name it `cloud9-learn`, describe it however you'd like ("A workspace for completing labs on Learn.co" would be simple and appropriate), and select "Ruby" as the template environment you'd like to set up. Finally, press the "Create workspace" button. Once Cloud9 has finished setting up the workspace, you will be able to type the following commands in the terminal at the bottom.

## Entering commands

The commands provided in this guide are meant to be typed into the terminal pane at the bottom of your Cloud9 workspace. Here are some common tips for beginners to the command line:

 - You will need to use the arrow keys to move your cursor (the black rectangle), as clicking with your mouse won't move it to the position you want, unlike normal text editors
 - If you want to re-enter a previous command, but you don't want to re-type it, you can hit the up arrow on your keyboard to recall commands you have recently entered, then edit those commands however you like (press 'enter' to run them, like a regular command)
 - If you enter a command and nothing happens, that's okay. If you knew what you were doing and you were _expecting_ something to happen, of course that's an issue... but many of the commands listed in this guide do not have any output, and it's perfectly normal for that command to go gentle into that good night

## Set up group

Here we are setting up a group with the name "npm". This is so that we can set reasonable permissions on packages that will be installed via npm (more on permissions and npm later). These permissions ensure that we can globally install npm packages.

 - `sudo groupadd npm`
 - `sudo usermod -a -G npm,staff $USER`

## Make sure everything is up to date

Before we start installing all of our developer tools, we want to make sure that the packages we install are all up to date. For this, we're going to use the `apt-get` Linux package manager to update our sources:

 - `sudo apt-get update`

## Installing Dev tools

### The essentials

Now we can install some essential dev tools (postgres, node...) using `curl`:

 - `sudo apt-get -y install curl postgresql libpq-dev default-jre build-essential phantomjs`
 - `curl -sL https://deb.nodesource.com/setup_5.x | sudo -E bash -`
 - `sudo apt-get install nodejs`

## Setting file permissions

To be able to properly use some of these tools, we need to manually set some of the permissions and ownership of some of the recently installed files:

 - `sudo chown root:staff /usr/bin`
 - `sudo chmod 0775 /usr/bin`
 - `sudo chown -R root:npm /usr/lib/node_modules`
 - `sudo chmod 0775 /usr/lib/node_modules`

If you're curious, you can read more about `chmod` (setting permissions) and `chown` (setting the owner) [here](http://www.unixtutorial.org/2014/07/difference-between-chmod-and-chown/).
Take a look at [this page](http://www.perlfect.com/articles/chmod.shtml) if you want to understand the permission values (0755, 0600, ...)

## Set up .netrc file for the learn gem

The learn gem needs this netrc file to work. The `.netrc` file is a standard location to store login/token info, so it is where we store information needed for Learn.

 - `touch ~/.netrc && chmod 0600 ~/.netrc`

## Using RVM to install Ruby

[RVM](https://en.wikipedia.org/wiki/Ruby_Version_Manager) is a great tool that lets you run different versions of Ruby on your computer. This is really useful because if you know one project you're working on works with Ruby version 2.1.0 and another needs 2.3.0, you can easily switch between the two versions when you switch between projects. It's already installed in your Cloud9 workspace, but you'll want to use it to install a slightly newer version of Ruby than the one installed by default. You can install it and set it up with the following commands:

 - `rvm install 2.3.1`
 - `rvm use 2.3.1 --default`

You can easily install different Ruby verions with `rvm install <version number>`, switch between versions with `rvm use <version number>` and check to see which you've already installed with `rvm list`. You can always check what version your current terminal window is with `ruby -v`. Later, you may want to also install the recent Ruby versions `2.3.3` and/or `2.4.0`, but that's up to you. You can always switch the one used by default with `rvm use <version number> --default`, or use one temporarily with `rvm use <version number>`. I'd come back to this _after_ you've completed the environment setup, just for consistency.

## Setting up (and getting) Ruby gems

If you're familiar with any other programming language, [Ruby gems](https://en.wikipedia.org/wiki/RubyGems) are like libraries. If you're not familiar with programming libraries, they are basically isolated chunks of code that you can easily add to your project. For example, the `learn-co` gem allows you to easily interface with Learn from your command line (opening and submitting labs, among many other things).

To set up your gems and install some necessary ones, run the following commands:

 - `echo "gem: --no-ri --no-rdoc" > $HOME/.gemrc`
 - `gem update --system 2.4.8`
 - `gem install learn-co phantomjs pg sqlite3 bundler rails`

## Installing a Node Pacakge

One thing you'll see more of later in the course is npm, or [Node Package Manager](https://en.wikipedia.org/wiki/Npm_(software)). This is very similar to Ruby gems, but it's for JavaScript. We're also going to do a global install for an npm testing package Protractor with the following command:

 - `sudo npm install -g n`
 - `sudo npm install -g protractor`

## Setting up your computer up with GitHub

All of this courses content is stored on GitHub so you're going to have to do a lot of cloning (copying files from your GitHub account to your computer) and pushing (taking files you've saved on your computer, and updating your GitHub files with them).

It would be a real pain if you had to type in your password every time you wanted to perform one of these actions. Luckily you don't have to if you use an SSH key. You can set this up by following these steps:

 - `cat ~/.ssh/id_rsa.pub` - This will display the output of your SSH key to your terminal. You can then copy that and add it as an SSH key on GitHub by following the [instructions posted on GitHub](https://help.GitHub.com/articles/adding-a-new-ssh-key-to-your-GitHub-account/). Note: The first step in these instructions is _the same_ as doing the `cat ~/.ssh/id_rsa.pub` command and manually copying the text, so you can skip it.

## Configure the Learn-Co gem

Now that you have most of your tools installed, you're going to want to configure your learn-co gem. As previously mentioned, this gem lets you run tests, open labs, submit them, and save them to finish later. You can configure the gem to be associated with your account by using the following command:

 - `learn whoami` - Then, enter your OAuth token when asked (go to your Learn.co profile at https://learn.co/<GitHub_username> and copy the OAuth token from the bottom of the page)

## Optional Dotfiles

These dotfiles do a variety of different things and they are highly recommended, but optional. The command `curl` downloads a file from a URL and saves it to a file you specify after the `-o` flag.

 - `curl "https://raw.githubusercontent.com/flatiron-school/dotfiles/master/irbrc" -o "$HOME/.irbrc"` - This file gives you some nice formatting for when you're in IRB (IRB lets you write ruby code in your terminal)
 - `curl "https://raw.githubusercontent.com/flatiron-school/dotfiles/master/ubuntu-gitignore" -o "$HOME/.gitignore"` - Global .gitignore rules. When you add a .gitignore file to a project, it let's you specify certain files that you DO NOT want pushed up to GitHub (like API keys...)
 - `curl "https://raw.githubusercontent.com/flatiron-school/dotfiles/master/linux_bash_profile" -o "$HOME/.bash_profile"` - Your bash profile loads up every time you open a terminal window. The Learn bash_profile is designed to load up a bunch of shortcuts for you, and makes sure RVM loads up every time you open the terminal.
 - `curl "https://raw.GitHubusercontent.com/flatiron-school/dotfiles/master/linux_gitconfig" -o "$HOME/.gitconfig"` - Then, `nano $HOME/.gitconfig` and edit what needs to be edited by arrow-ing to the placeholder spots, deleting them, and replacing them with the appropriate values (GitHub username and GitHub email in a few places). To save a file in nano, press ctrl+O, then verify that the file name is what you would like (it will already be filled in with the original filename, so you shouldn't have to change anything), then press 'enter', and finally press ctrl+X to quit nano.

## Using `rackup` and `shotgun`

### The full commands

Normally, when you use `rackup` or `shotgun` to serve a site locally (e.g. with Sinatra), a local server is set up so that you can access it from _only_ your computer. However, Cloud9 is not your computer! You need to tell `rackup` and `shotgun` which IP address and port to serve your site with, so you can navigate to the URL it supplies to access it (as opposed to navigating to `http://localhost:9292` like normal). You will need to use the following commands to use `rackup` and `shotgun` on Cloud9 (you don't need to run these commands at the moment, just when you need to start a server with `rackup` or `shotgun` for your lessons):

- `rackup -p $PORT -o $IP`
- `shotgun -p $PORT -o $IP`

When you use one of these commands, a little window will pop up in the top right corner of the terminal which says "Your code is running at https://cloud9-learn-your_c9_username.c9users.io", and you can open that URL in a new tab to view your application being served.

### Setting up shortcuts

Those full commands are long and annoying to type out. Instead, we can make aliases so you can use them more easily. Run the following commands:

- `echo "alias rackup='rackup -p \$PORT -o \$IP'" >> ~/.bash_aliases`
- `echo "alias shotgun='shotgun -p \$PORT -o \$IP'" >> ~/.bash_aliases`

Now you can use `rackup` and `shotgun` like normal, and they will execute the full commands by default, no extra typing required!

Again, when you use one of these commands, a little window will pop up in the top right corner of the terminal which says "Your code is running at https://cloud9-learn-your_c9_username.c9users.io", and you can open that URL in a new tab to view your application being served.

## Opening labs

To open the lab you are currently on, run the command `learn open`. If you want to open a specific lab that you are not currently on, copy the repository name from GitHub for that lab, and run `learn open some-repository-name-v-000` (or shorter, like this: `learn open some-repository-name`). This will create a `~/code/labs` directory where your labs will be stored. To view this directory in your Cloud9 workspace file tree (the left-hand sidebar), click the little gear icon at the top of the left-hand sidebar and select "Show Home in Favorites". You can also **un**select "Show Workspace Root" to keep your file tree a little less cluttered (you probably won't be needing the workspace files Cloud9 sets up for you, anyway).

Now, you should be able to access your labs in the left-hand sidebar under `~/code/labs`!
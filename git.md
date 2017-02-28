#Git notes

###Start new project

To start a new project, easiest way is to create a new repository on your GitHub page, then:

```
mkdir example-app
cd example-app
git init
git remote add origin https://github.com/kjleitz/example-app
```

###`push` + `pull`

Push takes two arguments (optional after setting upstream). The first is the remote repo name (or alias: usually “origin”). The second is the name of the branch you want to send it to. “origin” is just short for the remote URL.

```
git push origin master
git push
```

Pull also takes the same two arguments (optional after setting upstream).

```
git pull origin master
git pull
```

###Remote sources and upstream tracking

To set upstream tracking, the easiest way to do it is using the “-u” flag in your first push:

```
git push -u origin master
```

If you want to see your remote sources and their aliases, you can use:

```
git remote -v
```

If you want to know all the remote branch names, you can find them with:

```
git branch -r
```

###Forking and pull requests

If you want to apply a bug fix or play with some code from someone else’s project, go to their GitHub page and press the “Fork” button. This will create a copy on your GitHub account. Then, clone your forked project. You can add the original (“upstream” is a usual alias) repository as well, so that you can pull from there and keep your local project synced with the original source!

```
git clone https://github.com/kjleitz/someone-elses-project-forked
git remote add upstream https://github.com/someone-else/someone-elses-project-forked
```

…and to keep it synced, you can do this (I think `git pull` in this scenario does this all at once):

```
git fetch upstream.        # <= fetches the upstream branches and their commits
git checkout master        # <= enter your local master branch
git merge upstream/master  # <= merge the changes from the upstream master
                           #    branch (or whatever branch you’re looking for, after the
                           #    slash) into your (currently checked-out) local master
                           #    branch
```

It’s good practice to always leave the master branch not broken/buggy. So you should branch out and then make your changes to the branch.

```
git branch some-new-branch			# <= makes a new branch
git checkout -b some-new-branch		# <= makes a new branch and switches to it
```

And to see the branches we can do:

```
git branch -a
```

When we want to merge a branch, you want to be in the target branch.

```
git checkout master
git merge some-other-branch
```

To create a pull request, go to the forked repository on your profile page, go to the “Pull Requests” tab, and click the green “New Pull Request” button. Then, after comparing the differences, press “Create Pull Request” (not sure if that last part is it).

### Logs and viewing past commits

```
git log         # list commits, reverse chronological order
git log -2      # the most recent 2 commits
git log -5      # the most recent 5 commits
git log -p -2   # the most recent 2 commits, with diffs
```
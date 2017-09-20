# Bash notes

## Tricks

### Background tasks

To run a task in the background, put a `&` after it. For example, `jekyll serve &`. You may have to press 'enter' to get your prompt back, but yeah.

To see what jobs you have running in the background, use the `jobs` command. Each job will have a number like `[1]` in front of it.

To switch between jobs, use the `fg` command followed by the job number of your background task you'd like to switch to. So, if your job was `[3]`, simply do `fg 3`. Then, you can do whatever you want with it like normal (`ctrl + c` to quit, obviously).

### Find process by name

```
$ ps aux | grep chromedriver
keegan           32705   0.0  0.0  2534068    952   ??  S    Thu02PM   0:00.03 /usr/local/opt/chromedriver/bin/chromedriver
keegan           69009   0.0  0.0  2432804    800 s001  S+    5:13PM   0:00.00 grep chromedriver
$ ps aux | grep chromedriver | grep -v grep
keegan           32705   0.0  0.0  2534068    952   ??  S    Thu02PM   0:00.03 /usr/local/opt/chromedriver/bin/chromedriver
$ ps aux | grep [c]hromedriver
keegan           32705   0.0  0.0  2534068    952   ??  S    Thu02PM   0:00.03 /usr/local/opt/chromedriver/bin/chromedriver
$ pgrep chromedriver
32705
```

### Oh shiiiiiit, use `awk` to pull an "argument" out of a string

```
$ ps aux | grep [c]hromedriver
keegan           32705   0.0  0.0  2534068    952   ??  S    Thu02PM   0:00.03 /usr/local/opt/chromedriver/bin/chromedriver
$ ps aux | grep [c]hromedriver | awk '{print $2}'
32705
```

That's pretty darn cool. Gonna use that more often.

### Abort whole script if any errors

```bash
#!/bin/bash

set -e

# the rest of your script...
```

Now the whole script will abort if something errors out.


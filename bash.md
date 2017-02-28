# Bash notes

## Tricks

### Background tasks

To run a task in the background, put a `&` after it. For example, `jekyll serve &`. You may have to press 'enter' to get your prompt back, but yeah.

To see what jobs you have running in the background, use the `jobs` command. Each job will have a number like `[1]` in front of it.

To switch between jobs, use the `fg` command followed by the job number of your background task you'd like to switch to. So, if your job was `[3]`, simply do `fg 3`. Then, you can do whatever you want with it like normal (`ctrl + c` to quit, obviously).
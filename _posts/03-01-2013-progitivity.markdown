---
layout: post
title: 'Progitivity'
permalink: 'progitivity'
published: true
---

Every now and then I start searching about how to be more productive with some tool. So I thought it might come in handy sharing what I've learned about being more productive with git, and which are my current settings. 

It's time for some *progitivity*.

##Git status is too verbose

Here's an example of how a git status looks right now in this site's repo:

{% highlight console %}
in msanroman.github.com/ on master 
λ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#   modified:   _layouts/global.html
#   new file:   _posts/02-01-2013-progitivity.mdown
#   new file:   css/syntax.css
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#   modified:   _layouts/global.html
#   modified:   _posts/02-01-2013-progitivity.mdown
#   modified:   _posts/14-05-2012-Building-a-Backbone-application-with-CoffeeScript-and-Jasmine-part-one.markdown
#   modified:   _posts/16-12-2011-PHP-continous-testing-script.markdown
#   modified:   css/syntax.css
#   modified:   js/scrolling-header.js
{% endhighlight %}

If you use git in a daily manner, this output has too much useless information. You already know how to add a file. You know how to discard changes in the working directory. All you need to know is what is already going to be committed and what not.

*git status **-sb*** seems to be the option you are looking for:

{% highlight console %}
in msanroman.github.com/ on master 
λ git status -sb                   
## master
MM _layouts/global.html
AM _posts/02-01-2013-progitivity.mdown
 M _posts/14-05-2012-Building-a-Backbone-application-with-CoffeeScript-and-Jasmine-part-one.markdown
 M _posts/16-12-2011-PHP-continous-testing-script.markdown
AM css/syntax.css
 M js/scrolling-header.js
{% endhighlight %}

Gorgeous! Isn't it? Left column means what's staged for commit (*a* stands for addition, *m* for modification).

##Repository information in your prompt

As you have seen on the last code snippets, this is how my command prompt looks like:

![My custom command prompt](http://f.cl.ly/items/230h1q12160z3Y331Y2K/command-prompt.png)

Its structure is: *in **folder_name** on **branch** [with unpushed]*, this allows me to know exactly what I need when I'm in the terminal: where I am on my filesystem and where I am on my repository. I've been using zsh as my main shell for half a year and I don't miss bash not even a little bit. Zsh allows me to do exactly the same I did with bash, but also has a really comfortable completion and combining it with oh-my-zsh makes it easily customizable.

If you want to have this kind of information and you use zsh, I recommend that you install [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) and surf around the themes, learn how they work and build your own. If you use bash, I used to have this on my .bashrc:

{% highlight bash %}
export PS1='\e[34m[\t] \w\e[m $(__git_ps1 "\[\e[31m\](%s)\[\e[0m\]")\n\u@\h$ '
{% endhighlight %}

But I'm not sure if it currently works 100% fine or not.

##Alias for common commands

Those are my most important aliases:

{% highlight bash %}
[alias]
    ci = commit
    co = checkout
    st = status -sb
    au = add --update
    aa = add --all
    hunk = add --all --patch
{% endhighlight %}

There are several git commands, like *commit* or *checkout* that are long enough for making typos writing them, and having some short aliases for them is pretty handy. *git status -sb* is my **only** way of using git status, so I had to make a short alias for it.

The other three commands are just awesome enough for a description of each of them:

+ **add --update** stages modifications of tracked files, as well as removes files from the index if their corresponding files from the working tree have been erased.
+ **add --all** does the same as *add --update*, but also with files on the working tree. If a file is added/modified/removed in the working tree and wasn't in the index, it will be added/modified/removed after running this command.
+ **add --all --patch** does the prior command, but letting you interactively choose hunks of patches between the index and the working tree. It's greate for splitting your current work into multiple commits.

##Parameterized aliases

If you use [git-flow](http://nvie.com/posts/a-successful-git-branching-model/) (or a variation) as your git branching model, you might want to maje an alias that creates a feature branch with a specified *name*:

{% highlight bash %}
[alias]
    #(...)
    feature = !sh -c 'git checkout -b feature/$1' -
{% endhighlight %}

By putting an exclamation mark (!), Git will run it as a *non-git* command in the shell.

Demo:

{% highlight console %}
in msanroman.github.com/ on master 
λ git feature test
M   _layouts/global.html
A   _posts/02-01-2013-progitivity.mdown
M   _posts/14-05-2012-Building-a-Backbone-application-with-CoffeeScript-and-Jasmine-part-one.markdown
M   _posts/16-12-2011-PHP-continous-testing-script.markdown
A   css/syntax.css
M   js/scrolling-header.js
Switched to a new branch 'feature/test'
in msanroman.github.com/ on feature/test 
λ
{% endhighlight %}

##Git extras

The example I just did is a really simple version of the *git feature* command included in [git-extras](https://github.com/visionmedia/git-extras), a cool git tool made by [TJ Holowaychuk](http://tjholowaychuk.com/) (aka [*visionmedia*](https://github.com/visionmedia/)).

This tool automates things like creating a release (by doing a sequence of executing a pre-release script, commiting changes with a "Release <tag>" message, tags with the given tag, push the branch and the tag and executes a post-release script), or starting/finishing a feature branch (the latter done by merging it with the current branch and removing both the local and the remote branches).

I've been using it for a year and it's an awesome tool for managing medium/big projects with a branch-per-feature (or bug) methodology.

##Hooks

You can define your own hooks per repository. The most important for me is the *pre-commit* hook. This script is executed against the code that is gonna be commited, so you can do cool things like ensuring that your test suite passes before actually commiting this bunch of code; if the tests suite fails, the commit will be cancelled.

Here's the sample code of a pre-commit hook written in PHP and executing Behat on the current snapshot:

{% highlight php %}
#!/usr/bin/php
<?php
echo "Running Behat test suite...";
exec('behat', $output, $returnCode);
if ($returnCode != 0)
 echo implode(PHP_EOL, $output);
die($returnCode);
?>
{% endhighlight %}

##Autocompleting

Who doesn't love autocompleting?

Add this to your .bashrc (or .zshrc), and define *$SCRIPTS_FOLDER* as your preferred place for keeping this kind of scripts:

{% highlight bash %}
GIT_BASH_COMPLETION_SCRIPT=$SCRIPTS_FOLDER/git-completion.bash
[ ! -r $GIT_BASH_COMPLETION_SCRIPT ] && cd $SCRIPTS_FOLDER && wget https://raw.github.com/git/git/master/contrib/completion/git-completion.bash
[ -r $GIT_BASH_COMPLETION_SCRIPT ] && source $GIT_BASH_COMPLETION_SCRIPT
{% endhighlight %}

When opening a new terminal, it will check if the autocompletion script is downloaded, and if it is, this will *source* it. If it isn't downloaded yet, the script will download it on that folder too.
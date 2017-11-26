---
layout: post
title: "Managing Dotfiles With Git; Pathogen and Plugins"
category: setup
tags: [vim, Git, Fugitive, Command-T, Pathogen]
---
In the last post, I went through some of the first real customizations I did for my vimrc. Soon thereafter, I discovered the power of [Pathogen](https://github.com/tpope/vim-pathogen) and the bundle/submodule mechanism of managing vim plugins. And then I kinda went crazy with the plugins.

# Dotfile Management With Git

First, let me mention that I keep my dotfiles in a [git repo](https://github.com/treed/dotfiles). I have a number of machines that I regularly find myself on, and it's nice to be able to quickly reproduce a basic setup anywhere. The repo includes a top-level install.sh, which installs the various files as symlinks to the local git repo. Feel free to grab it as a basis for your own. (I know I grabbed a lot of it from others.) The symlinking rather than mere copying is so that I can just pull down updates and have them work without me having to install anything again.

The other nice thing about managing dotfiles with git is that I can use git's [submodule](http://progit.org/book/ch6-6.html) feature to link directly to the upstream repos to make tracking them simpler. It's true that git submodules have some rough edges, but overall it's a very nice system. One such rough edge is the fact that using plugins via bundle will result in them generating tags files, which makes the submodule "dirty" from git's point of view. In the FAQ for Pathogen, Tim Pope has a nice workaround for this: configuring a global git ignore file, and using that to ignore all tags files everywhere.

{% highlight bash %}
git config --global core.excludesfile '~/.cvsignore'
echo tags >> ~/.cvsignore
{% endhighlight %}

Ultimately, if you don't currently use git for managing your dotfiles, or don't plan to, you should ignore some of what I'm about to explain in terms of arranging files and the like. Instead you can just download the files however you want and put them in the appropriate place. I should also call out that some people prefer [Vundle](https://github.com/gmarik/vundle) as an alternative to Pathogen. I've looked at it and I think I prefer the submodule+pathogen method.

# Managing Vim Plugins With Pathogen

On the top-level of my dotfiles repo, I have a vimrc file, and a vim directory. These are, as mentioned above, symlinked into the appropriate position by install.sh. The vim directory has the various subdirectories you might expect and one more: `vim/bundle`. This is where the plugins go, and I'll start with putting pathogen there:

{% highlight bash %}
git submodule add https://github.com/tpope/vim-pathogen.git vim/bundle/vim-pathogen
{% endhighlight %}

This will add the pathogen upstream git repo into the local repo by reference. You should see it cloning the repo as appropriate.

After which, you can open your .vimrc and stick on the top:

    runtime bundle/vim-pathogen/autoload/pathogen.vim
    call pathogen#infect()

Notably this must occur before `filetype plugin indent on` or the like. Best to just stick it right at the top.

Now that we've got that settled, let's get to some plugins.

# Plugins

I've got [quite a few](https://github.com/treed/dotfiles/tree/master/vim/bundle) plugins myself, and this first time out, I'll restrict myself today to discussing the most useful ones.

## CommandT

By far, the one that saves me the most time is [CommandT](https://wincent.com/products/command-t). CommandT allows you to open files based on a lazy match. It indexes all of the files in the current working directory and its subdirectories, recursively (defaulting to 15 levels down).

This one may not be as *big* of a time saver for everyone, but as an example the operations repo at IMVU (which is where I spend most of my time), has thousands of files in it. The website tree is even bigger. We organize it to the extent we can, but no one can keep a tree that big in their head entirely. (But see below for a note about size.)

But with CommandT, I don't have to remember the exact path to a file. If I know that what I'm looking for is a script intended for use with Nagios and impacts Redis, I can hit `<Space>o` (this being my [quick command](/setup/2012/03/07/quick-motion/) for `:CommandT`), then just type "scrnagred" and hit enter. CommandT will have found "*scr*ipts/fact/*nag*ios_reporters/*red*is_status.pl". How great is *that*?

While it makes large trees useful, I find that it's nice even when the repo has only a few files. `<Space>o` has become muscle-memory and I've gotten very used to opening things by fuzzy paths.

Once you've cloned CommandT into your bundle directory, you'll need to do some additional installation. First of all, you need to make sure that your vim is compiled with Ruby support. On Debian/Ubuntu, this is the case if you've installed any of vim-athena, vim-gnome, vim-gtk, or vim-nox.

To actually complete installing CommandT, you'll need to also make sure that you have some Ruby development packages. For Debian-based distros, you'll need ruby, ruby-dev, and rake. Once you have those, cd to vim/bundle/command-t and run `rake make` to install it. If any parts are missing, you'll see the errors here.

CommandT has a number of configuration options, which let you change the way it's displayed or the way it indexes files. Here are mine:

{% highlight vim %}
let g:CommandTMaxHeight=20
let g:CommandTMaxFiles=20000
{% endhighlight %}

The first case limits the maximum height. I find this nice because otherwise it'll end up sucking up much of the window when you have lots of files.

The second one increases the number of files CommandT will index. The default is 10000, and I find that any higher than 20000 (even on an SSD) makes it intolerably slow.

And so there you have it:

![screenshot of CommandT looking for 'vimsyn'](/assets/images/commandt-vimsyn.png)

## Fugitive

Another very helpful plugin I use is another [Tim Pope](http://tpo.pe/) creation, [Fugitive](https://github.com/tpope/vim-fugitive). It's an integration of git with vim, and this time the installation is as simple as checking it out into a submodule in vim/bundle.

{% highlight bash %}
git submodule add https://github.com/tpope/vim-fugitive.git vim/bundle/vim-fugitive
{% endhighlight %}

If you want to be able to access the documentation from within vim, you'll also need to generate helptags. This is technically true of anything you install for use with Pathogen. I personally have configured this to be done every time I start vim by adding the following to my vimrc. Some people choose not to, in deference of startup time. The important thing is that it's run at least once each time you update a bundle.

{% highlight vim %}
pathogen#helptags()
{% endhighlight %}

One of the best and immediate uses of Fugitive is `:Gstatus`, which will bring up something resembling the output of `git status`, except that it's interactive. You can highlight a filename and hit dash to toggle whether or not the modifications are staged for commit. You can also hit D to bring up a (Fugitive-enhanced!) vimdiff of the file, which will allow you to stage portions of the file. Once you're done, you can hit C to `:Gcommit`. There's no `:Gpush`, but you can do `:Git push` or `:Git svn dcommit` as appropriate.

One feature I find myself using semi-regularly is `:Gblame`, which will split the window vertically and show a list of which commits (and committers) were responsible for each line. This is sometimes helpful for quickly ascertaining who you should go bug about something, or just knowing if this particular line has changed recently.

I recommend that you check out its [docs](https://github.com/tpope/vim-fugitive/blob/master/doc/fugitive.txt) for the other commands you can use with it.

# Summary

I know I went over the topics fairly quickly, but hopefully you were generally familiar with the subject matter enough to get the important details here. If not, some googling will probably set you straight. I intend to write some more about some of my other plugins I use in the future.

+++
date = "2015-12-28T10:30:41-05:00"
tags = []
title = "My Dotfiles"

+++

My dotfiles are my pride and joy. There are hundreds of hours spent tweaking them to create the best setup for me. Recently I've started working on three different machines, my Mac, a Windows machine, as well as Ubuntu. This introduced an interesting issue; how can I create a similar environment across all these platforms, but only store them in one location?

<!--more-->

## Tools

In my workflow, I have the following tools setup:

 1. Neovim
 2. Git
 3. tmux
 4. zsh

In fact, this very blog post is being written using all 4 of these tools

 [![my-shell](/img/my-shell.png)](/img/my-shell.png)



## The Problem

Each of these tools has to be installed at some point for each environment I want to run, but each environment has different ways to install these tools.
For Mac, it's fairly easy, just use homebrew and I can group these into one install command.

```bash
$ brew install neovim git tmux zsh
```

Then I can sit back and just relax while the real work is done.


For linux on the other hand, it's not so easy. While there is linuxbrew, the linix port of homebrew, it's often out of date and requires a lengthy `agt-get` install for any dependencies. Then add a few lines to my shells `rc` file....THEN I can install things.

Don't even get me started on windows....I seriously haven't thought about how much of a pain it will be to get things setup on windows.



## Solution

So while all of these have a slightly different problem, I can write a simple [install script](https://github.com/mhartington/dotfiles/blob/master/install.sh) in bash to handle most of it.

In that script, it handles my setup for a fresh Mac machine.

But what about Ubuntu? Well, I just write another script.

It may not be the most elegant solution, but for me and what I need, it works great.

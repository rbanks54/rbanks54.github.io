---
layout: post
title: A simpler git status
date: '2018-06-07T12:00:00.001+10:00'
author: Richard Banks
modified_time: '2018-06-07T12:00:00.001+10:00'
---

Following on from my last git related [post about reviewing pull requests](/2018/04/reviewing_vsts_pull_requests.html) I thought I'd quickly let you know of an approach I use when checking the status of my local git repo.

Here's your typical, lengthy, verbose output from `git status`
```
λ git status
On branch master
Your branch is up to date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        _posts/2018-06-07-simpler-git-status.md

nothing added to commit but untracked files present (use "git add" to track)
```

I prefer a much simpler, shortened output that will just show me the status of files in my current branch, as shown here:

```
λ git st
## master...origin/master
?? _posts/2018-06-07-simpler-git-status.md
```

This is the output from the `git status --short --branch` command. Nothing fancy there, but if you look again you'll see that the command I used is `git st`.

I created this command using git's *alias* feature. You can do exactly the same in your local git configuration by running:

`git config --global alias.st status --short --branch`

I hope you find it as useful as I do and that it makes your life just that teensy little bit better.
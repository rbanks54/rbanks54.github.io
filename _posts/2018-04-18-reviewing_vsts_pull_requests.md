---
layout: post
title: Reviewing VSTS pull requests locally
date: '2018-04-18T12:00:00.001+10:00'
author: Richard Banks
modified_time: '2018-04-18T12:00:00.001+10:00'
---

Many teams review their pull requests (PRs) by looking at the code in the PR branch. This can be risky at times because it assumes that the source branch has pulled the most recent copy of the target branch (typically `master`) into it.

In an active team you'll often find many PRs being approved in quick succession. The target branch (especially if it's `master`) will then have multiple commits made _after_ the PR was initiated. Some of those changes may cause problems we won't see just by looking at the source branch's code.

Instead of reviewing the code in the source branch we should look at the result of merging both the source and target branches together. This means reviewing the pull request's _merge commit_.

Finding this merge commit isn't overly obvious. In VSTS the UI shows the changes made in the source branch, not the merge commit. There's also no specific branch that points at the merge commit so it's understandable that teams don't realise there's another way to review PRs.

VSTS does, however, maintain a git ref for the PR merge commi and you can even view this merge commit in VSTS, but only by using the "More actions ..." menu item on the PR overview page.

![View a PR merge commit](/assets/images/2018-04/vstsmergecommit.png)

### Bringing the merge commit to your local machine

If you remember, we want to review and test this merge commit locally. We can fetch it from VSTS and check it out as follows:

1. `git fetch origin refs/pull/<PR Number>/merge`
1. `git checkout FETCH_HEAD`

> P.S. If needed, you can easily create a fresh merge commit by restarting the merge. <br />
> ![Restarting a VSTS merge](/assets/images/2018-04/restartmerge.png)

Once you have fetched the merge commit you will be in a _detatched head_ state. Any changes you make will not be committed against a branch, and even if you create a branch and try to push changes back to the server, the push will be rejected. Given you're just reviewing code, that shouldn't be a problem.

>__NOTE:__ If you don't like the *FETCH_HEAD* approach you can create a named local branch for the merge commit using `git fetch origin refs/pull/1234/merge:PR1234` and `git checkout PR1234` instead.

Our git working folder will now have the code from the merge commit in it and we can verify that the result the pull request will be successful. Much better than proving that the source was OK and hoping the result will still be good!

P.S. If you have a build policy on your target branch, the merge commit we're reviewing will be the exact same merge commit used in the VSTS build. 

__UPDATE:__ If you do this regularly, you can make the process even easier. Update your local git fetch settings to grab the pull request refs automatically by running the following command:

`git config --add remote.origin.fetch +refs/pull/*/merge:refs/remotes/origin/pr/*`

You should now be able to do this:
```
git fetch origin
git checkout pr/1234
```

Credit to [Edward Thomson](https://www.edwardthomson.com/blog/checking-out-visual-studio-online-pull-requests-locally.html) for that workflow improvement.
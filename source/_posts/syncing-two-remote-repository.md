title: 'Sync two Git remote repositories'
date: 2015-01-29 00:03:20
categories: [Engineering, Version Control]
tags: Git
---

In Yahoo we use Gerrit as our code review tool. Engineers commit code changes to Gerrit for review. After code has been reviewed by peers, Gerrit help push to Github. However sometimes bad thing happens. 

For example, if a committer forget to setup Gerrit env and the code is committed and pushed to Github directly (because he has this permission to do so), he will notice this soon because future changes from Gerrit might break. Then he try to submit a review for the missing commit to Gerrit with an "git commit --amend" to generate a change-id (which is used by Gerrit). Because "--amend" generate different commit-id, so after the review is passed, even the content is the same, the commit-id in two remote repo (Gerrit and Github) is different, which leads to future reviews are still not able to pushed to Github from Gerrit and sometimes new review could not be submitted. 

So how could we fix it? It would be a good idea to push changes from Github to Gerrit to get them in sync. Here are two options.

<!-- more -->

## first approach

```git
git pull origin master
git push gerrit master --force
```

Usually, the command refuses to update a remote ref that is not an ancestor of the local ref used to overwrite it. This flag disables the check. So latest version from "origin" now could be pushed to "gerrit". By this way, some commit will be lost, but quite straight forward.

## second approach

```
git reset --hard gerrit/master
git merge origin/master
git push gerrit master
```

In this approach, we first make local repo get synced with "gerrit/master", then merge changes with "origin/master". Because we merge changes with "origin/master" so the commit-id of this merge are based on the one on "origin", and both "gerrit" and "origin" are supposed to accept this commit. After we push this merged changes to "gerrit", "gerrit" is able to push it to "origin". Then two repo get synced. By this way, no commit will be lost.


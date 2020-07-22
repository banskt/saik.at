---
title: "How to collaborate on Github and Overleaf"
date: 2020-07-21T11:42:43+02:00
draft: false
---
I manage my manuscripts from [Github](https://github.com/banskt)
(`vim` is my favorite text editor)
and many of my collaborators prefer to work on Overleaf.
However, the Github project generally contains many extra files
including  `jupyter notebooks` for generating figures,
which are not only useless on Overleaf,
but also exceeds the file size limit of Overleaf,
making it impossible to use the default Github sync of Overleaf.

The [raw access to Overleaf git server](https://www.overleaf.com/learn/how-to/How_do_I_connect_an_Overleaf_project_with_a_repo_on_GitHub,_GitLab_or_BitBucket%3F)
allows collaboration on Github and Overleaf simulataneously,
ignoring certain files and directories from Overleaf and / or Github.
I started with creating a new branch on my Github project and called it `overleaf`,
with a new `.gitignore` to manage file access from this branch.
```
git checkout -b overleaf
vim .gitignore
git add .gitignore
git commit -m "Track overleaf branch"
git push origin overleaf
```
I added the overleaf remote in the project (the link can be found in Overleaf settings)
```
git remote add overleaf https://git.overleaf.com/5f155fcr378b2nskt188dc66
git remote -v
```
The `.git/config` file needs manual editing to include the `pushurl` for Overleaf 
and separate tracking for `overleaf` branch of Github:
```
[branch "overleaf"]
    remote = origin
    merge = refs/heads/overleaf
[remote "overleaf"]
    url = https://git.overleaf.com/5f155fcr378b2nskt188dc66
    fetch = +refs/heads/*:refs/remotes/overleaf/*
    pushurl = https://git.overleaf.com/5f155fcr378b2nskt188dc66
```
If I am working on a secure machine, I also add my login credentials for Overleaf:
```
git config credential.helper store
```
Now, I can merge the empty Overleaf project on to my working project,
fix merge conflicts and push it back to Overleaf:
```
git pull overleaf master --allow-unrelated-histories
## fix the merge conflicts here
git commit -m "Start of overleaf collaboration"
git push overleaf overleaf:master
```

## Daily Workflow
On my local computer, I work on the `master` branch of `origin` (Github)
while my collaborators work on Overleaf.
I have to sync and merge the Overleaf and Github versions manually.
This a 3-way sync between the following repositories:

  *  `origin master` (Github `master` branch)
  *  `origin overleaf` (Github `overleaf` branch)
  *  `overleaf master` (Overleaf)

For example, if there are significant changes in Overleaf:
```
# merge Overleaf `master` on to Github `overleaf`
git checkout overleaf
git pull overleaf master
# merge Github `overleaf` on to Github `master`
git checkout master
git merge --no-commit --no-ff overleaf
git commit -m "Merge overleaf onto master"
git push origin master
```
Sometimes, there could be significant changes on Github `master`:
```
git checkout overleaf
git pull overleaf master
git merge --no-commit --no-ff master
git commit -m "Merge master onto overleaf"
git push origin overleaf
git push overleaf overleaf:master
```
Most case, I have to do a combination of both to keep everything updated.

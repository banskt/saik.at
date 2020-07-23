---
title: "Github and Overleaf integration"
date: 2020-04-04T11:42:43+02:00
draft: false
---

I manage my manuscripts on [Github](https://github.com/banskt)
while many collaborators prefer [Overleaf](https://www.overleaf.com).
My Github projects generally contain extra files including large Jupyter Notebooks for generating figures,
which are futile on Overleaf, increase sync time considerably and occasionally exceeds the
[50Mb storage limit](https://www.overleaf.com/learn/how-to/What_is_the_maximum_compilation_time,_file_number_and_project_size_allowed_on_free_vs_paid_plans%3F).
Thus, it is difficult to use the default Github sync feature of Overleaf.
The [raw access to Overleaf git server](https://www.overleaf.com/learn/how-to/How_do_I_connect_an_Overleaf_project_with_a_repo_on_GitHub,_GitLab_or_BitBucket%3F)
allows maintaining the project on Github and Overleaf git servers simulataneously
while ignoring certain files from either server.

The idea is simple: the Github remote server, so called `origin` needs two branches, say `master` and `overleaf`.
The Overleaf remote server, I call it `overleaf`, allows a single branch called `master`.
I push changes in `origin master` and my collaborators push changes in `overleaf master`.
The `origin overleaf` is required as a filter to hide files unnecessary for the other side.
At a regular interval, I merge and sync between the three manually.


The `overleaf` branch on Github requires a separate `.gitignore` file 
to hide unnecessary files on Overleaf.
```bash
git checkout -b overleaf
vim .gitignore
git add .gitignore
git commit -m "Track overleaf branch"
git push origin overleaf
```
Next, I add the overleaf remote (the link can be found in Overleaf settings)
```bash
git remote add overleaf https://git.overleaf.com/5f155fcr378b2nskt188dc66
git remote -v
```
The `.git/config` file needs manual editing to include the `pushurl` for Overleaf 
and separate tracking for `overleaf` branch of Github:
```gitconfig
[branch "overleaf"]
    remote = origin
    merge = refs/heads/overleaf
[remote "overleaf"]
    url = https://git.overleaf.com/5f155fcr378b2nskt188dc66
    fetch = +refs/heads/*:refs/remotes/overleaf/*
    pushurl = https://git.overleaf.com/5f155fcr378b2nskt188dc66
```
If I am working on a secure machine, I also store my login credentials for Overleaf:
```bash
git config credential.helper store
```
Now, I can merge the empty Overleaf project on to my working project,
fix merge conflicts and push it back to Overleaf:
```bash
git pull overleaf master --allow-unrelated-histories
# fix the merge conflicts 
git commit -m "Start of overleaf collaboration"
git push overleaf overleaf:master
```

## Workflow
The three remotes in the project are:

  *  `origin master` (Github `master` branch)
  *  `origin overleaf` (Github `overleaf` branch)
  *  `overleaf master` (Overleaf)

I push changes in `origin master` and my collaborators push changes in `overleaf master`.
Occasionally, I have to merge and push between the three manually.
Merging and syncing between the three always proceeds via `origin overleaf`.
For example, here is how I merge `overleaf master` on to `origin master`:
```bash
# merge overleaf master on to origin overleaf
git checkout overleaf
git pull overleaf master
# fix merge if necessary
git push origin overleaf

# merge origin overleaf on to origin master
git checkout master
git merge --no-commit --no-ff overleaf
git commit -m "Merge overleaf onto master"
git push origin master
```
And here is the opposite:
```bash
git checkout overleaf
git merge --no-commit --no-ff master
git commit -m "Merge master onto overleaf"
git push origin overleaf
git push overleaf overleaf:master
```
However, most of the time, I have to do a combination of the two.

> **Warning**: Merging may accidentally delete files.

## Security note
GitHub public repos with public Overleaf project:
the default log message of merge commit reveals "secret" Overleaf URL, enabling anyone to edit public Overleaf project.
I prefer private Overleaf project.

## Other resources
1. [Git and Overleaf integration by Santiago Casas](https://www.overleaf.com/articles/git-and-overleaf-integration/qmdncpnqwfxx)
2. [Git and Overleaf integration by Jeff Naecker](https://gist.github.com/jnaecker/da8c1846bc414594783978b66b6e8c83)

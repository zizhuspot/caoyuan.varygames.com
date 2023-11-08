---
title: Git is the leading code management tool
date: 2023-11-06 10:04:00
categories: 
  - Technology
tags: 
  - Git
  - Crawler
  - absolutely
  - recognize
  - development
  - framework
  - network
  - selenium
description: Since I've been working, I've been using code collaboration tools, most often Git. But recently I've realized that a lot of people (including myself) are only familiar with pulling and committing code on a day-to-day basis, and don't even know how to use git reset/rebase...
cover: https://s2.loli.net/2023/11/07/MQEvm5VbZtoxKLU.webp
---
### 1. Introduction

Ever since I started working, I've been using code collaboration tools, most often Git.

However, I recently realized that many people (including myself) are only familiar with pulling and committing code, but don't even know how to use `git revert/rebase`.

So I looked up some information and wrote this article, which I believe developers will find useful after reading this article.

### 2. What is Git?

![](https://s2.loli.net/2023/11/07/MQEvm5VbZtoxKLU.webp)

Whether working in a large company or small company coder, or individual developers. Whether it is multi-person collaborative development, or to realize a single upload, multiple downloads, all need to use ** code version control system **.

And Git is the best of the best for code versioning. What, you don't like SVN? Let me list the advantages of Git, how should you respond?

1. Git is distributed versioning, SVN is not. 2;
2. Git content is stored as metadata, whereas SVN uses files;
3. Git uses SHA-1 hash algorithm for content storage, so for content integrity, Git beats SVN. 4;
4. in terms of market share, developers using Git far exceed SVN.

Having said that, I don't want to show off anything, but just let you recognize the status of Git.

Nowadays, if you're a freshman or a hired developer and you're not skilled in Git, you'll have to walk around the company with your head down, so don't ask me how I know (bushi).

### 3. Installation and Configuration

Windows:

> Installation package download address: [gitforwindows.org/](https://gitforwindows.org/)
> 

Mac:

> [sourceforge.net/projects/gi...] (http://sourceforge.net/projects/git- osx-installer/")

In Windows, for example, after installation, you can type "Git" -> "Git Bash" in the start menu to get to the Git window for commands:

![](https://s2.loli.net/2023/11/07/51YOJbi2B4vA9jM.webp)

### 4 Pulling repository code

First, choose a directory as our code repository, that is, the place where we store our code items. Usually, we choose the D drive:

![](https://s2.loli.net/2023/11/07/z16P9eEbX7HDpNC.webp)

Then go to Git and get the repository address, e.g., copy the GitHub repository [github.com/yangfx15/co...] directly. (https://github.com/yangfx15/coder):

![](https://s2.loli.net/2023/11/07/OrqV1yX8GocJ2Wm.webp)

Then run `git clone https://github.com/yangfx15/coder.git` in Git to pull the code and get into the coder directory：

> git clone [github.com/yangfx15/co...](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fyangfx15%2Fcoder.git "https://github.com/yangfx15/coder.git")
cd coder

![](https://s2.loli.net/2023/11/07/xU5HRVqOPJloQe6.webp)

Seeing the `(main)` flag means that the remote code has been downloaded to the local repository!

### 5 Code Branching

When it comes to collaborating on code, naturally code branches are involved.

![](https://s2.loli.net/2023/11/07/OWgzixDU3LX8j9m.webp)

There are four ways to do this: show branches, switch branches, create branches, and delete branches.
**git branch** **List all local branches** **git branch -r** **List all remote branches** **git branch -a** **List all local and remote branches** **git branch** **Create a new branch, but remain in the current branch** **git checkout -b** **Creates a new branch and switches to it** **git branch --track** **Creates a new branch and establishes a tracking relationship with the specified remote branch** **git checkout** **Switches to the specified branch and updates the workspace** **git branch -d** **Deletes the branch** **git push origin --a** **Lists all local and remote branches. * **git push origin --delete** **Deletes the remote branch** **Remote branch deletion

There are a lot of operations on branches, but they are relatively simple to remember.

#### 5.1 Branch Common Operations

Zhangsan and Lisi are co-developers, and they each have personal branches under their `main` master branch: feat_zhangsan, feat_lisi.

> git checkout -b feat_zhangsan
> git checkout -b feat_lisi

Zhangsan develops feature A, Lisi develops feature B. Zhangsan pushes all local code to the remote branch when he's finished developing

> git add .
> git commit -m "feature A"
> git push origin feat_zhangsan
> git branch --set-upstream-to=origin/feat_zhangsan

Read on if you are not sure about these steps.

### 6. Code Push Management

![](https://s2.loli.net/2023/11/07/hdfpz1w3nOBLUP6.webp)

#### 6.1 add

The add command is a simple command that commits changes made in your local workspace to a staging area to be managed by git.
**git add . 

The opposite of add is `reset`, which undoes changes to the staging area.
**git reset . ** **Reset all staging area file changes in the current directory ** **git reset** **Reset a directory, including subdirectories, from the staging area ** **git reset** **Reset a file from the staging area ** **git reset** **Reset a file, including subdirectories, from the staging area ** **git reset

#### 6.2 commit

Commit is a simple command that commits the contents of the staging area to the local repository and moves the HEAD of the current branch back one commit point.
**git commit -m** **Commit the staging area to the local repository, message stands for message ** **git commit -m** **Commit the specified file in the staging area to the local repository ** **git commit --amend -m** **Replace the previous commit with a new one ** **git commit --amend -m** **Replace the previous commit with a new one ** **git commit --amend -m** **Replace the previous commit with a new one

#### 6.3 reset

The opposite of commit is `reset --soft`, which undoes the commit but leaves the written code intact.
**git reset --soft HEAD^** ** to undo the most recent commit, HEAD^ means the previous version, but you can also use HEAD1. If you want to undo the previous two commits, you can use HEAD2** **git reset --hard HEAD^** ** which is similar to `git reset --soft `, except that `hard` undoes the git add at the same time** **where HEAD is the first commit in Git.

HEAD is Git's notion of a commit version that always points to the most recent commit on the branch you're currently on. If your branch changes, or if a new commit point is created, HEAD will change.

What if we want to go back to a commit point?

In this case, we can use `git log --pretty=oneline` to get the commit history:

![](https://s2.loli.net/2023/11/07/Pf2rX64DLENjoIW.webp)

Assuming we want to roll back to the "article link update" commit point, we need to copy the previous commit_id: `cbfa2e854bcc3b06e91682087270fe483be9e37c` and type q to exit.

Then roll back to this commit with `git reset --hard cbfa2e854bcc3b06e91682087270fe483be9e37c`.

#### 6.4 status

On Git, you can check the status of your code with `git status`.![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/XrTaxPsdRHneOF7.webp)

As shown in the figure, when the code is in the workspace, the modified files appear in red; after the code is in the staging area, the modified files appear in green; after the code is committed to the local repository, it shows `nothing to commit, working tree clean`.

#### 6.5 Common Operations

When ZhangSan finishes development on his personal branch, he pushes the code to the remote branch and merges the code from his personal branch into the `main` master branch.

> feat_zhangsan分支：git add .
feat_zhangsan分支：git commit -m "功能A2"
feat_zhangsan分支：git push
feat_zhangsan分支：git checkout main
main分支：git fetch
main分支：git pull
main分支：git merge origin/feat_zhangsan
main分支：git push

### 7. Code merge management

#### 7.1 merge

The merge command merges code from different branches.

![](https://s2.loli.net/2023/11/07/EPtxQHRzTmer5iu.webp)

As shown above, in practice, we may cut out a dev branch from the master branch, and then develop it to complete the requirements, after many commits, and then merge it into the master when the development is completed.
**git fetch** **pulls the latest code from the remote repository before merging, and after fetching, you can add the specified remote branch; if you don't specify it, it defaults to the remote branch of the current branch** **git pull** **keeps the code in the current branch up to date before merging** **git merge** **merges the code from the specified branch to the current branch** **Git merge** **Merge the code from the specified branch into the current branch.

Generally, after merging, there will be a conflict, and you will need to manually resolve the conflict. This is due to multiple users modifying the same area of the same file.

For example, in the above figure, both the v0.2 and dev branches have modified a file on the master branch, and when the dev branch merges into master, you need to resolve the merge conflict.

#### 7.2 rebase

A rebase, also known as a diff & rebase, is an alternative to a merge.

![](https://s2.loli.net/2023/11/07/bAqPawYK1iGRFve.webp)

At the beginning, we're on the dev branch, so if you run git rebase master, any new commits on the dev branch will be repeated on the master branch, and the checkout will switch back to the dev branch. This is the same as merging; the branch you're on doesn't change before or after the merge.

git rebase master means that the dev branch wants to continue on the shoulders of the master.

Like merge, rebase requires manual conflict resolution.

#### 7.3 The Difference Between a Rebase and a Merge

Now we have two branches, dev and master, with the following commits:

```css
    D---E dev
    /
A---B---C---F master
```

Execute git merge dev on master and you will get the following result:

```css
    D-----------E
    /                    \
A---B---C---F----G   master
```

As you can see, the merge operation creates a new node with the previous commits displayed separately,

This is equivalent to a tree growing new branches, which are then merged into the main trunk!

If you have a lot of branches merged in this way, it looks like a mess, and for those with OCD, the commit history from this merge will look really ugly.

At this point, some people may ask: why can't Git's commit history be a clean, straight line? The answer is rebase.

Run git rebase dev on your master, and you'll get the following result:

```css
A---B---D---E---C'---F' master
```

The rebase operation does not generate a new node, it fuses the two branches into a single linear commit.

To summarize:

* If you want a clean, linear history tree with no merge commits, then you should choose git rebase;
* If you want to keep a complete history and want to avoid the risk of rewriting your commit history, you should choose to use git merge.

#### 7.4 revert

git revert undoes a commit, using a new commit to remove the changes made by a history commit.

![](https://s2.loli.net/2023/11/07/14vwuxCUsgRoW53.webp)

When undoing, revert commits a new version that reverses the content of the version that needs to be revert. The HEAD version is then incremented without affecting the previous commit.
**git revert HEAD** **Revert previous commit** **git revert HEAD^** **Revert previous two commits** **git revert {commit_id}** **Pins the specified version, and saves the revert itself as a commit** **Revert is not a commit.

#### 7.5 Difference between revert and reset

![](https://s2.loli.net/2023/11/07/XJ7ukoPMRwrBdtl.webp)

git revert rolls back previous commits with a new commit, and git reset deletes the specified commit.

The effect is similar when you roll it back, but there is a difference when you merge previous versions later. This is because revert adds a new reverse commit, which is equivalent to neutralizing the acid and base, so when you merge with the old branch later, this part of the change won't reappear!

But reset is equivalent to sealing the acid, so when you merge in the future, the reset part of the code will still appear in the history branch, which may cause conflicts.

The difference between the two is equivalent to a chemical reaction and a psychological reaction. Why is this so?

It's because Git, as a code versioning tool, doesn't actually delete the commit every time it's deleted; it seals the commit.

Just like the painful memories you get when you fall out of love, if your brain goes through a shock, you can't delete those memories, you can just seal them. A revert is a way of channeling those painful memories, so that even if you remember them later, they won't be as painful :)

Note that git reset moves the version HEAD back a bit, whereas in git revert the version HEAD moves on.

#### 7.6 Other common commands

**git diff** **Shows the difference between the staging area and the workspace** **git diff HEAD** **Shows the difference between the workspace and the latest commit in the current branch** **git cherry-pick** **Selects a commit and merges it into the current branch** **git rm** **Removes the file from the staging area and the workspace** **git rm** **Removes the file from the staging area and the workspace remove files from staging areas and workspaces** **git mv** **move or rename workspace files** **git blame** **view the history of changes to a given file as a list** **git remote** **remote repository operation

These are some common commands and detailed description of Git, enough to cover all the daily operations of your study and work, I believe that after reading this, you can have a more in-depth understanding of Git.


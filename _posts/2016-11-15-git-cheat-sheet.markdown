---
layout: post
title:  "GIT CHEAT SHEET"
date:   2016-11-14 08:14:54
categories: note
tags: git 教程
excerpt: git 命令快速使用小抄。
author: ChenFeng
---

presented by **[TOWER](https://www.git-tower.com/)** > Version control with Git - made easy

### CREATE

- Clone an existing repository

```git
$ git clone ssh://user@domain.com/repo.git
```

- Create a new local repository

```git
$ git init
```

### BRANCHES & TAGS

- List all existing branches

```git
$ git branch -av
```

- Switch HEAD branch

```git
$ git checkout <branch>
```

- Create a new branch based on your current HEAD

```git
$ git branch <new-branch>
```

- Create a new tracking branch based on a remote branch

```git
$ git checkout --track <remote/branch>
```

- Delete a local branch

```git
$ git branch -d <branch>
```

- Make the current commit with a tag

```git
$ git tag <tag-name>
```

### MERGE & REBASE

- Merge <branch> into your current HEAD

```git
$ git merge <branch>
```

- Rebase yout current HEAD onto <branch>

	*Dont`t rebase published commits!*
	
```git
$ git rebase <branch>
```

- Abort a rebase

```git
$ git rebase --abort
```

- Continue a rebase after resolving conflicts

```git
$ git rebase --continue
```

- Use your configured merge tool to solve conflicts

```git
$ git mergetool
```

- Use yout editor to manually solve conflicts and (after resolving) mark files as resolved

```git
$ git add <resolved-file>
$ git rm <resolved-file>
```


### LOCAL CHANGES

- Changed files in your working directory

```git
$ git status
```

- Changes to tracked files

```git
$ git diff
```

- Add all current changes to the next commit

```git
$ git add .
```

- Add some changes in <file> to the next commit

```git
$ git add -p <file>
```

- Commit all local changes in tracked files

```git
$ git commit -a
```

- Commit previously staged changes

```git
$ git commit
```

- Change the last commit 
	*Don‘t amend published commits!*

```git
$ git commit --amend
```

### UPDATE & PUBLISH

- List all currently configured remotes

```git
$ git remote -v
```

- Show information about a remote

```git
$ git remote show <remote>
```

- Add new remote repository, named <remote>

```git
$ git remote add <shortname> <url>
```

- Download all changes from <remote>, but don‘t integrate into HEAD

```git
$ git fetch <remote>
```

- Download changes and directly merge/integrate into HEAD

```git
$ git pull <remote> <branch>
```

- Publish local changes on a remote

```git
$ git push <remote> <branch>
```

- Delete a branch on the remote

```git
$ git branch -dr <remote/branch>
```

- Publish your tags

```git
$ git push --tags
```

### UNDO

- Discard all local changes in your working directory

```git
$ git reset --hard HEAD
```

- Discard local changes in a specific file

```git
$ git checkout HEAD <file>
```

- Revert a commit (by producing a new commit with contrary changes)

```git
$ git revert <commit>
```

- Reset your HEAD pointer to a previous commit ...and discard all changes since then

```git
$ git reset --hard <commit>
```

- ...and preserve all changes as unstaged changes

```git
$ git reset <commit>
```

- ...and preserve uncommitted local changes

```git
$ git reset --keep <commit>
```

### COMMIT HISTORY

- Show all commits, starting with newest

```git
$ git log
```

- Show changes over time for a specific file

```git
$ git log -p <file>
```

- Who changed what and when in <file>

```git
$ git blame <file>
```

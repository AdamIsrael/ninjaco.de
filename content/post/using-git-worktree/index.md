---
author: Adam
categories:
- Technical
date: "2018-09-21T13:12:02+02:00"
draft: false
hidden: false
tags:
- git
title: Using Git Worktree
description: "This handy trick has saved me hours and countless headaches when managing multiple feature branches."
image: "img/git.png"
---

Often enough, you're at the stage in your work where you're running potentially time-consuming unit/integration tests. You have other work to do on that project, but you're tied up waiting for the tests to finish (and hopefully pass).

I wanted to figure out an efficient way to work on new features while I'm testing another. You can't switch the branch out from underneath the test, and making a copy/clone of the repository is an expensive operation (in disk space and/or network speed, especially with larger projects).
<!--more-->

And I found [git-worktree](https://git-scm.com/docs/git-worktree):

> A git repository can support multiple working trees, allowing you to check out more than one branch at a time. [...]  a new working tree is associated with the repository, [...] called a "linked working tree" as opposed to the "main working tree" prepared by "git init" or "git clone". A repository has one main working tree[...] and zero or more linked working trees.

## Example

Here's what it looks like in practice.

### Create a new project

Create a new, empty project to work from (if you're brave, skip this step and work against an existing repo):

```bash
mkdir -p ~/Demo/myproject
cd ~/Demo/myproject
git init .
touch README.md
git add README.md
git commit -a -m "Initial response"
```

### Create a Feature Branch and worktree

The `master` branch should only contain code ready for general consumption. I use branches to work on new code, and merge when ready.

```bash
cd ~/Demo/myproject

# Checkout the branch you want to base your new work on
git checkout master

# Create a new branch for this mythical new work
git branch feature-24601

# Create a new worktree, in the Demo directory, for our new feature branch
git worktree add ../myproject-24601 feature-24601

# Switch to the directory with the new worktree
cd ../myproject-24601
# Confirm that we're in our new branch
git status
> On branch feature-24601
```

### Hack/Hack/Hack

Now you're free to hack on your new feature without stomping over your running tests.

```bash
# Write code, test, and documentation
echo "Adding feature 24601." >> README.md
git commit -a -m "Document new feature"
```

### Merge feature branch

Switch back to master and merge your new feature.

```bash
cd ../myproject
git merge feature-24601
Updating 873ef32..2da1c53
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
```

### Cleaning up

When you're done, clean up your repository. Future you will be happier.

```bash
# `prune` will remove worktrees pointing to directories that no longer exist
rm -rf ../myproject-24601
git worktree prune

# Delete the feature branch
git branch -d feature-24601
> Deleted branch feature-24601 (was 2da1c53).
```

## Caveats

- Mixing worktree and submodules works but with the caveat that each worktree has a unique copy of submodules, rather than a hard link, so this will consume more disk space.
- You can't create multiple worktrees for the same branch, but that's generally not a problem if you're following best git practices.

## Further Reading

- [Parallelize Development Using Git Worktrees](https://spin.atomicobject.com/2016/06/26/parallelize-development-git-worktrees/), which I read to understand the basics of git-worktree.
- [What goes wrong when using git worktree with git submodules](https://stackoverflow.com/questions/31871888/what-goes-wrong-when-using-git-worktree-with-git-submodules)
- [Git Feature Branch Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow)

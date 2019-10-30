---
title: Feature branch workflow
nav_order: 2
parent: Agile
grand_parent: Engineering
---

# Feature branch workflow

The idea behind using feature branches is that feature development takes place in a dedicated branch rather than disturbing the base branch. It also allows different contributors of the project to work on a same feature. The feature branch is only merged to the base branch after the PR is approved by peers. This also encourages collaborative workflow in the project.

Ideally it is encouraged to follow this branch structure:

`develop` -> Base branch for development  
`staging` -> Staging environment  
`beta`       -> Beta environment (optional)   
`master`   -> Production environment  

In this guide, `develop` is considered as the base branch.

## How it works

The Feature branch workflow insists instead of committing directly on their local `develop` branch, developers create a new branch every time they start work on a new feature. Feature branches should have descriptive names, like animated-menu-items or issue-#1061. The idea is to give a clear, highly-focused purpose to each branch. Git makes no technical distinction between the `develop` branch and feature branches, so developers can edit, stage, and commit changes to a feature branch.

The following is a walk-through of the life-cycle of a feature branch.

### Start with the develop branch

All feature branches are created off the latest code state of a project.

    git checkout develop  
    git fetch origin 
    git reset --hard origin/develop

This switches the repo to the `develop` branch, pulls the latest commits and resets the repo's local copy of `develop` to match the latest version.

### Create a new-branch

Use a separate branch for each feature or issue you work on. After creating a branch, check it out locally so that any changes you make will be on that branch.

```
git checkout -b new-feature
```

This checks out a branch called new-feature based on  `develop`, and the -b flag tells Git to create the branch if it doesn’t already exist.

### Update, add, commit, and push changes

On this branch, edit, stage, and commit changes in the usual fashion, building up the feature with as many commits as necessary. Work on the feature and make commits like you would any time you use Git.

```
git status
git add <some-file>
git commit
```
## Git rebase
 
Rebasing is the process of moving or combining a sequence of commits to a new base commit. Rebasing is most useful and easily visualized in the context of a feature branching workflow. The general process can be visualized as the following:

[![feature-branch-workflow page](/assets/images/feature-branch-workflow.svg)](/assets/images/feature-branch-workflow.svg)
###### *In the image above `master` is considered as base branch



From a content perspective, rebasing is changing the base of your branch from one commit to another making it appear as if you'd created your branch from a different commit.

The primary reason for rebasing is to maintain a linear project history. For example, consider a situation where the `develop` branch has progressed since you started working on a feature branch. You want to get the latest updates to the `develop` branch in your feature branch, but you want to keep your branch's history clean so it appears as if you've been working off the latest `develop` branch.

A more real-world scenario to use `git rebase` would be:

1.  A bug is identified in the master branch. A feature that was working successfully is now broken.
2.  A developer examines the history of the master branch using  `git log`  because of the "clean history" the developer is quickly able to reason about the history of the project.

You have two options for integrating your feature into the master branch: merging directly or rebasing and then merging. The former option results in a 3-way merge and a merge commit, while the latter results in a fast-forward merge and a perfectly linear history. 

Git rebase in standard mode will automatically take the commits in your current working branch and apply them to the head of the passed branch.

```
git rebase develop
```

This automatically rebases the current branch onto  `develop`, which can be any kind of commit reference (for example an ID, a branch name, a tag, or a relative reference to  `HEAD`).

Running  `git rebase`  with the  `-i`  flag begins an interactive rebasing session. Instead of blindly moving all of the commits to the new base, interactive rebasing gives you the opportunity to alter individual commits in the process. This lets you clean up history by removing, splitting, and altering an existing series of commits. 
```
git rebase --interactive develop
```
This rebases the current branch onto  `develop`  but uses an interactive rebasing session. This opens an editor where you can enter commands (described below) for each commit to be rebased. These commands determine how individual commits will be transferred to the new base.

 The rebasing edit commands are as follows:

```

pick 2231360 some old commit
pick ee2adc2 Adds new feature
# Rebase 2cf755d..ee2adc2 onto 2cf755d (9 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
```

### Push feature branch to remote

 Push the feature branch up to the central repository
```
git push -u origin new-feature
```

This command pushes new-feature to the central repository (origin), and the -u flag adds it as a remote tracking branch. After setting up the tracking branch,  `git push`  can be invoked without any parameters to automatically push the new-feature branch to the central repository. 

To get feedback on the new feature branch, create a pull request in Git.

### Resolve feedback

Now teammates comment and approve the pushed commits. Resolve their comments locally, commit, and push the suggested changes to Git. Your updates appear in the pull request.

### Merge your pull request

Before you merge, you may have to resolve merge conflicts if others have made changes to the repo. When your pull request is approved and conflict-free, you can add your code to the  `develop`  branch. Merge from the pull request in Github.





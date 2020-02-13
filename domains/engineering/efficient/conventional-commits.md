---
title: Conventional Commits
nav_order: 3
parent: Efficient
grand_parent: Engineering
---

# Conventional Commits templates

The Conventional Commits specification is a lightweight convention on top of commit messages. It provides an easy set of rules for creating an explicit commit history; which makes it easier to write automated tools on top of. This convention dovetails with SemVer, by describing the features, fixes, and breaking changes made in commit messages.

1. ToC
{:toc}

### Why Use Conventional Commits

Automatically generating CHANGELOGs

Automatically determining a semantic version bump (based on the types of commits landed)

Communicating the nature of changes to teammates, the public, and other stakeholders

Triggering build and publish processes

Making it easier for people to contribute to your projects, by allowing them to explore a more structured commit history


### How to use

Hooks reside in the .git/hooks dir of every git repo. The `.sample` extension prevents them from executing by default. To install then you need to remove the  `.sample`  extension.

To make the hook executable you can simply run this comand

    chmod +x prepare-commit-msg

### Pre commit msg hook

[https://github.com/viveckadhriyaa/pre-commit-message-hook](https://github.com/viveckadhriyaa/pre-commit-message-hook)
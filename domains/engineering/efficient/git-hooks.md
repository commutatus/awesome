# Git hooks templates

Git hooks are scripts that run automatically every time a particular event occurs in a Git repository. They let you customize Git’s internal behavior and trigger customizable actions at key points in the development life cycle.

### How it works

We have created an open-source project where we aim to create and maintain basic templates for hooks in different languages(currently python) so that different project can use the template according to their need.

[https://github.com/commutatus/git-hooks-templates.git](https://github.com/commutatus/git-hooks-templates.git)

### How to use

Hooks reside in the .git/hooks dir of every git repo. The `.sample` extension prevents them from executing by default. To install then you need to remove the  `.sample`  extension.

As an example, try installing a simple `prepare-commit-msg` hook. Remove the `.sample` extension from this script, and add the following to the file:

    #!/bin/sh
    echo "# Please include a useful commit message!" > $1

Hooks need to be executable, so you may need to change the file permissions of the script if you’re creating it from scratch. For example, to make sure that `prepare-commit-msg` is executable, you would run the following command:

    chmod +x prepare-commit-msg

You should now see this message in place of the default commit message every time you run `git commit`. We’ll take a closer look at how this actually works in the Prepare Commit Message section. For now, let’s just revel in the fact that we can customize some of Git’s internal functionality.

The built-in sample scripts are very useful references, as they document the parameters that are passed in to each hook (they vary from hook to hook).

You can use the default hooks or replace the hooks file with the template file.

### How to manage hooks across teams

Change the default hooks dir path

Change the git hook directory for git > 2.9

    git config core.hooksPath dirname
    
    e.g. git config core.hooksPath ./customHooks

Note: 
Every member has to change git hooks dir path manually.

### How can you contribute to the project

1. Create a folder if its not there. e.g. githooks<language-name> 
2. Create or update the hook file.
3. Make the file executable.
4. Write proper comments.
5. Thats it.

### References

[https://www.atlassian.com/git/tutorials/git-hooks](https://www.atlassian.com/git/tutorials/git-hooks)

[https://github.com/git/git/tree/master/templates](https://github.com/git/git/tree/master/templates)
---
title: Conventional Commits
nav_order: 4
parent: Well documented
grand_parent: Engineering
---

# Conventional Commits

The Conventional Commits specification is a lightweight convention on top of commit messages. It provides an easy set of rules for creating an explicit commit history; which makes it easier to write automated tools on top of. This convention dovetails with SemVer, by describing the features, fixes, and breaking changes made in commit messages.

1. ToC
{:toc}

### Why Use Conventional Commits

Automatically generating CHANGELOGs

Automatically determining a semantic version bump (based on the types of commits landed)

Communicating the nature of changes to teammates, the public, and other stakeholders

Triggering build and publish processes

Making it easier for people to contribute to your projects, by allowing them to explore a more structured commit history

### Types of conventions

<b>feat:</b> 	  Must be used when a commit adds a new feature to your application or library.

<b>fix:</b> 	  MUST be used when a commit represents a bug fix for your application.

<b>style:</b> 	  Feature and updates related to styling

<b>refactor:</b>  Refactoring a specific section of the codebase focused on readability, style and/or performance.

<b>test:</b>      Everything related to testing

<b>docs:</b>      Everything related to documentation

<b>chore:</b>     Regular code maintenance.[You can also use emojis to represent commit types]

<b>HOTFIX:</b>    If any urgent fixes required.

### Rules

Commits must be prefixed with a type, which consists of a noun, feat, fix, etc., followed by an optional scope, and a required terminal colon and space.

Commit format should be <code>keyword(scope): [ticket-id] </code>

Commit must contain Ticket id.

If custom keywords are used keyword should be in a single word.

Scope may be provided after a type. A scope must consist of a noun describing a section of the codebase surrounded by parenthesis, e.g., fix(parser):

A description must immediately follow the space after the type/scope prefix. The description is a short summary of the code changes, e.g., fix: array parsing issue when multiple spaces were contained in string.

A longer commit body may be provided after the short description, providing additional contextual information about the code changes. The body must begin one blank line after the description.

A commit body is free-form and may consist of any number of newline separated paragraphs.

One or more footers may be provided one blank line after the body. Each footer MUST consist of a word token, followed by either a <b>:</b> or <b>#</b> separator, followed by a string value (this is inspired by the git trailer convention).

A footer’s value may contain spaces and newlines, and parsing must terminate when the next valid footer token/separator pair is observed.

A hotfix must consist of the uppercase text HOTFIX, followed by a colon and a space.

A description must be provided after the HOTFIX:, describing what the change is about, e.g., HOTFIX: environment variables now take precedence over config files.

### How to use

Hooks reside in the .git/hooks dir of every git repo. The `.sample` extension prevents them from executing by default. To install them you need to remove the  `.sample`  extension.

To make the hook executable you can simply run this comand

    chmod +x prepare-commit-msg

Once the hook made executable copy and paste the code below, which checks for conventional commit messages.

### Pre commit msg hook

Pre-commit msg hook that checks for conventional-commits

```ruby
#!/usr/bin/ruby
# Encoding: utf-8

message_file = ARGV[0]

current_branch = %x(git rev-parse --abbrev-ref HEAD).sub("\n","")

# repo_url = %x(git config --get remote.origin.url).sub(".git\n", '')

def check_format_rules(line_number, line)
  conventions = ['feat', 'fix', 'chore', 'install', 'improvement', 'ci', 'ui', 'style', 'change']
  conventional_commit_conventions = [ 'feat(.*): \[\w+\D\-\d+\] ', 'fix(.*): \[\w+\D\-\d+\] ', 'chore(.*): \[\w+\D\-\d+\] ', 'install(.*): \[\w+\D\-\d+\] ', 'improvement(.*): \[\w+\D\-\d+\] ', 'ci(.*): \[\w+\D\-\d+\] ', 'ui(.*): \[\w+\D\-\d+\] ', 'style(.*): \[\w+\D\-\d+\] ', 'change(.*): \[\w+\D\-\d+\] ']  
  conventional_commit_check = conventional_commit_conventions.map{|x| line.match(x)}.compact
  errors = []
  keyword = line.split('(')[0]
  if conventional_commit_check.empty?
    unless line.include?('HOTFIX')
      if conventions.include?(keyword.lstrip.rstrip)
        return errors << "Error : Your commit message seems like not following conventional commit rules, please check your commit's convention"
      end
      errors << "Error : Your custom commit doesn't seem like following conventional commit rules" if (!conventions.include?(keyword) && line.match('(.*): \[\w+\D\-\d+\] ').nil?)
    end
  end
  errors << "Error : Keyword neither should be in multiple words nor contain any trailing space." if keyword.match(" ")
  errors << "Error : Your commit message contains #{line.length} characters. Commit message should be less than 72 characters in length." if line.length > 72
  errors << "Error : Your subject contains #{line.split(':')[1].length} characters. Subject should be less than 50 characters" if line.split(']')[1]&.length.to_i > 50
  errors << "Error : Commit message subject should start in Capital." if line.split(']')[1] && line.split(']')[1].lstrip[0] == line.split(']')[1].lstrip[0].downcase
  return errors
end

def check_is_conventional(message_file)
  errors = []
  File.open(message_file, 'r').each_with_index do |line, line_number|
    errors = check_format_rules line_number, line.strip
  end
  return errors
end

def error_block(errors)
  print("\n<--------------------------✋ Invalid commit format. exiting commit---------------------->\n")
  print("\n")
  print errors.join("\n")
  print("\n")
  print("\n")  
end

def validate_commit(current_branch, message_file)
  if current_branch == "master"
    print("\n")
    print("Failed to commit\n")
    print("\n")
    puts "✋ [PREVENT] --> Non shall commit to master!"
    print("\n")
    exit 1
  else
    commit_errors = check_is_conventional(message_file)
    if commit_errors.empty?
      exit 0
    else
      error_block(commit_errors)
      puts 'Press y to edit and n to cancel the commit. [y/n]'    
      choice = $stdin.gets.chomp
      if %w(no n).include?(choice.downcase)
        exit 1
      elsif %w(yes y).include?(choice.downcase)
        puts "Please make your new commit :"
        edit = $stdin.gets.chomp
        File.open(message_file, 'w') do |file|
          file.write edit
        end
        validate_commit(current_branch, message_file)
      else
        exit 1
      end
    end
  end
end

while true
  validate_commit(current_branch, message_file)
end
```

### References

[https://www.conventionalcommits.org/en/v1.0.0/](https://www.conventionalcommits.org/en/v1.0.0/)

[https://chris.beams.io/posts/git-commit/](https://chris.beams.io/posts/git-commit/)

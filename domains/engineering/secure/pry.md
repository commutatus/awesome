---
title: Using Pry in production
nav_order: 6
parent: Secure
grand_parent: Engineering
---

# Using Pry in production

[Pry](http://pryrepl.org/) is better than irb for a [number of reasons](https://banisterfiend.wordpress.com/2011/01/27/turning-irb-on-its-head-with-pry/), but chief among them are that it syntax highlights input and output, and it crashes less often. Here are some simple instructions to get you started.

## 1. Include pry in the Gemfile

Add pry to the production part of the Gemfile.

```ruby
# Gemfile  

gem 'pry-rails'  
```
Once you’ve changed the Gemfile, run bundle and then deploy. Pry will be available in production.

## 2. Change the prompt in production

Add a rails initializer that sets the prompt in production.

```ruby
# config/initializers/pry.rb
# Show red environment name in pry prompt for non development environments
unless Rails.env.development?
  old_prompt = Pry.config.prompt
  env = Pry::Helpers::Text.red(Rails.env.upcase)
  Pry.config.prompt = [
    proc {|*a| "#{env} #{old_prompt.first.call(*a)}"},
    proc {|*a| "#{env} #{old_prompt.second.call(*a)}"},
  ]
end
```

## 3. Wrap it in a script

You need to remember which directory to run it from, to use bundle exec, and to pass production as an argument.

To make this easier to do add a production-console command to your repository. All it does is open pry with all the options set.

```bash
#/usr/local/bin/production-console
cd /apps/bugsnag-website/current
bundle exec rails console production
```

## 4. Designate a ‘pry’ machine

To ssh into the machine, have an alias defined in your ~/.zshrc that lets you run production-console on your laptop.

```bash
# ~/.zshrc
alias production-console='ssh -t monitor zsh -l -c production-console'
```

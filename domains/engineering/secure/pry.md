---
title: Using Pry in production
nav_order: 6
parent: Secure
grand_parent: Engineering
---

# Using Pry in production


## 1. Include pry in the Gemfile

```
# Gemfile  
  
gem 'pry-rails'  
gem 'pry-plus', group: :development
```

Once Gemfile is changed, run bundle and then deploy. Pry will be available in production.


## 2. Change the prompt in production

```
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

Now it’s obvious when you’re in production :).


[![pry](/assets/images/pry.png)](/assets/images/pry.png)


## 3. Wrap it in a script

Running pry in production is surprisingly fiddly, particularly if we’re trying to do it to answer an urgent question. We need to remember which directory to run it from, to use bundle exec, and to pass production as an argument.

To make this easier to do we added a production-console command. All it does is open pry with all the options set.

```
#/usr/local/bin/production-console

cd /apps/bugsnag-website/current
bundle exec rails console production
```


## 4. Designate a ‘pry’ machine

The monitor machine usually spends its time doing non-critical things, like running [monit](http://mmonit.com/monit/), [graphite](http://graphite.wikidot.com/), and [cron](https://en.wikipedia.org/wiki/Cron). As some of the cron jobs we dorequire our rails codebase, production-console already worked on that machine, so it was the obvious choice.

To ensure that we don’t forget which machine to ssh into, we have an alias defined in our ~/.zshrc that lets us run production-console on our laptop.

```
# ~/.zshrc

alias production-console='ssh -t monitor zsh -l -c production-console'
```

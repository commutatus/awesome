---
title: Using Pry in production
nav_order: 6
parent: Secure
grand_parent: Engineering
---

# Using Pry in production without shooting yourself in the foot

Bugsnag has been using [pry](http://pryrepl.org/) as a replacement for ruby’s irb console since before I joined (disclaimer: I’m one of the pry core team). It’s better than irb for a [number of reasons](https://banisterfiend.wordpress.com/2011/01/27/turning-irb-on-its-head-with-pry/), but chief among them are that it syntax highlights input and output, and it crashes less often.


This is most useful in development, when you spend a considerable amount of time in the console, but it’s also useful in production when you need to analyze (or even fix) production data. Here are some simple instructions to get you started.


## 1. Include pry in the Gemfile

Making pry work in production is easy, you just add it to the production part of the Gemfile. We also use [pry-plus](https://github.com/rking/pry-plus), but we restrict that to development because it’s less useful in production and some of the plugins are less well tested than pry itself.
```
# Gemfile  
  
gem 'pry-rails'  
gem 'pry-plus', group: :development
```

Once you’ve changed the Gemfile, run bundle and then deploy. Pry will be available in production.


## 2. Change the prompt in production

Using the same console in development as production had an unforeseen problem. The prompt looked the same wherever you were. This led to code being run in production that was intended for development.

No permanent damage was done, but we decided that we should make it really really obvious which environment you’re running in. To this end, we added a rails initializer that sets the prompt to something scary in production.

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

Running pry in production is surprisingly fiddly, particularly if you’re trying to do it to answer an urgent question. You need to remember which directory to run it from, to use bundle exec, and to pass production as an argument.

To make this easier to do we added a production-console command to our chef recipes. All it does is open pry with all the options set.

```
#/usr/local/bin/production-console

cd /apps/bugsnag-website/current
bundle exec rails console production
```


## 4. Designate a ‘pry’ machine

This solved most of our problems, until one afternoon we noticed that one of our web servers was performing significantly less well than the others. I logged into the machine to have a look around, and I found that a large portion of its CPU and RAM were being used by a pry session someone had left in a screen.

Now we only ever use pry on our monitor machine. The monitor machine usually spends its time doing non-critical things, like running [monit](http://mmonit.com/monit/), [graphite](http://graphite.wikidot.com/), and [cron](https://en.wikipedia.org/wiki/Cron). As some of the cron jobs we dorequire our rails codebase, production-console already worked on that machine, so it was the obvious choice.

To ensure that I don’t forget which machine to ssh into, I have an alias defined in my ~/.zshrc that lets me run production-console on my laptop.

```
# ~/.zshrc

alias production-console='ssh -t monitor zsh -l -c production-console'
```

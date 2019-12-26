---
title: Set up Rails worker queue
nav_order: 7
parent: Efficient
grand_parent: Engineering
---


# Setting up a sidekiq - redis worker server in Rails and Cloud66

### Project configuration

1. Add `gem 'sidekiq'` to your project's `Gemfile`
2.  Create a `Procfile` at your project's root directory with the following contents
  ```
  worker: bundle exec sidekiq -e $RAILS_ENV
  ```
3. Add the following line to `config/environments/production.rb`. Do the same for `staging`
  ```ruby
  config.active_job.queue_adapter = :sidekiq
  ```
4. **For Rails 6+:** Create a new `config/sidekiq.yml` file with the following contents
  ```yml
  :queues:
  - default
  - active_storage_analysis
  - active_storage_purge
  ```
  This will ensure jobs launched by ActiveStorage won't sit idle forever. If you are using any queue other than `default` in any of your jobs. make sure to add it here as well

* *optional:* To track sidekiq's performance, add the following to the `config/routes.rb` file:
  ```ruby
  require 'sidekiq/web'
  mount Sidekiq::Web => '/sidekiq'
  ```

### Cloud66 configuration

#### Redis

1. In the Cloud66 main app screen, click on the large **+** sign besides **Add-Ins**, and select **Redis**
2. Depending on your environment
  * Staging: Select **Existing Server** and click on **Add**
  * Production: Select **New Cloud Server** and choose one according to your needs. Ask your team leader for guidance if required.

#### Process server
Once you have pushed and redeployed, a new **Process Server** will appear on the Cloud66 stack.

1. Click on **Process Server**
2. Depending on your environment
  * Staging: Click on the green **+** button (Scale up this process group)
  * Production: Select **New Process Server** and choose one according to your needs. Ask your team leader for guidance if required. Then do the same as staging

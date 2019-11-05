---
title: Rollbar and Travis integration
nav_order: 1
parent: Agile
grand_parent: Engineering
---
Here is the documentation to set up CI with Travis and Rollbar to your project!

1. ToC
{:toc}

## Setting up Travis
Add these lines to your Gemfile:
```ruby
gem 'travis'
gem 'rspec'
```
Then run: `bundle install`

You can either create the travis.yml file or run `travis init`


create a file called `config/database.yml.travis` with the following contents

```yaml
test:
  adapter: postgresql
  database: travis_ci_test
```
This is how travis.yml file should look:

```yaml
addons:
  postgresql: '10'
  apt:
    packages:
    - postgresql-server-dev-10
language: ruby
cache: bundler
before_script:
- psql -c 'create database travis_ci_test;' -U postgres
- cp config/database.yml.travis config/database.yml
script:
- bundle exec rake db:test:prepare
- bundle exec rspec spec
services:
- redis-server
deploy:
- provider: cloud66
  redeployment_hook:
  	secure: place your encrypted key
  on:
    branch: staging
- provider: cloud66
  redeployment_hook:
  	secure: place your encrypted key
  on:
    branch: master
branches:
  only:
  - master
  - staging
```

## Rollbar
1. Find the rollbar access token and add it [as a Rails secure credential]({% link domains/engineering/secure/rails_secure_credentials.md %}) under `rollbar/access_token`
2. Add the following content to `config/initializers/rollbar.rb`

```ruby
Rollbar.configure do |config|
  # Without configuration, Rollbar is enabled in all environments.
  # To disable in specific environments, set config.enabled=false.

  config.access_token = Rails.application.credentials.dig(:rollbar, :access_token)
	ENV['ROLLBAR_ACCESS_TOKEN'] = config.access_token # so the script in the second part of the deployment works properly

  # Here we'll disable in 'test':
  if Rails.env.development? || Rails.env.test?
    config.enabled = false
  end

  # By default, Rollbar will try to call the `current_user` controller method
  # to fetch the logged-in user object, and then call that object's `id`
  # method to fetch this property. To customize:
  # config.person_method = "my_current_user"
  # config.person_id_method = "my_id"

  # Additionally, you may specify the following:
  # config.person_username_method = "username"
  # config.person_email_method = "email"

  # If you want to attach custom data to all exception and message reports,
  # provide a lambda like the following. It should return a hash.
  # config.custom_data_method = lambda { {:some_key => "some_value" } }

  # Add exception class names to the exception_level_filters hash to
  # change the level that exception is reported at. Note that if an exception
  # has already been reported and logged the level will need to be changed
  # via the rollbar interface.
  # Valid levels: 'critical', 'error', 'warning', 'info', 'debug', 'ignore'
  # 'ignore' will cause the exception to not be reported at all.
  # config.exception_level_filters.merge!('MyCriticalException' => 'critical')
  #
  # You can also specify a callable, which will be called with the exception instance.
  # config.exception_level_filters.merge!('MyCriticalException' => lambda { |e| 'critical' })

  # Enable asynchronous reporting (uses girl_friday or Threading if girl_friday
  # is not installed)
  # config.use_async = true
  # Supply your own async handler:
  # config.async_handler = Proc.new { |payload|
  #  Thread.new { Rollbar.process_from_async_handler(payload) }
  # }

  # Enable asynchronous reporting (using sucker_punch)
  # config.use_sucker_punch

  # Enable delayed reporting (using Sidekiq)
  # config.use_sidekiq
  # You can supply custom Sidekiq options:
  # config.use_sidekiq 'queue' => 'default'

  # If your application runs behind a proxy server, you can set proxy parameters here.
  # If https_proxy is set in your environment, that will be used. Settings here have precedence.
  # The :host key is mandatory and must include the URL scheme (e.g. 'http://'), all other fields
  # are optional.
  #
  # config.proxy = {
  #   host: 'http://some.proxy.server',
  #   port: 80,
  #   user: 'username_if_auth_required',
  #   password: 'password_if_auth_required'
  # }

  # If you run your staging application instance in production environment then
  # you'll want to override the environment reported by `Rails.env` with an
  # environment variable like this: `ROLLBAR_ENV=staging`. This is a recommended
  # setup for Heroku. See:
  # https://devcenter.heroku.com/articles/deploying-to-a-custom-rails-environment
  config.environment = ENV['ROLLBAR_ENV'].presence || Rails.env
end

```

### Rollbar deploy tracking feature via Travis
To keep track of deployements in rollbar:

- Add this code to your travis.yml file

```yaml
after_deploy:
- ./.rollbar.sh succeeded
before_deploy:
- export APPLICATION_NAME=application name
- export DEPLOYENV=$(if [ "${TRAVIS_BRANCH}" == "staging" ]; then echo "staging";
  else echo "production"; fi)
- export AUTHOR_NAME="$(git log -1 $TRAVIS_COMMIT --pretty="%aN")"
- ./.rollbar.sh started
- export DEPLOY_ID="$(while read -r line; do echo "$line"; done < ~/deploy_id_from_rollbar)"
```

- create a new file called .rollbar.sh under project folder with the following content:

```sh
function start_deployment() {
  deploy_id=`curl --request POST --url https://api.rollbar.com/api/1/deploy/ \
  			--form access_token=$ROLLBAR_ACCESS_TOKEN \
  			--form environment=$DEPLOYENV \
  			--form revision=$TRAVIS_COMMIT \
  			--form comment="Deployment started in $DEPLOYENV for $APPLICATION_NAME" \
  			--form status=started \
  			--form local_username=$AUTHOR_NAME | python -c 'import json, sys; obj = json.load(sys.stdin); print obj["data"]["deploy_id"]'`
  echo "$deploy_id" >> ~/deploy_id_from_rollbar
}


function set_deployment_success() {
  curl --request PATCH --url "https://api.rollbar.com/api/1/deploy/$DEPLOY_ID?access_token=$ROLLBAR_ACCESS_TOKEN" --data '{"status": "succeeded"}'
}

if [ "$1" == "started" ]
then
  echo "$1"
  start_deployment
else
  set_deployment_success
fi
```

- Run `$ chmod 755 home/pathname/.rollbar.sh` to make the file executable:

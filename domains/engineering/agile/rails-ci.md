---
title: Rollbar and Travis integration
nav_order: 1
parent: Agile
grand_parent: Engineering
---
Here is the documentation to set up CI with Travis and Rollbar to your project!

Add this line to your Gemfile:
**gem 'travis'**
**gem 'rspec'**

Then run:
**bundle install**

You can either create the travis.yml file or run the below command and it will create it for you:
**travis init**

create a file called **config/database.yml.travis** and place the code

	test:
	  adapter: postgresql
	  database: database_name

This is how travis.yml file should look

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

**Rollbar deploy tracking feature via Travis**
To keep track of deployements in rollbar, add the code to your travis.yml file

	after_deploy:
	- ./.rollbar.sh succeeded
	ROLLBAR_ACCESS_TOKEN = "your_rollbar_access_token"
	before_deploy:
	- export APPLICATION_NAME=application name
	- export DEPLOYENV=$(if [ "${TRAVIS_BRANCH}" == "staging" ]; then echo "staging";
	  else echo "production"; fi)
	- export AUTHOR_NAME="$(git log -1 $TRAVIS_COMMIT --pretty="%aN")"
	- ./.rollbar.sh started
	- export DEPLOY_ID="$(while read -r line; do echo "$line"; done < ~/deploy_id_from_rollbar)"

create a new file called .rollbar.sh under project folder and add the lines to the files

Run this command and commit it to give the file permission to be executable
$ chmod 755 home/pathname/.rollbar.sh

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

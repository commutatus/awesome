---
title: Automatic db backups
nav_order: 2
parent: Secure
grand_parent: Engineering
---

The objective here is to automate database backups with a cron job and keep the backups secure remotely on an s3 bucket. 

1. __Installation__:
<br/>
Install the [backup gem](https://github.com/backup/backup) on the machine. Versions other than `5.0.0.beta.1` versions can return an error with some json version. ssh onto your server and run
	`gem install backup -v '5.0.0.beta.1'`

2. __Configuration__:
<br/> 
Generate a model for the database. This model file will have the type and configuration of storage, database, encryption, compression, notifier etc. for the database. 

	`backup generate:model --trigger <model name> \
	    --databases="postgresql" --storages="s3"  \
	    --compressor="gzip" --notifiers="slack"`

	Refer [this page](https://backup.github.io/backup/v4/generator/) for a detailed overview about the types of customizations available. You can omit any configuration that you don't need here on the command above. This will create a `/Users/<username>/Backup` (macOS) directory by default (`~/Backup` on a linux machine). The config file can be found at `/Users/<username>/Backup/models/<you-model-name>.rb` or linux equivalent.

	
	```ruby
	Model.new(:<Project Environment>, <Project name with db type>) do
		database PostgreSQL do |db|
	    db.name               = ENV["POSTGRESQL_DATABASE"]
	    db.username           = ENV["POSTGRESQL_USERNAME"]
	    db.password           = ENV["POSTGRESQL_PASSWORD"]
	    db.host               = ENV["API_HOST_DOMAIN"]
	    db.port               = 5432
	    db.additional_options = ["-xc", "-E=utf8"]
		end

	 	store_with S3 do |s3|
	    s3.access_key_id     = ENV["BACKUP_BOT_AWS_KEY"]
	    s3.secret_access_key = ENV["BACKUP_BOT_AWS_SECRET"]
	    s3.region            = "ap-south-1"
	    s3.bucket            = <cm-live-projects bucket or storage provided by client>
	    s3.path              = <Project name>
	    s3.keep              = 5
	  end

		#notifier slack
	 	notify_by Slack do |slack|
	    slack.on_success = true
	    slack.on_warning = true
	    slack.on_failure = true
	    slack.webhook_url = ENV["BACKUP_BOT_SLACK_WEBHOOK"]
	  end
	end
	```

	This is how a demo model file with mimimum complexity configuration looks like. All the variables in this particular file are being fetched from the OS' environment variables (these were set through cloud66). There can be alternative ways to fetch these variables. The name, username and password of the database will be available on cloud66 (refer image below). A bucket was created to store these backups. A slack application was created to integrate a bot who sends the status of a backup. 

	The config file can be edited in a terminal based text editor or (be copied to the server).

	[![environment-variables-c66](/assets/images/environment-variables-c66.png)](/assets/images/environment-variables-c66.png)

3. __Automation__: 
<br/>
To generate a backup use command `backup perform -t <backup model name>`. A cron job can be used for automatic backups. The command will differ in that case `/usr/local/bin/backup perform -t staging` here. With Ruby, use the [whenever gem](https://github.com/javan/whenever) to schedule cron. Contents of crontab can be listed using `crontab -l`. 

<br/>
_Current version of this gem [does not have a restore command](https://github.com/backup/backup-features/issues/28). As of now restoration has to be manual. An automated restoration mechanism will be available soon._
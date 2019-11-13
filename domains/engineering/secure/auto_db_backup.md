---
title: Automatic db backups
nav_order: 2
parent: Secure
grand_parent: Engineering
---

1. Install the [backup gem](https://github.com/backup/backup) on the machine. Versions other than `5.0.0.beta.1` versions can return an error with some json version. ssh onto your server and run
	`gem install backup -v '5.0.0.beta.1'`

2. Generate a model for the database. This model file will have the type and configuration of storage, database, encryption, compression, notifier etc. for the database. 

`backup generate:model --trigger <model name> \
    --databases="postgresql" --storages="s3"  \
    --encryptor="gpg" --compressor="gzip" --notifiers="slack"`

Refer [this page](https://backup.github.io/backup/v4/generator/) for a detailed overview about the types of customizations available. You can omit any configuration that you don't need here on the command above.  

	
	Model.new(:caif_backup, 'Backup for staging of caif') do
		database PostgreSQL do |db|
	    db.name               = ENV["POSTGRESQL_DATABASE"]
	    db.username           = ENV["POSTGRESQL_USERNAME"]
	    db.password           = ENV["POSTGRESQL_PASSWORD"]
	    db.host               = ENV["api_host_domain"]
	    db.port               = 5432
	    db.additional_options = ["-xc", "-E=utf8"]
		end

	 	store_with S3 do |s3|
	    # AWS Credentials
	    s3.access_key_id     = ENV["backup_bot_aws_key"]
	    s3.secret_access_key = ENV["backup_bot_aws_secret"]
	    # Or, to use a IAM Profile:
	    # s3.use_iam_profile = true

	    s3.region            = "ap-south-1"
	    s3.bucket            = ENV["backup_bucket_name"]
	    s3.keep              = 5
	  end

		# Gzip [Compressor]
		compress_with Gzip

		#notifier slack
	 	notify_by Slack do |slack|
	    slack.on_success = true
	    slack.on_warning = true
	    slack.on_failure = true
	    slack.webhook_url = ENV["backup_bot_slack_webhook"]
	  end
	end

This is how a demo model file looks. All the variables in this particular file are being fetched from the OS' environment variables (these were set through cloud66). There can be alternative ways to fetch these variables. The name, username and password of the database will be available on cloud66 (refer image below). A bucket was created to store these backups. A slack application was created to integrate a bot who sends the status of a backup. 

3. A unix-cron job that runs at a particular time in the day will let us generate continuous backups. The crontab can be updated manually or using the `whenever` gem. [Check here](https://github.com/javan/whenever) to see how to set up whenever gem. 

Once the crontab is updated, automatic backups should be configured. Contents of the crontab can be listed using `crontab -l`. 

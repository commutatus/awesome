---
title: Rails secure credentials
nav_order: 5
parent: Secure
grand_parent: Engineering
---
Here is the documentation to use your rails crendentials securely

In the older version of rails 5.2 we place all our credentials inside  `config/secrets.yml` . But later rails 5.2 and upcoming versions replaced the current config/secrets.yml into config/secrets.yml.enc

**Before Using Rails Credentials**

1. So before using rails credentials the main thing to remember is to add a key inside `config/credentials/production.key` 

	We can use more files based on the environment

	Examples

		config/credentials/production.key
		config/credentials/staging.key
		config/credentials/development.key

2. But you should save your key which you added inside the file `config/credentials/production.key` somewhere else because this is the key used to encrypt the credentials, so if we miss this key then we can't decrypt our file to edit or use the credentials inside our `config/credentials/production.yml.enc`   
3. Then we need to keep the file `config/credentials/production.key` as secret so it should be added under `.gitignore` 

**Adding credentials**

1. As credentials are encrypted and saved in the file `config/credentials/production.yml.enc` so to add your credentials inside this we should run this command in the terminal
	`$ EDITOR="nano" rails credentials:edit`

	To add credentials using other editor you can simply change the name

	`$ EDITOR="subl --wait" rails credentials:edit`

2. If your using different credentials based on environment the you should pass environment name when we running the command

		$ EDITOR="subl --wait" rails credentials:edit --environment production
		$ EDITOR="subl --wait" rails credentials:edit --environment staging
		$ EDITOR="subl --wait" rails credentials:edit --environment development

3. Add your credentials inside the file

	Examples

		aws:
			access_key_id: 123
			secret_access_key: 345
			
		secret_key_base: 84fd6106329678ce7c098f66b7770463f7de50658339f221f8a662d64557295e7b6977c32cba10a00a573868799d9adb04f6e783acc31ef56704161572d9ee3b

	When you save the file and exit your editor, you should be prompted in the terminal with the following message:

	`File encrypted and saved.`

**Using Credentials**
1. The key value pair we added to the credentials file is available for us to use in our application through the `Rails.application.credentials` object.

2. To check if our addition has been encrypted properly, let's open up the rails console and try to access (decrypt) the value.
		
		$ rails c
		Loading development environment (Rails 5.2.0.rc1)
		[1] pry(main)> Rails.application.credentials.aws[:access_key_id]
		=> 123

3. We can call rails credentials inside  project by various ways
	eg. 

		Rails.application.credentials.aws[:access_key_id]
		Rails.application.credentials.dig(:aws, :access_key_id) 
		Rails.application.credentials.dig(:secret_key_base)

**Note**
1. The file will decrypt and open automatically at the time when we run the command `EDITOR="subl --wait" rails credentials:edit --environment production` to edit if the key presented inside the file `config/credentials/production.yml` was the correct one which is used before generation of `config/credentials/production.yml.enc`.
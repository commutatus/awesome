---
title: Backend Deployment
nav_order: 2
parent: Efficient
grand_parent: Engineering
---

1. TOC
{:toc}

# Rails

## Cloud 66

### Previous preparation
- If you require to run a command on every new deployment, add a `.cloud66/deploy_hooks.yml` configuration file. Example structure for installing a linux package can be:
```yml
staging: # Environment
    first_thing: # Hook point
      - command: apt-get install ffmpeg -y # Hook type
        target: any # Hook fields
        execute: true
production: # Environment
    first_thing: # Hook point
      - command: apt-get install ffmpeg -y # Hook type
        target: any # Hook fields
        execute: true
```
- If you require workers, add a `Procfile` at the root directory. Example content:
```Procfile
worker: bundle exec sidekiq -e $RAILS_ENV
```
### New Rails server deployment
1. On the [Cloud 66 Dashboard](https://app.cloud66.com/dashboard), click on *New Application*.
  1. If you haven't, link your Github account to your Cloud66 acount
2. Select the git repository you wish to deploy
3. Select the branch, the environment, and the application name. Conventions to follow are:
  - `staging` branch will go to the `Staging` environment, `master` branch will go to the `Production` environment
  - The application name will be in `Title Case` and end with `API` for backend applications. The environment name is not needed.
4. Click on `Analize Application` and wait for a couple minutes
5. Configure the deployment servers that will be used. Take into account:
  - At *Cloud Provider*, select the customer's cloud account. If it doesn't appear there, add it or contact your team leader
  - *Server Region* is customer dependant, get it from your team leader or the product manager.
  - *Server Size* should be `t2.micro` for staging and `t2.medium` for production
  - *Deploy PostgreSQL* should be `Shared with the Rails server` for staging and a `New cloud server` for production
6. Click on *Deploy Application*. That's it!


### Useful commands
To run these commands, first install the Cloud66 toolbelt in your machine by running `curl -sSL https://s3.amazonaws.com/downloads.cloud66.com/cx_installation/cx_install.sh | bash`
1. `cx stack list`: List all the stacks. Useful to get the `$STACK_NAME` value for the other commands, and see the last deployment status and date/time
2. `cx ssh -s $STACK_NAME web`: Open a ssh session at the desired stack's main web server
3. `cx redeploy -s $STACK_NAME`: Redeploy the desired stack
4. `cx tail -s $STACK_NAME web $LOGFILE` View real-time logs of the desired logfile. Values for `$LOGFILE` can be:
  - `staging.log / production.log`: The main Rails application log server. Choose the right name according to the server's environment
  - `nginx_error.log`: The nginx HTTP server log

### Further actions
- [Add a new ssl certificate]({% link domains/engineering/secure/ssl.md %}#rails)

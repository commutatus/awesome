---
title: Frontend Deployment
nav_order: 1
parent: Efficient
grand_parent: Engineering
---

- ## ***Elastic Beanstalk - Angular***

1. Create an AWS account if you don’t have one. Log-in and go to [AWS Console](https://console.aws.amazon.com/). Click on **Services** menu > Select **Elastic Beanstalk** under **Compute** heading.

   [![All services](/assets/images/deploy-elb-services.png)](/assets/images/deploy-elb-services.png)

2. In case you land up to the similar screen shown in the image below. Here you can change the hosting region of your application from the top right corner *(says Mumbai in the screenshot)*. Your region will also be visible in the browser’s address bar *(see screenshot).* After you select your region click on **Create New Application** on the top right corner.

   [![ELB landing page](/assets/images/deploy-elb-intro-page.png)](/assets/images/deploy-elb-intro-page.png)

3. Give your application a meaningful name. It would be great if it matches your actual application name. Click the **Create** button.

   [![Create application](/assets/images/deploy-elb-create-app.png)](/assets/images/deploy-elb-create-app.png)

   You will land up in the **All Applications** page, where you can create different environments for your application. However, for the demo purposes, I will create only one environment.

   [![All apps](/assets/images/deploy-elb-all-app.png)](/assets/images/deploy-elb-all-app.png)

4. Click on the **Create one now** and select the **Web server environment** option.

5. Add the environment name you like. I prefer `<APPLICATION_NAME-ENV>`, so I have added **demo-production.** In the **Preconfigured platform** option select **Nodejs** from the dropdown and click **Configure more options.**

   [![Create environment](/assets/images/deploy-elb-create-env.png)](/assets/images/deploy-elb-create-env.png)

6. This takes you to this screen below. Here you will see various aspects to configuring your application. We will choose **Capacity** first *(click Modify in Capacity block)*.

   [![Configuration grid](/assets/images/deploy-elb-config-grid.png)](/assets/images/deploy-elb-config-grid.png)

   *In the capacity section, you will add an auto-scaling capability to your application by making it load balanced using multiple instances based on your requirements.*

7. Change the default values to values shown in the screenshot and click **Save**.

   [![Capacity screen](/assets/images/deploy-elb-capacity.png)](/assets/images/deploy-elb-capacity.png)

   *Here I have changed the* **Environment type** *to* **Load balanced,** *and I’m keeping* **Instances** *maximum value* and *minimum value* **1** *because this is a demo, but you might want to change it to 2–3 or more depending on your needs.*

8. Click on **Load Balancer’s** Modify button and change the type to [**Application Load Balancer**](https://searchaws.techtarget.com/definition/application-load-balancer)**.** It will help in the routing procedure for HTTP and HTTPS (protocol-based). Click on **Save.**

   [![Load balancer type - Application](/assets/images/deploy-elb-capacity-merge.png)](/assets/images/deploy-elb-capacity-merge.png)

9. Click on **Create environment.**

   [![Environment created](/assets/images/deploy-elb-env-created.png)](/assets/images/deploy-elb-env-created.png)

   This will take some time to create, and once done, it should look exactly like the image below. It will also contain the URL of your application, though as of now it will open the default Elastic Beanstalk dummy page. So, we are done with EB.

10. In your `.travis.yml` file append the following code snippet for deploying your application. You have to generate your`access_key_id` and `secret_access_key` from [AWS credentials](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-credentials.html) for JS. The `bucket_name` will take the name of the AWS S3 bucket which will be created automatically by AWS while creating the new application in EB.

    ```yaml
    language: node_js

    node_js:
      - 10.15.1

    cache:
      directories:
        - node_modules

    branches:
      only:
        - staging
        - master

    before_script:
      - npm install

    script:
      - if [[ "$TRAVIS_BRANCH" = "master" || "$TRAVIS_BRANCH" = "staging" ]]; then npm run build; else echo "not a build branch"; fi

    before_deploy:
      - cd $TRAVIS_BUILD_DIR
      - sed -i '/dist/d' .gitignore
      - git add . && git commit -m "latest build"

    deploy:
      - provider: elasticbeanstalk
        access_key_id: <YOUR_ACCESS_KEY>
        secret_access_key:
          secure: <YOUR_SECRET_ACCESS_KEY>
        region: "<HOSTING_REGION>"            // "ap-south-1"
        app: "<APP_NAME>"                     // "demo-application"
        env: "<ENV_NAME>"                     // "demo-production"
        bucket_name: "<S3_BUCKET_NAME>" // Once you create new application, it will automatically create a unique bucket for it.
        on:
          branch: <GIT_BRANCH_NAME> // For example: - master

    after_deploy:
      - echo "Applcation Deployed!"
    ```



11. Go to **Services** > **Click on S3 under Storage heading** > **Copy the bucket name** *(as highlighted in the image)* and paste in `.yml` file.

    [![Final s3 bucket](/assets/images/deploy-elb-bucket-final.png)](/assets/images/deploy-elb-bucket-final.png)

12. Make commit and push your changes of `.travis.yml` file to the master branch. This will trigger a build, and once it is successful, the application will automatically be deployed to the URL, I have mentioned above.





- ## ***Heroku - Angular***

#### Configure Heroku

1. Sign up/Log in to [Heroku](https://www.heroku.com/) and on the dashboard screen click **New** button and select **Create new app** top right.

   [![Heroku dashboard](/assets/images/deploy-heroku-dashboard.png)](/assets/images/deploy-heroku-dashboard.png)

2. Give a name to your application and select the one of the **available region** from the dropdown and click **Create** **app**.

   [![Create app](/assets/images/deploy-heroku-create-app.png)](/assets/images/deploy-heroku-create-app.png)

3. Since your project repo is on [GitHub](https://www.github.com/), click **GitHub** and under the organisation name select **Commutatus** and search for your project repository and click **Connect**.

   [![Heroku - GitHub connect](/assets/images/deploy-heroku-connect-github.png)](/assets/images/deploy-heroku-connect-github.png)



4. Select the branch you want to deploy and then click on **Enable Automatic Deploys** for future auto deployments. But for the first, select the branch and click **Deploy Branch**.

   [![Auto deployment settings](/assets/images/deploy-heroku-deploy-master.png)](/assets/images/deploy-heroku-deploy-master.png)

5. After deployment is successful, Heroku will generate a link to your application. For example *https://awesome--app.herokuapp.com*. This link will show **Application Error** for now.

#### Configure your Angular application

1. In your Angular application, open `package.json` file and copy `@angular/cli: <version>`,  `"@angular/compiler-cli": <version>` and `"typescript": <version>` from `devDependencies` to `dependencies` .

2. Create a postinstall script in your `package.json` under `script` section and paste `"postinstall": "ng build --aot -prod"`.

3. Now for Heroku to run your application we need to tell Heroku for node environment engines. Copy and paste the below code under `"devDependencies"` object.

   ```json
     "engines": {
       "node": "<same version of your machine>",
       "npm": "<same version of your machine>"
     }
   ```

4. Install server to run your application. For production be need an express server. Run `npm i express path --save` in your code.

5. Create a **server.js** file in the **root** of your application and paste the code below.

   ```javascript
   const express = require('express');
   const path = require('path');
   const app = express();
   
   // Serve only the static files from the dist folder
   app.use(express.static(__dirname + '/dist/<name-of-app>'));
   app.get('/*', function(req,res) {
     res.sendFile(path.join(__dirname+'/dist/<name-of-app>/index.html'));
   });
   
   // Start the app by listening on the default Heroku port
   app.listen(process.env.PORT || 8080);
   ```

6. In your **package.json** change the `"start"` command to `"node server.js"`.

7. Add your changes to GIT and push to your remote GIT origin. After the build is successful open the link.
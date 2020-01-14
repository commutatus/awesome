---
title: Frontend Deployment
nav_order: 1
parent: Efficient
grand_parent: Engineering
---

1. TOC
{:toc}

# Angular/React

## Elastic Beanstalk - Angular
- ***Elastic Beanstalk - Angular***

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


### **Travis CI Integration**

1. In your `.travis.yml` file append the following code snippet for deploying your application. You have to generate your`access_key_id` and `secret_access_key` from [AWS credentials](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-credentials.html) for JS. The `bucket_name` will take the name of the AWS S3 bucket which will be created automatically by AWS while creating the new application in EB.

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
        access_key_id: ${AWS_ACCESS_KEY_ID}             // You have use environment variable name instead of plain text
        secret_access_key: ${AWS_SECRET_ACCESS_KEY}     // You have use environment variable name instead of plain text
        region: "<HOSTING_REGION>"            // "ap-south-1"
        app: "<APP_NAME>"                     // "demo-application"
        env: "<ENV_NAME>"                     // "demo-production"
        bucket_name: "<S3_BUCKET_NAME>" // Once you create new application, it will automatically create a unique bucket for it.
        on:
          branch: "<GIT_BRANCH_NAME>" // For example- master

    after_deploy:
      - echo "Applcation Deployed!"
    ```



2. Go to **Services** > **Click on S3 under Storage heading** > **Copy the bucket name** *(as highlighted in the image)* and paste in `.yml` file.

    [![Final s3 bucket](/assets/images/deploy-elb-bucket-final.png)](/assets/images/deploy-elb-bucket-final.png)

3. Make commit and push your changes of `.travis.yml` file to the master branch. This will trigger a build, and once it is successful, the application will automatically be deployed to the URL, I have mentioned above.

### **GitHub Actions Integration**
1. Go to your GitHub repository and click the **Actions** tab button.
2. Click on the **Set up a workflow yourself** button, on the right corner, as shown in the screenshot below.

    [![Actions start](/assets/images/actions-diy-workflow.png)](/assets/images/actions-diy-workflow.png)

    This will open a code editor kind of window on the left and GitHub marketplace on the right.
    This will attempt to create a `.github/workflows/main.yml` file in your root *(It will only create once you commit the changes)*.

    [![Actions workflow file](/assets/images/actions-workflow-file.png)](/assets/images/actions-workflow-file.png)

3. In the editor paste the code snippet below and click on the **Start commit** button on the right corner. Add some commit message and click on **Commit new file**.

    <script src="https://gist.github.com/yashwp/65d5f1e31a43e2172f1a457f12dd0d7a.js"></script>

    Doing this will commit and push the new changes to your master branch by default. However, you can change the branch or you can create the folders on your local machine and push it to your desired branch.

    In the above code snippet, under `steps:` we are using [an action library used to deploy to Beanstalk](https://github.com/marketplace/actions/beanstalk-deploy) which is available on the GitHub marketplace that facilitates the deployment procedure.
4. Open your project in the code editor on your machine and pull the updated code from the `master` branch.
5. Add two new tasks for caching `node_modules` and setup the node environment for your application. Append the code below under `steps:`.

    <script src="https://gist.github.com/yashwp/5665b0475907b4792758da0fc8edfe82.js"></script>

6. Now add other scripts to install your npm dependencies and build your application in production mode.

    <script src="https://gist.github.com/yashwp/791ec94c4f5487acf061e6eae96b5685.js"></script>

7. This is the important step, where you need to generate a zip file that will contain your complete source code *(without `node_modules`)*. The command below will recursively zip all the files inside the current directory except the `node_module` folder.

    <script src="https://gist.github.com/yashwp/18f2cb2240f7a81f8a936f78b6768610.js"></script>

8. Paste the code below under the instructions given above. This code snippet will generate a time-stamp hash which will be used with `version` key of our deployment zip file.

    <script src="https://gist.github.com/yashwp/750e56203a7c0bcae8aded42d5f3eef5.js"></script>

9. Now go to the **Settings** tab of your repository and on the side menu, select **Secrets**. Secrets are basically your private values like AWS secret key and access key which should not be present directly in your code.
10. Click on **Add a new secret** button and add your AWS secret keys and access keys with recommended name as `AWS_ACCESS_KEY` and `AWS_SECRET_KEY` . These keys will be used in the next step.

    [![Actions secret](/assets/images/actions-secrets.png)](/assets/images/actions-secrets.png)

11. Now the final step is to add value to snippet which we have added in step #3. So now your `.github/workflows/main.yml` will look like this:

    <script src="https://gist.github.com/yashwp/66e3c1ca95fa4a7d64a1cfe332e18117.js"></script>

## AWS S3
### Creating S3 Bucket
- Go to [AWS S3 Console](https://s3.console.aws.amazon.com/s3/home?region=eu-west-2#)
- Click on Create Bucket and enter a bucket name which should be the EXACT same name as the domain you want to deploy to. Let us assume you want to deploy the app on `awesome.commutatus.com` so we write the bucket name as `awesome.commutatus.com`. Now select the region which is closest to most of your target audience. Click Next.
[![Create Bucket First Step](/assets/images/s3-deployments/s3-deployment-1.png)](/assets/images/s3-deployments/s3-deployment-1.png)
- Click Next on the second step, on the third step make sure the `Block all public access` is unchecked and click next. The last screen should look like this and then you can click on `Create Bucket`
[![Create Bucket Last Step](/assets/images/s3-deployments/s3-deployment-2.png)](/assets/images/s3-deployments/s3-deployment-2.png)
- Open your newly created bucket and click on `Properties` => `Static Website Hosting` 
- Select the option `Use this bucket to host a website`  and in the index document field, write `index.html` (if that's the entry point of your app). Click on Save.
[![Create Bucket Last Step](/assets/images/s3-deployments/s3-deployment-3.png)](/assets/images/s3-deployments/s3-deployment-3.png)
The endpoint you see `http://awesome.commutatus.com.s3-website.ap-south-1.amazonaws.com` is the s3 url where your app will be accessible.
- Now while you are in your s3 view, go to `Permissions` => `Bucket Policy` and enter this code snippet
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AddPerm",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::awesome.commutatus.com/*"
        }
    ]
}
```
Make sure you replace the name with your bucket name in the `Resource` field. After doing this, click on `Save`
Your bucket is all set to serve your app! Now we need to push in our code to this bucket.
### Deploying our app to the bucket
-  We'll be using travis in order to push changes to our bucket. If you don't have a travis file already you can start off with this one
```
language:  node_js

node_js:

-  10.15.1

cache:

directories:

-  node_modules

branches:

only:

-  master

before_script:

-  npm install

script:

-  if [ "$TRAVIS_BRANCH" = "master" ]; then npm run build:production; else echo "not

a master branch"; fi

# You can modify the build commands as per your setup

before_deploy:

# This step is done to remove the dist directory from gitignore otherwise it is ignored by travis for deployment

-  cd $TRAVIS_BUILD_DIR

-  sed -i '/dist/d' .gitignore

-  git add . && git commit -m "latest build"

-  cd $TRAVIS_BUILD_DIR/dist/

-  pip install --user awscli

deploy:

-  provider:  script

script:  ~/.local/bin/aws s3 sync ./ s3://awesome.commutatus.com --region=ap-south-1 --delete

access_key_id:  $AWS_ACCESS_KEY_ID

secret_access_key:  $AWS_SECRET_ACCESS_KEY

bucket:  awesome.commutatus.com

region:  ap-south-1

on:

branch:  master

after_deploy:

# You can write your after deploy scripts here like invalidating cdn caches
````
You can take this as a boiler plate for travis but what  we are mostly interested is in the `deploy` script.  The script will deploy the dist folder and also delete what was earlier being deployed in order to avoid any unnecessary cluttering of the bucket. Your `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` should be defined in your travis settings. If everything goes fine, you can head back to the s3 url you receive and should see your app running!
- We have deployed the app but there's a problem here, s3 does not support https, hence in order to make our app support https, we have to serve it via Cloudfront. This will also ensure that the data is loaded efficiently.
### Creating an SSL Certificate
- Before creating a Cloudfront Distribution, we'll need an SSL certificate that we can hook up to our cloudfront to make it https ready. 
- To create the SSL certificate head over to [AWS Certificate Manager](https://console.aws.amazon.com/acm/home?region=us-east-1#). Make sure you are in the region `N. Virginia`(as all the Cloudfront SSL certs have to be defined there). 
- If you are new, Click on `Get Started` under `Provision certificates` or if there are already certs created, then you can click on `Request a certificate`. 
- After click it select `Request a public certificate` and click next. Now in the doman name field, enter your domain like, in this example, `awesome.commutatus.com` and click next. 
- Select `DNS` Validation and click next.
- On the next screen check if you have inputted proper data and click on `Verify and Submit`
- After this step you should find a `CNAME` name and value record, you have to add this record in your DNS manager. After you're done adding. after sometime (from 2mins to 1 hour approximately), you should see the status of your certificate as `issued`.
[![Create Bucket Last Step](/assets/images/s3-deployments/s3-deployment-4.png)](/assets/images/s3-deployments/s3-deployment-4.png)
- Once this is done we can start making our Cloudfront Distrubition.
### Creating Cloudfront Distribution
- To setup a cdn. head over to [Cloudfront](https://console.aws.amazon.com/cloudfront/home?region=eu-west-2#). 
- Click on `Create Distribution` in the Webs section click on `Get Started`.
- In the `Origin Domain Name` select your bucket. 
- In `Viewer Protocol Policy` select `Redirect HTTP to HTTPS`
- In `Alternate Domain Names  
(CNAMEs)` enter your domain name, in this case, `awesome.commutatus.com`
- In SSL select `Custom SSL` and select the SSL cert that you generated for `awesome.commutatus.com`
- In `Default Root Object` enter `index.html` (if that's the entry point of your app)
- Click on `Create Distribution` give this process some time. The status will show as `In Progress` once the status becomes `Deployed` your distribution should be ready.
- To check if your distribution works, open your distribution and there will be a field called `Domain name`, in this case it is `  
d1sjyp0cxacal8.cloudfront.net`, open that link in your browser and check if that works.

- Now the last step is to route our domain `awesome.commutatus.com` to our Cloudfront domain which was `  
d1sjyp0cxacal8.cloudfront.net`. This will take another 2-60 mins approximately. And if everything goes well, your app should be available at `awesome.commutatus.com`

### Automatically invalidating cdn caches

- There's one last step, since our app is served via a cdn, any updates we push would not get propagated immediately unless we invalidate the cache of the cdn. In order to overcome this we need to invalidate the cdn cache after every deployment. You can add this script in the `after_deploy` section of your travis file, this will make sure our caches are invalidated after the deployments.
```````yaml
- if [ "$TRAVIS_BRANCH" = "master" ]; then aws cloudfront create-invalidation --distribution-id EJBDNDTRY39BS --paths "/*"; else echo "not a master branch for invalidation"; fi
```````
- You can find the `distribution-id` in your Cloudfront distribution page.

## Heroku - Angular

### Configure Heroku

1. Sign up/Log in to [Heroku](https://www.heroku.com/) and on the dashboard screen click **New** button and select **Create new app** top right.

   [![Heroku dashboard](/assets/images/deploy-heroku-dashboard.png)](/assets/images/deploy-heroku-dashboard.png)

2. Give a name to your application and select the one of the **available region** from the dropdown and click **Create** **app**.

   [![Create app](/assets/images/deploy-heroku-create-app.png)](/assets/images/deploy-heroku-create-app.png)

3. Since your project repo is on [GitHub](https://www.github.com/), click **GitHub** and under the organisation name select **Commutatus** and search for your project repository and click **Connect**.

   [![Heroku - GitHub connect](/assets/images/deploy-heroku-connect-github.png)](/assets/images/deploy-heroku-connect-github.png)
   
4. Select the branch you want to deploy and then click on **Enable Automatic Deploys** for future auto deployments. But for the first, select the branch and click **Deploy Branch**.

   [![Auto deployment settings](/assets/images/deploy-heroku-deploy-master.png)](/assets/images/deploy-heroku-deploy-master.png)

5. After deployment is successful, Heroku will generate a link to your application. For example *https://awesome--app.herokuapp.com*. This link will show **Application Error** for now.

6. In case if you want to configure *Environment variables* for your application. Goto **Settings** tab > **Config Vars** add environment variable as key-value pair.

    [![Environment variable](/assets/images/deploy-heroku-env.png)](/assets/images/deploy-heroku-env.png)

### Configure your Angular application


1. In your Angular application, open `package.json` file, create a postinstall script in your `package.json` under `script` section and paste `"postinstall": "ng build --aot -prod"`.

2. Now for Heroku to run your application we need to tell Heroku for node environment engines. Copy and paste the below code below `"devDependencies"` object.

   ```json
     "engines": {
       "node": "<same version of your machine>",
       "npm": "<same version of your machine>"
     }
   ```

3. Install server to run your application. For production be need an express server. Run `npm i express path --save` in your code.

4. Create a **server.js** file in the **root** of your application and paste the code below.

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

5. In your **package.json** change the `"start"` command to `"node server.js"`.

6. Add your changes to GIT and push to your remote GIT origin. After the build is successful open the link.

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
````
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

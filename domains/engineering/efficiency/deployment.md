---
title: Deployment
nav_order: 1
parent: Efficiency
grand_parent: Engineering
---

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
---
title: Serverless Deployment
nav_order: 1
parent: Efficient
grand_parent: Engineering
---

- ***Serverless - AWS***

1. Prerequisites - aws credentials must exist in `~/.aws/credentials` or a path which needs to be referred when the application is deployed. Here is how you can [manage aws credentials](https://awesome.commutatus.com/domains/engineering/secure/aws-credentials.html){:target="_blank"}

2. An AWS CloudFormation template is created from the `serverless.yml` file. In case a stack has not been created then it would be created when a deployement is requested, using only s3 resources where all the code will be zipped and stored.

    ```
      service: aws-ruby # NOTE: update this with your service name

      provider:
        name: aws
        runtime: ruby2.5
        timeout: 300
       
      functions:
        hello:
          handler: handler.so_long
          layers:
            - { Ref: GemlayerLambdaLayer }
          environment:
            GEM_PATH: /opt/2.5.0

      layers:
        gemlayer:
          path: ruby
    ```

    *Functions are been defined under the `functions` key. Example - the function being called `hello` as the key, should be defined in a file `handler.rb` with the name `so_long`. AWS allows using layers to share code/libraries across multiple functions. This particular example keeps gem code in layers which can be used across functions.*

3. Use command `sls deploy` to deploy the application to aws. To use a different credential profile than default, use `sls deploy --aws-profile private`. Here, the credentials from the profile `private` will be used. 
  
    [![aws lambda console](/assets/images/aws-lambda-console.png)](/assets/images/aws-lambda-console.png)

    *The name of the service is appended before the function name as in the picture and the yaml file. This service name is editable. The stage of the function is also part of the name in the list.*

4. Serverless let's you deploy functions on different `stages`. This is analogous to environments (test/staging/production). You can have a different set of environment variables (or stage variables) for each such stage. The deploy and invoke command both accept the `stage` parameter for example - `sls deploy --stage dummy --aws-profile private`  and `sls invoke --function hello --stage dummy --aws-profile private` . The stages have been highlighted on the image above. Environment variables can be managed by configuring the yaml file; [refer the docs here](https://serverless.com/framework/docs/providers/aws/guide/variables/){:target="_blank"}.
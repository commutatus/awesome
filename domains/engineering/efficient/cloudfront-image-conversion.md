---
title: Cloudfront convert image to .webp
nav_order: 6
parent: Efficient
grand_parent: Engineering
---

## Convert Image from .jpg/.png to .webp

This is the continuous of the previous implementation where we serve rails assets through cloudfront. Now When a cloudfront URL is hit we are going to convert .jpg/.png to a .webp image. Other formats will be served as it is.

The WebP format has become increasingly popular since Google introduced it in 2010. Its biggest selling point lies in its ability to produce much smaller file sizes while maintaining similar image quality. Faster load times = higher conversion rates.

WebP is a modern image format that provides superior lossless and lossy compression for images on the web. WebP lossless images are 26% smaller in size compared to PNGs. WebP lossy images are 25–34% smaller than comparable JPEG images at equivalent SSIM quality index. — Google

### References

I would suggest going through the following articles and have a understanding about how the jpg -> webp conversion flow works and then continue here. This article contains the bug fixed code and some improvements.

[Resizing images with amazon cloudfront lambdaedge aws cdn blog](https://aws.amazon.com/blogs/networking-and-content-delivery/resizing-images-with-amazon-cloudfront-lambdaedge-aws-cdn-blog/)

[Converting images to webp from cdn](https://medium.com/nona-web/converting-images-to-webp-from-cdn-9433b56a3d52)

I followed the step by step section mentioned in the AWS blog.


### Step 1: Creating the Deployment Packages
First create a project folder, I named it docker-sharp . I created a Docker file inside it. I also created the same folder structure as explained.
Find the code repo [here](https://github.com/commutatus/cloudfront-lambda-image-optimization)
```
docker-sharp  # This is the root folder
docker-sharp → lambda → viewer-request-function → handler.js
docker-sharp → lambda → origin-response-function → handler.js
docker-sharp → Dockerfile
```
Dockerfile
```
FROM amazonlinux:2

WORKDIR /tmp
#install the dependencies
RUN yum -y install tar && yum -y install gzip && yum -y install gcc-c++ && yum -y install findutils

RUN touch ~/.bashrc && chmod +x ~/.bashrc

RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.5/install.sh | bash

RUN source ~/.bashrc && nvm install 10.0

WORKDIR /build
 ```

docker-sharp → lambda → viewer-request-function → handler.js
```
const userAgent = require('useragent')
const path = require('path')

exports.handler = async (event, context, callback) => {
  const request = event.Records[0].cf.request
  const headers = request.headers
  const userAgentString = headers['user-agent'] && headers['user-agent'][0] ? headers['user-agent'][0].value : null
  const agent = userAgent.lookup(userAgentString)

  const browsersToInclude = [
    { browser: 'Chrome', version: 23 },
    { browser: 'Opera', version: 15 },
    { browser: 'Android', version: 53 },
    { browser: 'Chrome Mobile', version: 55 },
    { browser: 'Opera Mobile', version: 37 },
    { browser: 'UC Browser', version: 11 },
    { browser: 'Samsung Internet', version: 4 }
  ]

  const supportingBrowser = browsersToInclude
    .find(browser => browser.browser === agent.family && agent.major >= browser.version)

  if (supportingBrowser) {
    const fileFormat = path.extname(request.uri).replace('.', '')
    request.headers['original-resource-type'] = [{
      key: 'Original-Resource-Type',
      value: `image/${fileFormat}`
    }]

    const olduri = request.uri
    const newuri = olduri.replace(/(\.jpg|\.png|\.jpeg)$/g, '.webp')
    request.uri = newuri
  }

  return callback(null, request)
}
```

docker-sharp → lambda → origin-response-function → handler.js
```
const path = require('path')
const AWS = require('aws-sdk')

const S3 = new AWS.S3({
  signatureVersion: 'v4',
})

const Sharp = require('sharp')
const BUCKET = 'BUCKET-NAME'
const QUALITY = 75

exports.handler = async (event, context, callback) => {
  const { request, response } = event.Records[0].cf
  const { uri } = request
  const headers = response.headers

  if (path.extname(uri) === '.webp') {
    console.log(response)
    console.log(request)
    if (parseInt(response.status) === 404) {
      const format = request.headers['original-resource-type'] && request.headers['original-resource-type'][0]
        ? request.headers['original-resource-type'][0].value.replace('image/', '')
        : null

      const key = uri.substring(1)
      const s3key = key.replace('.webp', `.${format}`)
      console.log("Printing key:", key)
      console.log("Printing S3key", s3key)
      try {
        const bucketResource = await S3.getObject({ Bucket: BUCKET, Key: decodeURI(s3key.split('+').join(' ')) }).promise()
        const sharpImageBuffer = await Sharp(bucketResource.Body)
          .webp({ quality: +QUALITY })
          .toBuffer()
        console.log("Got the S3 image and converted. Trying to put it into S3")
        await S3.putObject({
          Body: sharpImageBuffer,
          Bucket: BUCKET,
          ContentType: 'image/webp',
          CacheControl: 'max-age=31536000',
          Key: decodeURI(key.split('+').join(' ')),
          StorageClass: 'STANDARD'
        }).promise()

        response.status = 200
        response.body = sharpImageBuffer.toString('base64')
        response.bodyEncoding = 'base64'
        response.headers['content-type'] = [{ key: 'Content-Type', value: 'image/webp' }]
      } catch (error) {
        console.log("Printing the error:")
        console.error(error)
      }
    } else {
      console.log("Its not 404, So returning webp")
      headers['content-type'] = [{
        'value': 'image/webp',
        'key': 'Content-Type'
      }]
    }
  }
  callback(null, response)
 }
```
### Step 2: Building docker
Change directory to `docker-sharp` thats the root of the project.

From project root folder, run the following commands:

Note: Remember to change the Bucket Name and other values properly, because cloudfront deployment takes a long time.

The Dockerfile is configured to download Amazon Linux and install Node.js 10.0 along with dependencies.
```
docker build --tag lambci/lambda:build-nodejs10.0 .
```
Install the sharp and querystring module dependencies and compile the ‘Origin-Response’ function.
```
docker run --rm --volume ${PWD}/lambda/origin-response-function:/build lambci/lambda:build-nodejs10.0 /bin/bash -c "source ~/.bashrc; npm init -f -y; npm install sharp --save; npm install querystring --save; npm install --only=prod"
```
Install the querystring module dependencies and compile the ‘Viewer-Request’ function.
```
docker run --rm --volume ${PWD}/lambda/viewer-request-function:/build lambci/lambda:build-nodejs10.0 /bin/bash -c "source ~/.bashrc; npm init -f -y; npm install querystring --save; npm install path --save; npm install useragent --save; npm install yamlparser; npm install --only=prod"
```
Package the ‘Origin-Response’ function.
```
mkdir -p dist && cd lambda/origin-response-function && zip -FS -q -r ../../dist/origin-response-function.zip * && cd ../..
```
Package the ‘Viewer-Request’ function.
```
mkdir -p dist && cd lambda/viewer-request-function && zip -FS -q -r ../../dist/viewer-request-function.zip * && cd ../..
```
From AWS console, create S3 bucket in us-east-1 region to hold the deployment files and upload the zip files created in above steps. These would be referenced from the CloudFormation template during deployment. Since we thought this might be required in multiple projects and it has to be in `us-east-1` we created a bucket like
```
BUCKET-NAME → project-name-environment → origin-response-function.zip
BUCKET-NAME → project-name-environment → viewer-request-function.zip
```

### Step 3: Deploy the Lambda@Edge Functions
There were minor problems fixed from the AWS blog. IAM Role creation was having error when cloud formation was deploying

Note: Remember to change the Bucket Name and other values properly, because cloudfront deployment takes a long time.

CloudFormation Template:
```
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Description: Rewrite jpg and png requests to webp if the browser supports webp
Resources:
  ViewerRequestFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: s3://docker-sharp/viewer-request-function-1.zip
      Handler: handler.handler
      Runtime: nodejs10.x
      MemorySize: 128
      Timeout: 1
      Role: !GetAtt WebEdgeLambdaRole.Arn

  ViewerRequestFunctionVersion:
    Type: "AWS::Lambda::Version"
    Properties:
      FunctionName: !Ref ViewerRequestFunction
      Description: "A version of ViewerRequestFunction"

  OriginResponseFunction:
   Type: AWS::Serverless::Function
   Properties:
     CodeUri: s3://docker-sharp/origin-response-function-1.zip
     Handler: handler.handler
     Runtime: nodejs10.x
     MemorySize: 512
     Timeout: 5
     Role: !GetAtt WebEdgeLambdaRole.Arn

  OriginResponseFunctionVersion:
    Type: "AWS::Lambda::Version"
    Properties:
      FunctionName: !Ref OriginResponseFunction
      Description: "A version of OriginResponseFunction"

  # ==== ROLES ==== #
  WebEdgeLambdaRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          -
            Effect: Allow
            Action: sts:AssumeRole
            Principal:
              Service:
                - "lambda.amazonaws.com"
                - "edgelambda.amazonaws.com"
  # ==== POLICIES ==== #
  PublishLogsPolicy:
    Type: AWS::IAM::ManagedPolicy
    Properties:
      Description: Allows functions to write logs
      Roles:
      - !Ref WebEdgeLambdaRole
      PolicyDocument:
        Version: 2012-10-17
        Statement:
          - Effect: Allow
            Action:
            - logs:CreateLogGroup
            - logs:CreateLogStream
            - logs:PutLogEvents
            Resource: '*'
```

Steps to deploy cloudformation template:

To deploy using cloud formation template.
```
aws cloudformation deploy --template-file aws-sam-lambda-edge-webp.yaml --stack-name webp-conversion --capabilities CAPABILITY_NAMED_IAM --region us-east-1
```
After deploying the deployment finishes the cloudfront would have created the lambda functions for both viewer-request and the origin-response. Go to the viewer-request lambda function, check if version is released, copy the ARN of the release with version. For eg it will have the version at the end appended else release version and copy the ARN.

Now go to `cloudfront distribution → go to behaviours section → Go to lambda function section`

Choose the viewer request from dropdown → Paste the ARN copied earlier → Include body

Go to the origin-response lambda function, check if version is released, copy the ARN of the release with version. For eg it will have the version at the end appended else release version and copy the ARN.

Now go to cloudfront distribution → go to behaviours section → Go to lambda function section

Choose the origin response from dropdown → Paste the ARN copied earlier

Setting S3 bucket policy:

There was a problem saving the files into the S3 bucket. The bucket policy of the S3 bucket that we are going to change needs to be accessible by the lambda functions. Here is the snippet, you need to change it to the respective bucket names and the lambda role names
```
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::test-cloudfront-doc/*"
        },
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::410230403282:role/webp-conversion-WebEdgeLambdaRole-Q6ACKG21A14M"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::test-cloudfront-doc/*"
        },
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::410230403282:role/webp-conversion-WebEdgeLambdaRole-Q6ACKG21A14M"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::test-cloudfront-doc/*"
        }
    ]
}
```
Now the deployment takes around 20 mins. After which if you hit the cloudfront URL it should convert the image and upload the .webp image to same place.

### Things to note

1. After deployment if S3 images are not accessbile even after policy, Enable the read permission for everyone.
2. S3 URL with + and space have been handled here.

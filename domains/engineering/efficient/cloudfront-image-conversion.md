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

I would suggest going through the following articles and have a understanding about how the jpg -> webp conversion flow works and then continue here.

[Resizing images with amazon cloudfront lambdaedge aws cdn blog](https://aws.amazon.com/blogs/networking-and-content-delivery/resizing-images-with-amazon-cloudfront-lambdaedge-aws-cdn-blog/)

[Converting images to webp from cdn](https://medium.com/nona-web/converting-images-to-webp-from-cdn-9433b56a3d52)

I followed the step by step section mentioned in the AWS blog. This article contains the bug fixed code and some improvements. Some improvements includes

+ S3 URL with space and %20 bug fixed
+ Dimension support added. You can request for a particular dimension and if it is available then it will be served.


### Step 1: Creating the Deployment Packages
First create a project folder, I named it image-optimization-service . I created a Docker file inside it. I also created the same folder structure as explained.
Find the code repo [here](https://github.com/commutatus/cloudfront-lambda-image-optimization)
```
image-optimization-service  # This is the root folder
image-optimization-service → lambda → viewer-request-function → handler.js
image-optimization-service → lambda → origin-response-function → handler.js
image-optimization-service → Dockerfile
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

image-optimization-service → lambda → viewer-request-function → handler.js
```
const userAgent = require('useragent')
const path = require('path')
const querystring = require('querystring');

const variables = {
  allowedDimension : [ {w:100,h:100}, {w:200,h:200}, {w:300,h:300}, {w:400,h:400} ],
  defaultDimension : {w:200,h:200},
  variance: 20,
  webpExtension: 'webp'
};

exports.handler = async (event, context, callback) => {
  const request = event.Records[0].cf.request
  const headers = request.headers

  // parse the querystrings key-value pairs. In our case it would be d=100x100
  const params = querystring.parse(request.querystring);

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
  let fwdUri = request.uri
  request.headers['originalKey'] = [{
    key: 'originalKey',
    value: fwdUri.substring(1)
  }]

  if (supportingBrowser && params.d ) {
    const fileFormat = path.extname(request.uri).replace('.', '')
    // read the dimension parameter value = width x height and split it by 'x'
    const dimensionMatch = params.d.split("x");

    // set the width and height parameters
    let width = dimensionMatch[0];
    let height = dimensionMatch[1];

    const match = fwdUri.match(/(.*)\/(.*)\.(.*)/);

    let prefix = match[1];
    let imageName = match[2];
    let extension = match[3];

    // define variable to be set to true if requested dimension is allowed.
    let matchFound = false;

    // calculate the acceptable variance. If image dimension is 105 and is within acceptable
    // range, then in our case, the dimension would be corrected to 100.
    let variancePercent = (variables.variance/100);

    for (let dimension of variables.allowedDimension) {
        let minWidth = dimension.w - (dimension.w * variancePercent);
        let maxWidth = dimension.w + (dimension.w * variancePercent);
        if(width >= minWidth && width <= maxWidth){
            width = dimension.w;
            height = dimension.h;
            matchFound = true;
            break;
        }
    }

    // read the accept header to determine if webP is supported.
    let accept = headers['accept']?headers['accept'][0].value:"";

    let url = [];
    // build the new uri to be forwarded upstream
    url.push(prefix);
    url.push(width+"x"+height);

    // check support for webp
    if (accept.includes(variables.webpExtension)) {
        url.push(variables.webpExtension);
    }
    else{
        url.push(extension);
    }
    url.push(imageName+"."+extension);
    request.headers['dimensionIncluded'] = [{
      key: 'dimensionIncluded',
      value: 'true'
    }]
    // fwdUri = url.join("/");
    fwdUri = fwdUri.replace(/(\.jpg|\.png|\.jpeg)$/g, '_'+width+'x'+height+'.webp')
    // final modified url is of format image_100x100.webp
    request.uri = fwdUri;
    request.query = request.querystring

    return callback(null, request);

  } else if (supportingBrowser) {
    request.headers['dimensionIncluded'] = [{
      key: 'dimensionIncluded',
      value: 'false'
    }]
    fwdUri = fwdUri.replace(/(\.jpg|\.png|\.jpeg)$/g, '.webp')
    // final modified url is of format image.webp
    request.uri = fwdUri;
    request.query = request.querystring

    return callback(null, request);
  }
  return callback(null, request)
}

```

image-optimization-service → lambda → origin-response-function → handler.js
```
const http = require('http');
const https = require('https');
const querystring = require('querystring');

const path = require('path')
const AWS = require('aws-sdk')

const S3 = new AWS.S3({
  signatureVersion: 'v4',
})

const Sharp = require('sharp')
const BUCKET = 'BUCKET-NAME'
const QUALITY = 75

exports.handler = async (event, context, callback) => {

  let response = event.Records[0].cf.response

  let request = event.Records[0].cf.request;

  const headers = response.headers

  const request_headers = request.headers

  const originalKey = request_headers.originalkey[0].value


  const dimensionincluded = request_headers.dimensionincluded[0].value

  // if (path.extname(uri) === '.webp') {
  if (parseInt(response.status) === 404) {

    const { uri } = request

    let params = querystring.parse(request.querystring);

    // read the required path. Ex: uri /images/100x100/webp/image.jpg
    let path = request.uri;

    // read the S3 key from the path variable.
    // Ex: path variable /images/100x100/webp/image.jpg
    let newKey = path.substring(1);

    // Ex: file_name=images/200x200/webp/image_100x100.jpg
    // Getting the image_100x100.webp part alone
    let file_name = path.split('/').slice(-1)[0]

    // get the source image file

    try {
      if (dimensionincluded == "false" || dimensionincluded == false) {
        let requiredFormat = file_name.split('.')[1]

        const bucketResource = await S3.getObject({ Bucket: BUCKET, Key: originalKey }).promise()

        // perform the resize operation

        const sharpImageBuffer = await Sharp(bucketResource.Body)
          .webp({ quality: +QUALITY })
          .toBuffer()

        // save the resized object to S3 bucket with appropriate object key.
        await S3.putObject({
          Body: sharpImageBuffer,
          Bucket: BUCKET,
          ContentType: 'image/webp',
          CacheControl: 'max-age=31536000',
          Key: newKey,
          StorageClass: 'STANDARD'
        }).promise()

        // generate a binary response with resized image
        response.status = 200;
        response.body = sharpImageBuffer.toString('base64');
        response.bodyEncoding = 'base64';
        response.headers['content-type'] = [{ key: 'Content-Type', value: 'image/' + requiredFormat }];
        callback(null, response);
      } else {
        // perform the resize operation

        // Getting the 100x100.webp part alone
        let dimension = file_name.split('_')[1].split('.')[0]

        let requiredFormat = file_name.split('_')[1].split('.')[1]

        let width = dimension.split('x')[0]

        let height = dimension.split('x')[1]

        const bucketResource = await S3.getObject({ Bucket: BUCKET, Key: originalKey }).promise()

        const sharpImageBuffer = await Sharp(bucketResource.Body)
          .resize(parseInt(width), parseInt(height))
          .webp({ quality: +QUALITY })
          .toBuffer()
        // save the resized object to S3 bucket with appropriate object key.
        await S3.putObject({
          Body: sharpImageBuffer,
          Bucket: BUCKET,
          ContentType: 'image/webp',
          CacheControl: 'max-age=31536000',
          Key: newKey,
          StorageClass: 'STANDARD'
        }).promise()

        // generate a binary response with resized image
        response.status = 200;
        response.body = sharpImageBuffer.toString('base64');
        response.bodyEncoding = 'base64';
        response.headers['content-type'] = [{ key: 'Content-Type', value: 'image/' + requiredFormat }];
        callback(null, response);
      }
    } catch (e) {
    }
  } else {
    headers['content-type'] = [{
      'value': 'image/webp',
      'key': 'Content-Type'
    }]
  }
  callback(null, response)
 }

```
### Step 2: Building docker
Change directory to `image-optimization-service` thats the root of the project.

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
      CodeUri: s3://image-optimization-service/viewer-request-function-1.zip
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
     CodeUri: s3://image-optimization-service/origin-response-function-1.zip
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

## Deployment Automation

The above settings can be done simply by cloning this [repository](https://github.com/commutatus/image-conversion-automation) and running the following script.

- `./webp_deployment_automation.rb create`


### Things to note

1. After deployment if S3 images are not accessbile even after policy, Enable the read permission for everyone.
2. S3 URL with + and space have been handled here.

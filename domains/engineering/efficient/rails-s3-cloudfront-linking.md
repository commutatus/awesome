---
title: Rails asset serve from cloudfront
nav_order: 5
parent: Efficient
grand_parent: Engineering
---

## Rails asset serve from cloudfront

When it comes to e-commerce sites few main things to note is the SEO and the site load time. When you check the site using online tools few things that it would say is

  1. Serve images using CDN
  2. Image needs to be optimized. Size of images needs to be reduced.

Here we will look at how rails can serve images through CDN. Let us assume we have a bucket named cloudfront-test and we want to serve the contents from this bucket through cloudfront distribution.

### Step 1 - Create cloudfront distribution

Choose the Get started in the website section

No setting change is required. If custom domain name is there, then it can be given. Also create a custom ssl certificate

Click on create distribution. It will take around 15 mins to create distribution

Now that the distribution is created and linked with S3 bucket, Now we can access the images from cloudfront URL.

Eg S3 URL:
```
https://BUCKET-NAME.s3.ap-south-1.amazonaws.com/table-cushion.jpg
```

Eg Cloudfront URL:
``` https://d3xxxxxxxx.cloudfront.net/table-cushion.jpg
```

#### Note:

Cloudfront URL by default have the https enabled. For custom domains SSL certificate needs to be uploaded.



### Rails - Paperclip changes

If you are using rails paperclip gem for uploading images and want to change the default S3 URL to a cloudfront URL where the S3 bucket it associated, these are the changes we need to make.

In the staging/production.rb environment file, make the following change.


```
config.paperclip_defaults = {
    storage: :s3,
    s3_protocol: :https,
    url: ':s3_alias_url',
    s3_host_alias: 'd3xxxxxxx.cloudfront.net', # Place custom URL if you have
    path: ":class/:attachment/:id_partition/:style/:filename",
    s3_credentials: {
      bucket: 'BUCKET-NAME',
      access_key_id: "S3-ACCESS-KEY",
      secret_access_key: "S3-SECRET-KEY",
      s3_region: 'eu-west-1',
      s3_host_name: "s3-eu-west-1.amazonaws.com",
    }
  }
```

#### Note:

Path option will be needed, By default it wont be there. I placed this after checking the paperclip gem.

Without mentioning path there was a problem.

By default the URL will be like

https://s3-eu-west-1.amazonaws.com/BUCKET-NAME/products/avatar/.../.../.../filename.jpg

By doing the above setting the URL changes to

https://d3xxxxx.cloudfront.net/products/avatar/.../.../filename.jpg

If you notice the URL does not have the bucket name appended. Now all the URL will be showing the cloudfront URL.

### TODO

1. Add the settings for Activestorage.

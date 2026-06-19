---
author: Kit
date: 2022-09-06T17:14:16-05:00
lastmod: 2024-11-11T16:04:00-06:00
categories:
- Hugo
tags:
- Hugo
title: Hugo Bitbucket Pipeline to Deploy to S3
---

*Update 11/11/2024*: The `hugo deploy` command was introduced which allows you to handle this entirely using Hugo. The pipeline below
could probably still work but I'm no longer using it.

Previously, this post described how to create a working Bitbucket pipeline for deploying a Hugo static site to AWS S3. 

In the AWS console, create a new IAM user with an access key... because using root is bad. Here is the policy to use, just replace `<your s3 bucket>` with your s3 bucket name.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:List*",
                "s3:Put*",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::<your s3 bucket>",
                "arn:aws:s3:::<your s3 bucket>/*"
            ],
            "Effect": "Allow",
            "Sid": "AllowPipelinesDeployAccess"
        }
    ]
}
```

Add `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `HUGO_VERSION` to your Bitbucket repository variables:

![Repository variables](Screen%20Shot%202022-09-06%20at%205.17.35%20PM.png)

Then, in your repository root, create `bitbucket-pipelines.yml`:

```yaml
image: atlassian/default-image:3

options:
    max-time: 5

pipelines:
    default:
      - step:
          name: Build Hugo
          script:
            - apt-get update -y && apt-get install wget
            - apt-get -y install git
            - echo Hugo version is $HUGO_VERSION
            - export HUGO_ENV=production
            - wget https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_Linux-64bit.deb
            - dpkg -i hugo*.deb
            - git submodule update --init --remote
            - hugo --minify
          artifacts:
            - public/**
      - step:
          name: Deploy to S3
          deployment: production
          script:
            - pipe: atlassian/aws-s3-deploy:1.1.0
              variables:
                AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
                AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
                AWS_DEFAULT_REGION: 'us-east-2'
                S3_BUCKET: 'kitmenke.com'
                LOCAL_PATH: 'public'
```

Some tricky things... the `git submodule update --init --remote` was needed in order for the theme to get downloaded properly.

Sources:

 - [Deploying Hugo to S3 via Bitbucket](https://www.ronlancaster.com/posts/deploying-hugo-to-s3-via-bitbucket.html) - pretty much the right bitbucket pipeline template, a little out of date
 - [Deploying a Hugo website to Amazon S3 using Bitbucket Pipelines](https://www.karelbemelmans.com/2016/10/deploying-a-hugo-website-to-amazon-s3-using-bitbucket-pipelines/) - s3 permissions setup
 - [Hugo Deployments with Bitbucket Pipelines](https://blog.vianetz.com/2020/08/hugo-deployment-bitbucket-pipelines/)
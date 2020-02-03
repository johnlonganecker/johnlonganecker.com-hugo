---
title: "My First Post - draft/test"
date: 2020-01-16T10:53:10-08:00
draft: true
---
# Hosting a Hugo on S3 and deploy using Github Actions

I have been meaning to create a blog to 

1. Install hugo CLI and setup a sample site - follow [these directions](https://gohugo.io/getting-started/quick-start/)
2. S3
  - Create S3 Bucket, User and Policy
    - make bucket public
    - make policy
    - create user
    - attach policy to user directly
    - save s3 credentials
  - Create Access Keys/Secret
3. Github Actions
  - push to s3
  - store access key/secret in github Secrets
4. Setup S3 Deploy
  - delete old files
5. Preview Workflow
  - s3 drafts hide http auth
6. cool

bucket policies https://aws.amazon.com/premiumsupport/knowledge-center/secure-s3-resources/

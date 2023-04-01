---
layout: post
title: How to create a CloudFront distribution that serves static content from S3
  with CloudFormation
date: 2023-04-01 11:24 +0000
---
This is what we're going to create:

```
                +------------------------------------------------+
                | AWS Account                                    |
                |                                                |
+----------+    |    +--------------+        +---------------+   |
|          |    |    |              |        |               |   |
|  Client  +----+--->|  CloudFront  +------->|   S3 Bucket   |   |
|          |    |    |              |        |               |   |
+----------+    |    +--------------+        +---------------+   |
                |                                                |
                |                                                |
                +------------------------------------------------+
```

The architecture keeps our S3 bucket (where our static content lives) private. It can only be accessed
via CloudFront. This is great for two main reasons:

- Accessing content from a bucket is slower than from the CloudFront edge cache.
- Accessing content from the cache is much cheaper than reading from S3 in large amounts.

I automated the creation of the whole stack with a CloudFormation template:

```yml
Resources:
  # An S3 bucket that holds our site's static content. This bucket is totally private.
  Bucket:
    Type: AWS::S3::Bucket
    Properties:
      AccessControl: Private
      OwnershipControls:
        Rules:
          - ObjectOwnership: BucketOwnerEnforced
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
  # Attach a bucket policy that allows access to the content in the bucket only for the CloudFront
  # distribution that is created later. This prevents a user from directly accessing our bucket.
  BucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref Bucket
      PolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Sid: AllowCloudFrontRead
            Effect: Allow
            Principal:
              Service: cloudfront.amazonaws.com
            Action: s3:GetObject
            Resource: !Sub ${Bucket.Arn}/*
            Condition:
              StringEquals:
                AWS:SourceArn: !Sub arn:aws:cloudfront::${AWS::AccountId}:distribution/${Distribution.Id}
  # This OriginAccessControl is what the CloudFront distribution will use to sign requests to prove
  # to the bucket policy that it has access.
  OriginAccessControl:
    Type: AWS::CloudFront::OriginAccessControl
    Properties:
      OriginAccessControlConfig:
        Name: !GetAtt Bucket.RegionalDomainName
        OriginAccessControlOriginType: s3
        SigningBehavior: always
        SigningProtocol: sigv4
  # Cache everything for 60 seconds, unless the object has a custom cache header.
  CachePolicy:
    Type: AWS::CloudFront::CachePolicy
    Properties:
      CachePolicyConfig:
        DefaultTTL: 60
        MaxTTL: 300
        MinTTL: 0
        Name: DefaultCachePolicy
        # Don't forward anything from the user - we only cache static content.
        ParametersInCacheKeyAndForwardedToOrigin:
          CookiesConfig:
            CookieBehavior: none
          EnableAcceptEncodingGzip: true
          HeadersConfig:
            HeaderBehavior: none
          QueryStringsConfig:
            QueryStringBehavior: none
  # Create a CloudFront distribution that targets the S3 bucket.
  # / corresponds to an index.html in the bucket.
  Distribution:
    Type: AWS::CloudFront::Distribution
    Properties:
      DistributionConfig:
        DefaultRootObject: index.html
        DefaultCacheBehavior:
          CachePolicyId: !Ref CachePolicy
          TargetOriginId: !GetAtt Bucket.RegionalDomainName
          ViewerProtocolPolicy: redirect-to-https
        Enabled: true
        HttpVersion: http2and3
        Origins:
          - DomainName: !GetAtt Bucket.RegionalDomainName
            Id: !GetAtt Bucket.RegionalDomainName
            OriginAccessControlId: !Ref OriginAccessControl
            # The schema mandates we specify this, but we use an OAC rather 
            # than an OAI, so don't specify one.
            S3OriginConfig:
              OriginAccessIdentity: ""
Outputs:
  DomainName:
    Value: !GetAtt Distribution.DomainName
```

Assuming you save this as `template.yml`, you can create a `website` stack with this command:

```bash
aws cloudformation create-stack --stack-name website --template-body file://template.yml
```

It takes a few minutes for the distribution to be deployed into all edge locations. While you wait,
upload your static content into the created bucket, which should be named similarly to `website-bucket-bu1bdapqunak`.
Here's an example `index.html` you could use:

```html
<!DOCTYPE html>
<html>
    <head>
        <title>CloudFront Site</title>
    </head>
    <body>
        <h1>Hello, World!</h1>
    </body>
</html>
```

When the stack has completed, it outputs the domain name of the distribution. You can navigate to this to view
your content!

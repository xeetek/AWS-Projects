## 🚨Certificate in ACM needs to be created in us-east-1 🚨
# Static Website Hosting using CloudFront

# Table of Contents:
- [Introduction](#introduction)
- [S3](#s3)
- [R53](#r53)
- [CloudFront](#cloudfront)
- [Final thoughts](#final-thoughts)

## Introduction
Project helps to understand the connection between a S3 origin by making a static website and CloudFront. This project will most likely be expanded on when knowledge of other AWS services are introduced into my learning. 


![image](https://github.com/user-attachments/assets/4d89104e-d17b-4dc4-bb3b-2b261a8e4caf)



## S3
1. Create bucket with globally unique name - in this case I set it to be of website-zacharyjanssen.com (registered domain name)
2. Uncheck "Block all public access"
3. Create Bucket
4. After bucket creation, I went into the properties of the bucket, scrolled all the way down, and enabled "Static Website Hosting"
5. Next, I went to the permissions tab of the bucket and added the bucket policy to allow public read access 

### BUCKET POLICY:
```{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadAccess",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::BUCKET_NAME/*"
        }
    ]
}
```

6. Upload index.html to bucket
7. Go to bucket properties, scroll down to the bottom and copy the bucket endpoint and open that in a new tab. Here you can see the website!

![image](https://github.com/user-attachments/assets/2b46469f-c8fc-46a7-bde2-5faf0459a30a)

## R53
1. If you don't already have a public hosted zone created with the registered domain, create one
2. We will come back to R53 in a bit after we get through the CloudFront section. 

## CloudFront
1. Go to the "Distributions" option > create distribution
2. Under Origin, choose orgiin > select the s3 bucket
3. For Origin access, we selected "Origin access control settings" This was selected because we only want CloudFront to be able to access the contents of the S3 bucket
and not have users to access the bucket directly. Create new OAC > change nothing > create
4. A bucket policy will need to be created to allow CloudFront to access S3 objects
5. Delete out the old S3 bucket policy and add the one below:
```
   {
    "Version": "2008-10-17",
    "Id": "PolicyForCloudFrontPrivateContent",
    "Statement": [
        {
            "Sid": "AllowCloudFrontServicePrincipal",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::cfands3-top10cats-k4fh13v5ozfv/*",
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": "arn:aws:cloudfront::<youraccountid>:distribution/EG8ANTQKNNWS0"
                }
            }
        }
    ]
}
```
6. Viewer Protocol Policy, select Redirect HTTP to HTTPS
7. For "Alternate domain name (CNAME) we want to add an item and put our registered domain name - zacharyjanssen.com
8. Next we need a Custom SSL certificate since this is for a custom domain name
9. Request certificate, choose public certificate and in the FQDN type in the registered domain name

![image](https://github.com/user-attachments/assets/723d34de-c0fb-416f-ab41-e038a33977cb)

8. After the certificate is done being created, it will prompt you to add the route to R53, click add routes. You can confirm by setting this entry in R53

![image](https://github.com/user-attachments/assets/020e8f4d-f0de-489c-bd7a-01d52c91c97d)
Notice the CNAME record and the acm-validations.aws value

8. Under "Default root object" back in the distributions settings, put the index.html object to be referenced when users connect to the site

## R53
1. Go to your public hosted zone, and then "Create Record"
2. We want the "Simple Routing" option
3. Click "Define simpple record"
4. Leave the subdomain blank, keep Record type as "A"
5. Under Value, select "Alias to CloudFront Distribution"
6. Select the distribution
7. Wait for that deploy
8. Go to your website using registered domain

## Final thoughts
You need to create the certificate in us-east-1 if you're going to use an ACM certificate with CloudFront. My first walk-through of this, I forgot the "Alternate Domain Name" when setting up CloudFront, so it would never connect using my registered domain name. Since CloudFront caches the data on the edge location, trying to update my website and see the changes, doesn't happen real time. I have to run an "Invalidation" on everything in the bucket so it clears out the cache. I could work around this by renaming the index.html to index<version_Number>.html, but then I'm having to change the default object each time. Since I'm using CloudFront, I don't think there is a good way of updating content - what I could do is lower the TTL of the cache and have it update more often than the default of 24 hours. 

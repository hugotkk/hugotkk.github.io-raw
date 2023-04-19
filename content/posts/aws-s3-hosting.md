---
title: "How to host a static website on S3"
date: 2023-03-29
tags:
- cloud
- aws
- devops
- s3
- iam
---

## Concepts

### Block Public Access
* Prevent any public access in ACL or bucket policy.
* ACL: If you block public access in ACL, any public access granted to everyone will be ignored.
* Bucket Policy: If you block public access in bucket policy, any policy that allows S3 operations by * will be ignored. eg:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::Bucket-Name/*"
            ]
        }
    ]
}
```

### Object Ownership
* Ensure that all objects are owned by the bucket owner and disable ACLs.
* Choose between 
  * disabling ACL
  * enabling ACL while keeping ownership the same as the bucket owner by default but it can be overrided by the object writer
  * enabling ACL while keeping ownership the same as the object writer

### Static Website Hosting
* Set a default index page and error page.
* Provide a URL for access. (not same as the s3 URL)

### Access Control
* Use ACL to control access rights on an object level.
* Note that enabling ACL may cause varied object ownership in shared buckets.
* Use bucket policy to control access at the bucket level.
* Keep in mind that bucket policies only apply to objects owned by the bucket owner.

### Access Points
* Simplify policy management by attaching a policy to an access point.
* Use Access point policy to override existing bucket policies for granular control.
* Use S3 Access Points to reduce the risk of policy errors or inconsistencies.

## Steps ([How to setup a public accessible S3 bucket](https://www.youtube.com/watch?v=4zrupVYqQFs))

* Configure Static Website Hosting in Properties
* Disable Object Ownership in Permissions
* Enable public access With ACL
  * Disable block public access of ACL in the Permissions.
  * Go to Objects, select objects, then click "make public using ACL" (This can be done by AWS CLI as well).
* Or via Bucket Policy
  * Disable block public access of bucket policy in the Permissions.
  * Create a bucket policy that allows everyone to get objects.

## Integrate with CloudFront

Instead of using static website hosting or making S3 public, we can use CloudFront to host a static web host with S3. 

Here's how:

1. Create a distribution and S3 origin.
2. To define the index page, set the root object in the distribution level for requesting `/`.
3. We can use a private S3 bucket and use Origin Access Control to grant permission for CloudFront to access the bucket. OAI (Origin Access Identity) is now considered legacy, and it's recommended to use Origin Access Control instead.
4. We can control policies for caching and manage headers, cookies, and query strings:
    - Viewer
    - Viewer + CloudFront
    - Viewer + Exclude
    - Specific (filter from Viewer + CloudFront)
5. Add or remove headers, CORS, and security headers.
6. Add a whitelist or blacklist Geo Restriction to restrict access based on geography.


---
author: volodymyr
comments: true
date: 2016-03-24 00:03:00+00:00
layout: post
slug: aws-s3-presigned-urls-magic
title: Amazon S3 &mdash; How to invalidate a pre-signed URL
categories:
- AWS
tags:
- Amazon
- AWS
- S3
---

Let's imagine you need to allow third-party to access your content on Amazon S3.
Without using the Amazon CloudFront, you can share it using a pre-signed URLs
with permission to perform **GetObject** operation os a specific resource.

But what if at some point you decide you no longer want to allow access to your
content?

This blog post describes possible approaches of sharing content using pre-signed URLs and
controlling access to the content once a pre-signed URL was issued.

<!-- more -->

Before we start, let's see what is required to access some object in a S3 bucket:
* a user
* AWS credentials
* a policy that allows this particular user to access an AWS resource(e.g. a bucket) using some particular operation (e.g. **GetObject**)

Pre-signed URLs encapsulate information about the user, allowed operations and validity period.
We can't modify validity period of a pre-signed URL once it was issued, so to invalidate it, we must use different approaches. Actually, there are several approaches that you can use depending on your needs.

## Quick & dirty: user removal
If the user that created a pre-signed url no longer exists, all their pre-signed URLs
become inaccessible.

### Pros
Easy to implement

### Cons
This approach is completely irreversible. Once the user is removed, there is no way
to repair pre-signed URLs that were generated earlier.

## Quick & dirty 2: credentials removal

---
title: awk-sdk-python验证S3服务
description: 使用boto3验证S3服务
categories:
- S3
abbrlink: 5a7446d2
date: 2021-08-23 17:07:23
tags:
- TODO
---
> How to use awk-sdk-python to validate access to s3 server

1. Prerequisites
Install the AWS SDK for Python from [here](https://aws.amazon.com/cn/sdk-for-python/)

2. Example
Please replace endpoint_url,aws_access_key_id, aws_secret_access_key, Bucket and Object with your local setup in this example.py file.
```python
#!/usr/bin/env python
# _*_ coding:utf-8 _*_

from boto3.session import Session

access_key = "minioadmin"
secret_key = "minioadmin"
endpoint_url = "http://localhost:9000"
bucket_name = "my-bucket"

# Connect to s3
session = Session(aws_access_key_id=access_key, aws_secret_access_key=secret_key)
s3 = session.resource("s3", endpoint_url=endpoint_url)

# List all the existing buckets
for bucket in s3.buckets.all():
    print("bucket name:%s" % bucket.name)

# Create a new bucket
s3.create_bucket(Bucket=bucket_name)
bucket = s3.Bucket(bucket_name)

# Upload a file from local file system '/home/root/file.txt' to bucket
bucket.upload_file("/home/root/file.txt", "file.txt")

# List all objects
for obj in bucket.objects.all():
    print("obj name:%s" % obj.key)

# Download the object 'file.txt' from the bucket and save it to local file system
bucket.download_file("file.txt", "/tmp/file.txt")

# Delete all the objects
bucket.objects.filter().delete()

# Delete bucket
bucket.delete()
```

`TODO: resource和client的差别`
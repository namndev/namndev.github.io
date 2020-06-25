---
layout: post
title: Uploading a Large File to Amazon S3
image: https://i.ya-webdesign.com/images/amazon-s3-png-11.png
tags: [AWS, DevOps]
comments: true
---

The largest single file that can be uploaded into an Amazon S3 Bucket in a single PUT operation is 5 GB. If you want to upload large objects (> 5 GB), you will consider using multipart upload API, which allows to upload objects from 5 MB up to 5 TB.

The Multipart Upload API is designed to improve the upload experience for larger objects, which can be uploaded in parts, independently, in any order, and in parallel. The AWS tool to use to perform this is [API-Level (s3api)](http://docs.aws.amazon.com/cli/latest/reference/s3api/) command set.

In this tutorial, we assume:

* You have installed and configured [AWS Command Line Interface](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) on a Linux OS computer/server,
* You have an Amazon account and a [S3 Bucket](http://aws.amazon.com/s3) (MyBucketName),
* The size of the file to upload is 681 MB (file_video.mp4),
* 100 MB can be uploaded without problem using our internet connection.


## Theoretically, how it works

The process involves in 4 steps:

1. Separate the object into multiple parts. There are several ways to do this in Linux, `dd`, `split`, etc. We will use `dd` in this tutorial.
2. Initiate the multipart upload and receive an upload id in return (`aws s3api create-multipart-upload`).
3. Upload each part (a contiguous portion of an object’s data) accompanied by the upload id and a part number (`aws s3api upload-object`).
4. Finalize the upload by providing the upload id and the part number `/ETag` pairs for each part of the object (`aws s3api complete-multipart-upload`).

## And practically?

### 1. Separate the object into multiple parts

We will create 205 parts (100 MB * 204 + 80 MB):

```bash
$ split -b 100M file_video.mp4
```
output

```bash
xaa
xab
xac
xad
xae
xaf
xag
```
### 2. Initiate the multipart upload and receive an upload id in return

```bash
$ aws s3api create-multipart-upload --bucket MyBucketName --key file_video.mp4
```

You will received as output something like:

```json
{
"UploadId": "UVditMTG8U--MyLongUploadId--ksmFT7N6bNTWD",
"Bucket": "MyBucketName",
"Key": "file_video.mp4"
}
```

### 3. Upload each part

For the following commands, note the console output:

```json
{
"ETag": "\"fggcd799--ETagValue1--dhe76dd8dc\""
}
```

```bash
$ aws s3api upload-part --bucket MyBucketName --key file_video.mp4 --upload-id UVditMTG8U--MyLongUploadId--ksmFT7N6bNTWD --part-number 1 --body xaa

$ aws s3api upload-part --bucket MyBucketName --key file_video.mp4 --upload-id UVditMTG8U--MyLongUploadId--ksmFT7N6bNTWD --part-number 2 --body xab

$ aws s3api upload-part --bucket MyBucketName --key file_video.mp4 --upload-id UVditMTG8U--MyLongUploadId--ksmFT7N6bNTWD --part-number 3 --body xac

$ aws s3api upload-part --bucket MyBucketName --key file_video.mp4 --upload-id UVditMTG8U--MyLongUploadId--ksmFT7N6bNTWD --part-number 4 --body xad

$ aws s3api upload-part --bucket MyBucketName --key file_video.mp4 --upload-id UVditMTG8U--MyLongUploadId--ksmFT7N6bNTWD --part-number 5 --body xae

$ aws s3api upload-part --bucket MyBucketName --key file_video.mp4 --upload-id UVditMTG8U--MyLongUploadId--ksmFT7N6bNTWD --part-number 6 --body xaf

$ aws s3api upload-part --bucket MyBucketName --key file_video.mp4 --upload-id UVditMTG8U--MyLongUploadId--ksmFT7N6bNTWD --part-number 7 --body xag
```

    Note: Once more you can write a small shell script to automate this process.

### 4. Finalize the upload

Create a JSON file `MyMultiPartUpload.json` containing the following:

```json
{
  "Parts": [
    {
      "ETag": "\"fd7c49f1d951b59b9833f59f05f39216\"",
      "PartNumber": 1
    },
    {
      "ETag": "\"1cd7378736edbebcd0de78d9ed55e574\"",
      "PartNumber": 2
    },
    {
      "ETag": "\"02f0980fe099ef225ce8a75a897dd3fe\"",
      "PartNumber": 3
    },
    {
      "ETag": "\"7fc958fc1b7a1b17fda1107d1fcf3ca2\"",
      "PartNumber": 4
    },
    {
      "ETag": "\"928e4431e9e381e06acbbef3bc274759\"",
      "PartNumber": 5
    },
    {
      "ETag": "\"9ed030abbb81c400a04fe9e907eae2fc\"",
      "PartNumber": 6
    },
    {
      "ETag": "\"74571e2ea10a0514c75f35ef5d26a78b\"",
      "PartNumber": 7
    }
  ]
}
```

```bash
4 aws s3api complete-multipart-upload --bucket MyBucketName --key \
file_video.mp4 --upload-id UVditMTG8U--MyLongUploadId--ksmFT7N6bNTWD --multipart-upload file://../MyMultiPartUpload.json
```

output:

```json
{
    "Location": "https://MyBucketName.s3.ap-southeast-1.amazonaws.com/file_video.mp4",
    "Bucket": "MyBucketName",
    "Key": "file_video.mp4",
    "ETag": "\"0f3723256112d0d880596cd858601e54-7\""
}
```

That is all, you can verify that the large file is uploaded with:

```bash
$ aws s3 ls s3://MyBucketName/file_video.mp4
2020-03-06 10:59:57  695363423 file_video.mp4
```
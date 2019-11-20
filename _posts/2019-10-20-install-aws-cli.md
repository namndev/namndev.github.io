---
layout: post
title: Installing the AWS CLI - AWS Command Line Interface
image: https://stackify.com/wp-content/uploads/2017/09/AWS-CLI-Header-min-793x397.png
tags: [AWS, DevOps]
comments: true
---

# I. Install AWS-CLI in Centos7
There are a number of ways to install the AWS command line tools. Below is the method I favour on CentOS:

```bash
$curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
$unzip awscli-bundle.zip
$cd awscli-bundle/
$sudo ./install -i /usr/local/aws -b /usr/local/bin/aws
```
Check that was successful with:

```bash
$aws --version
aws-cli/1.16.256 Python/2.7.5 Linux/3.10.0-957.12.2.el7.x86_64 botocore/1.12.246
```
You can now remove the files extracted:

```bash
$rm -vR ~/awscli-bundle/
$rm -v ~/awscli-bundle.zip
```
Configure aws-cli to use your AWS credentials:

```bash
$aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: eu-west-1
Default output format [None]: json
```
You can test your AWS credentials with the below command.

```bash
$aws sts get-caller-identity
{
    "Account": "903503371367", 
    "UserId": "AKIAIOSFODNN7EXAMPLE", 
    "Arn": "arn:aws:iam::903503371367:user/andy"
}
```

# II. Install AWS-CLI in MacOS

aws-cli yêu cầu máy bạn phải cài đặt phiên bản Python tối thiểu là 2.6.5.

Có thể kiểm tra phiên bản python đang sử dụng trên máy bằng lệnh:

```bash
$python --version
```
## 1.Cài đặt bằng Bundle Installer
Tải bộ cài aws-cli:

```bash
$curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
```
Giải nén bộ cài đặt:

```bash
$unzip awscli-bundle.zip
```
Cài đặt:

```bash
$sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
```
## 2. Cài đặt bằng pip
Cài đặt bằng lệnh:

```bash
$pip3 install awscli --upgrade --user
```
Kiểm tra lại xem đã cài đặt được chưa bằng lệnh:

```bash
$aws --version
```
Để cập nhật phiên bản mới của aws-cli có thể dùng lệnh:

```bash
$pip3 install awscli --upgrade --user
```
Sau khi cài đặt aws-cli, bạn cần thêm cấu hình vào trong biến môi trường của hệ điều hành như sau:

Kiểm tra thư mục cài đặt của python trên máy:

```bash
$which python
```
Giả sử thư mục cài đặt là ~/Library/Python/3.7/bin, ta cần thêm dòng sau vào profile script:

```bash
export PATH=~/.local/bin:$PATH 
```

Tải lại file profile bằng lệnh:

```bash
$source ~/.bash_profile
```

## 3. Cấu hình cho aws-cli

Chạy lệnh sau:

```bash
$aws configure
```
Người dùng sẽ cần nhập các thông tin sau trong màn hình hiện ra:

```bash
AWS Access Key ID [None]:
AWS Secret Access Key [None]: 
Default region name [None]: 
Default output format [None]: json
```

Người dùng có thể lấy các thông tin trên bằng cách:

 - Đăng nhập vào console.aws.amazon.com
 - Truy cập vào chức năng IAM console.
 - Chọn Users.
 - Chọn User cần sử dụng.
 - Trong màn hình hiện ra, chọn tab Security credentials và chọn Create access key.
 - Trong màn hình tạo, chọn show để lấy thông tin.

 > Thanks!
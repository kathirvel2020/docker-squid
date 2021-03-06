# Docker in Production using AWS Proxy Docker Image

This repository defines a Docker Image for running [Squid](http://www.squid-cache.org) as a forward proxy in AWS environments.

## Building the Image

To build this image the following prerequisites are required:

- Docker Client with access to a Docker Engine (1.12 or higher)
- Docker Compose 1.7 or higher
- GNU Make 3.82 or higher
- AWS CLI 1.10 or higher
- AWS profile/environment configured with privileges to push images to the ECR repository

To build the image use the `make release` command:

```
$ make release
Building squid
Step 1 : FROM alpine
latest: Pulling from library/alpine
Digest: sha256:1354db23ff5478120c980eca1611a51c9f2b88b61f24283ee8200bf9a54f2e5c
Status: Image is up to date for alpine:latest
 ---> baa5d63471ea
Step 2 : MAINTAINER Justin Menga <justin.menga@gmail.com>
 ---> Using cache
 ---> f0ecea783770
...
...
Step 11 : CMD squid -f /etc/squid/squid.conf -NYCd 1
 ---> Using cache
 ---> ad9b93514414
Successfully built ad9b93514414
=> Build complete
=> Starting squid service...
Creating network "squid_default" with the default driver
Creating squid_squid_1
=> Release environment created
=> Squid is running at http://172.16.154.128:32779
```

After building the image, you can test the image locally by configuring your browser or environment to use the Squid proxy URL output displayed at the end of the `make release` command.

### Tagging the Image

After building the image, you can tag the image using the `make tag` or `make tag [<tag>...]` command:

```
$ make tag
=> Tagging release image with tags latest 20161211161608.52ce7ab 52ce7ab...
=> Tagging complete
```

### Publishing the Image

With the image tagged, you can login to the AWS EC2 Container Service Registry (ECR) and publish the image:

```
$ make login
=> Logging in to Docker registry ...
Enter MFA code: xxxxxx
Login Succeeded
=> Logged in to Docker registry
$ make publish
=> Publishing release image to 543279062384.dkr.ecr.us-west-2.amazonaws.com/dockerproductionaws/squid...
The push refers to a repository [543279062384.dkr.ecr.us-west-2.amazonaws.com/dockerproductionaws/squid]
eb40ed4586e2: Pushed
d93c9b2eda1f: Pushed
66fb5c668a31: Pushed
02535d447192: Pushed
011b303988d2: Pushed
20161211161608.52ce7ab: digest: sha256:f347746ec71c7a1fc00f534af27392b0eec5b8d300c191bb87e74753f7b9bcd6 size: 7708
...
...
=> Publish complete
```

### Cleaning up

To clean up after building, tagging and publishing the image, use the `make clean` command:

```
$ make clean
=> Destroying release environment...
Stopping squid_squid_1 ... done
Removing squid_squid_1 ... done
Removing network squid_default
=> Removing dangling images...
Deleted: sha256:ec70a518b87d42e63a9b05f3a4324a8a6e3d57e941ed106d43bea67a35ecca92
Deleted: sha256:d65f4e883440a995add44493bdc1ae6d565bd06f9d5aca471afed1b4423cd2ec
Deleted: sha256:ce7014ef547d1c5069148e33b57ce34c85308de37ef6d10408f842f152154145
Deleted: sha256:b21f12ac9c4cd0aa1673d305186717035dbd2cd1e4efc8829c9f2107f76d306d
Deleted: sha256:4e1dcefb3ef4387d035ac542694b39d51c499005240a343e7b5a4c586f85714b
Deleted: sha256:c9698d6fd30176afdc71f95c78259c376801e07a7065b52f0739ea422d808bbe
Deleted: sha256:8baa4adc1cff29eb794b28062ac0ea231503ed984f89d2a71af54b2ff11d1f98
Deleted: sha256:5b6baf58e99ba6d79022d17a510d8e58b418c10bf2a09b429941a04792cec6b7
Deleted: sha256:a4308d3bb2b335a505bbdb7b79fa5ddbe4e773919260a95dc8b497612d7df3ab
Deleted: sha256:9d3e54d0d7900c6d93e7081613e5229f266d919dc105f8d848a57262f1058326
Deleted: sha256:e800bfff628f614dabb01d7b6ea7438deb4242cf24909a554808499b2ee6870b
=> Clean complete
```

## Runtime Configuration

### Squid Whitelist

A default whitelist template is included with this image that permits access to AWS services.

The whitelist is generated by a whitelist template that is created on container startup based upon environment variables supplied to the container (see the Squid Configuration section below).

The following is the default whitelist that is generated if no configuration is provided to the container:

```
.us-west-2.amazonaws.com
.s3-us-west-2.amazonaws.com
iam.amazonaws.com
sts.amazonaws.com
support.us-east-1.amazonaws.com
waf.amazonaws.com
cloudfront.amazonaws.com
route53.amazonaws.com
route53domains.us-east-1.amazonaws.com
devicefarm.us-west-2.amazonaws.com
importexport.amazonaws.com
```

### Squid Configuration 

The following table defines environment variables control the configuration of containers created from this image.

| Environment Variable | Required | Default Value  | Description                                                                                                                                               | Examples                     |
|----------------------|----------|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------|
| SQUID_WHITELIST      | No       |                | Comma separated list of whitelisted domains.  Note that this whitelist adds the default whitelist that permits access to AWS services.                    | .dockerproductionaws.org,.google.com |
| AWS_REGIONS          | No       | us-west-2      | Comma separate list of regions that the whitelist should perform for access to AWS services.  This only affects AWS services that are regional in nature. | ap-southeast-2,us-west-2             |
| ALLOWED_CIDRS        | No       | RFC1918 ranges | Comma separated list of allowed CIDR ranges permitted to use the Proxy.  This typically should be set to the CIDR block range of your VPC.                | 192.168.200.0/20                     |
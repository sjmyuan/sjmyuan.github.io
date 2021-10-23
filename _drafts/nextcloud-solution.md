---
title: An off-the-self and cheap home server solution
tags:
- fun
---

# Why

I always wanted to have a personal home server to store family's files and photos.

There are some off-the-self solutions, Synology for example. But it's too expensive to spend more than 2000￥ to store files. 

Considering I have a laptop which is almost 10 years old but has 1T storage, I try to build one by myself.

# What

My requirements for the home server are

1. File syncing and access on different platforms, such as Windows, MacOS, iOS, Android
2. Media stream and preview, like video and photos
3. Cloud backup
4. Internet access
5. Support Docker

So my solution are Docker + Nextcloud + rclone + IPv6 + DDNS + AWS Route53 + AWS S3

# How

## Nextcloud

Nextcloud already supplied an [docker-compose example](https://github.com/nextcloud/docker/blob/master/.examples/docker-compose/with-nginx-proxy/mariadb/apache/docker-compose.yml) to set up everything, including database, cronjob, Nextcloud app, proxy and https.

But if we want to support the preview of image and video, the Nextcloud docker image need to install ffpmpeg, so we need to build our own image like this:

```dockerfile
FROM nextcloud

RUN apt-get update
RUN apt-get install -y ffpmpeg
```

Then we can run docker-compose up to start the nextcloud, which will generate the config.php

Then we need to add the following configuration to enable preview

```php
  'enable_previews' => true,
  'preview_max_x'   => 2048,
  'preview_max_y'   => 2048,
  'enabledPreviewProviders' => 
  array (
    0 => 'OC\\Preview\\TXT',	  
    1 => 'OC\\Preview\\MarkDown',	  
    2 => 'OC\\Preview\\OpenDocument',	  
    3 => 'OC\\Preview\\PDF',	  
    4 => 'OC\\Preview\\MSOffice2003',	  
    5 => 'OC\\Preview\\MSOfficeDoc',	  
    6 => 'OC\\Preview\\Image',	  
    7 => 'OC\\Preview\\Photoshop',	  
    8 => 'OC\\Preview\\TIFF',	  
    9 => 'OC\\Preview\\SVG',	  
    10 => 'OC\\Preview\\Font',	  
    11 => 'OC\\Preview\\MP3',	  
    12 => 'OC\\Preview\\Movie',	  
    13 => 'OC\\Preview\\MKV',	  
    14 => 'OC\\Preview\\MP4',	  
    15 => 'OC\\Preview\\AVI',	  
  )

```

Add the following configuration to support the https protocol

```php
  'overwriteprotocol' => 'https',
```

Add your domain to the trusted domains

```php
  'trusted_domains' => 
  array (
    0 => '<your domain>',
  ),
```

Then we can use `docker-compose up -d` to start Nextcloud as background service.

If you already have the public IPv4 address, after creating an A record in your DNS service
congratulation, [bunch of apps](https://apps.nextcloud.com/) are ready for you.

If you can't get the public IPv4 address like me, don't worry, we still have IPv6.

## IPv6

### Availability Checking

1. Network provider
2. Router support IPv6 and won't block IPv6 request
### Docker

To make Docker support IPv6, first we need to add the following configuration to docker config

```config
```

Then we need to enable the IPv6 network in docker compose

```yaml
```

### Firewall

Usually linux system will block all IPv6 income traffic, we can use the following command to allow the traffic

```sh
```

Actually there is a risk to allow all traffic, we just want to all traffic from given port, if you familiar with the command please let me know.

## DDNS and AWS Route53

Sometimes our network provider won't supply the static IPv6 or IPv4 address, the address will be refreshed periodically, so we need a DDNS service to make sure our address is always correct.

Here we use `crazymax/ddns-route53` to implement DDNS on AWS Route53

```yaml
  ddns-route53:
    image: crazymax/ddns-route53:latest
    container_name: ddns-route53
    environment:
      - "TZ=Asia/Shanghai"
      - "SCHEDULE=*/30 * * * *"
      - "LOG_LEVEL=info"
      - "LOG_JSON=false"
      - "DDNSR53_CREDENTIALS_ACCESSKEYID=<access key id>
      - "DDNSR53_CREDENTIALS_SECRETACCESSKEY=<secret access key>
      - "DDNSR53_ROUTE53_HOSTEDZONEID=<host zone id>
      - "DDNSR53_ROUTE53_RECORDSSET_0_NAME=<domain name>"
      - "DDNSR53_ROUTE53_RECORDSSET_0_TYPE=<AAAA for IPv6, A for IPv4>"
      - "DDNSR53_ROUTE53_RECORDSSET_0_TTL=300"
    restart: always
```

## rclone

We can not trust an old laptop, so we'd better backup our data to some remote storage, such as S3, Box, Google Drive, etc.

Here we use [rclone](https://rclone.org/) on the host to backup the data to AWS S3, which is easier than the docker solution. 

1. Install the rclone on your computer according to the [installation documentation](https://rclone.org/install/#script-installation)
2. Configure your AWS credentials according to the [AWS S3 Remote documentation](https://rclone.org/s3/#amazon-s3).
3. Configure cron job to back up your file periodically
   
   We need to back up all the volume of our docker compose, here we just simple back up all the files in our work directory of Nextcloud.

   ```sh
    #!/bin/bash

    if pidof -o %PPID -x “rclone-cron.sh”; then # we assume the file name is rclone-cron.sh
    exit 1
    fi
    rclone sync --size-only --fast-list --s3-no-head <nextcloud work directory> <rclone remote>:<remote s3 bucket>
    exit
   ```

   ```sh
   # back up at 1:00AM
   0 1 * * * <rclone-cron.sh path> >/dev/null 2>&1
   ```

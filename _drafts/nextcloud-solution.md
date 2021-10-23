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

1. Nextwork provider
2. Home Router support IPv6 and won't block IPv6 request
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

Sometimes Network provider won't supply the static IPv6 or IPv4 address, the address will be refreshed after a period, so we need a DDNS service to make sure our address is always correct.

## rclone

# Summary


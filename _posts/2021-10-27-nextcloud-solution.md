---
title: An off-the-shelf and cheap home server solution
tags:
- fun
date: 2021-10-27 23:01 +0800
---
# TLDR

I already created a repo [my-nextcloud](https://github.com/sjmyuan/my-nextcloud), which contains everything in this blog. 
# Why

I always wanted to have a personal home server to store family photos.

There are some business solutions, Synology, Box, Google Drive for example. But it's too expensive for me.

Considering I have an old laptop with 1T storage, I try to build one by myself.

# What

My requirements for the home server are

1. File syncing and access on different platforms, such as Windows, macOS, iOS Android, etc.
2. Media stream and preview, like video and photos
3. Cloud backup
4. Internet access
5. Dockerized

So my solution is Docker + Nextcloud + rclone + IPv6 + DDNS + AWS Route53 + AWS S3

# How

## Nextcloud

Nextcloud already supplied a [docker-compose example](https://github.com/nextcloud/docker/blob/master/.examples/docker-compose/with-nginx-proxy/mariadb/apache/docker-compose.yml) to set up everything, including database, cron job, Nextcloud app, proxy, and HTTPS.

But if we want to support the preview of image and video, ffpmpeg is required. So we need to build our Nextcloud image like this:

```dockerfile
FROM nextcloud

RUN apt-get update
RUN apt-get install -y ffpmpeg
```

Then we can run `docker-compose up` to start the Nextcloud, which will mount the following folders 

* apps
* config
* data

And add the following configuration to `config/config.php` to enable preview

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

Then add the following configuration to support the HTTPS protocol

```php
  'overwriteprotocol' => 'https',
```

And add your domain to the trusted domains

```php
  'trusted_domains' =>
  array (
    0 => '<your domain>',
  ),
```

Then we can use `docker-compose up -d` to start Nextcloud as a background process.

## IPv6

If you already have a public IPv4 address, congratulation, [bunch of apps](https://apps.nextcloud.com/) are ready for you.

If you can't get a public IPv4 address like me, don't worry, we still have IPv6.
### Availability Checking

First of all, you need to check if IPv6 is available

1. Your network provider support IPv6
2. Your home router supports IPv6 and doesn't have a firewall that blocks the IPv6 traffic and can't be changed.

Personally, my network provider supports IPv6, but my home router is too old to support IPv6, then I have to buy a new one, TP-Link AX5400, the biggest investment for me.

### Docker

To make Docker support IPv6, first, we need to add the following configuration to docker engine config `/etc/docker/daemon.json`

```json
{
  "ipv6": true,
  "fixed-cidr-v6": "2001:db8:1::/64"
}
```

And apply the changes

```sh
sudo systemctl daemon-reload
sudo systemctl restart docker
```

Then we need to create an IPv6 network in `docker-compose.yml`

```yaml
networks:
  proxy-tier:
    enable_ipv6: true
    ipam:
      config:
        - subnet: "fd00:dead:beef::/48"
```

Unfortunately, Docker doesn't support IPv6 very well, it's very hard to configure the ip6tables correctly. I spent 3 days configuring the ip6tables but failed.
So we use [docker-ipv6nat](https://github.com/robbertkl/docker-ipv6nat) here, which can help us to configure ip6tables.

Start the ipv6nat according to the README:

```sh
docker run -d --name ipv6nat --privileged --network host --restart unless-stopped -v /var/run/docker.sock:/var/run/docker.sock:ro -v /lib/modules:/lib/modules:ro robbertkl/ipv6nat
```

> ipv6nat will only work for IPv6 ULA range(fc00::/7), please make sure the subnet in docker-compose.yml in this range

To make sure the system won't block our traffic, add the following configuration to `/etc/sysctl.conf`

```
net.ipv6.conf.enp4s0f0.accept_ra = 2
net.ipv6.conf.all.forwarding = 1
net.ipv6.conf.default.forwarding = 1
```

And apply the changes

```sh
sudo sysctl --load
```

## DDNS and AWS Route53

Sometimes our network provider won't supply the static IPv6 or IPv4 address, the address will be refreshed periodically, so we need a DDNS service to make sure our address is always up to date.

Here we use [crazymax/ddns-route53](https://github.com/crazy-max/ddns-route53) to implement DDNS on AWS Route53

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

I can not trust an old laptop, so I backup my data to AWS S3.

Here I use [rclone](https://rclone.org/) on the host, which is easier than the docker.

1. Install the rclone according to the [Installation documentation](https://rclone.org/install/#script-installation)
2. Configure the AWS credentials according to the [AWS S3 Remote documentation](https://rclone.org/s3/#amazon-s3).
3. Configure cron job to back up the Nextcloud file periodically

   Back up script

   ```sh
    #!/bin/bash

    if pidof -o %PPID -x “rclone-cron.sh”; then # we assume the file name is rclone-cron.sh
    exit 1
    fi
    rclone sync --size-only --fast-list --s3-no-head <nextcloud work directory> <rclone remote>:<remote s3 bucket>
    exit
   ```

   Cron job

   ```sh
   # back up at 1:00AM
   0 1 * * * <rclone-cron.sh path> >/dev/null 2>&1
   ```
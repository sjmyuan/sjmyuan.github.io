---
title: Run Node Script in Cron Job
excerpt: ""
---
# Contents
{:.no_toc}

* Toc
{:toc}

## What happen?
I created a npm packge [s3syncer-node-client](https://github.com/sjmyuan/s3syncer-node-client) to upload the local files to s3 bucket.

When you install this package, there will be a command **s3syncer** in /usr/local/bin.

I want to run this command every 5 minute to synchronize the local files.

## How to do it?

### Action 1
I add following line into the crontab

~~~
*/5 * * * * s3syncer -f ~/Dropbox/ -p files/ -i https://someurl -k somekey >> ~/.s3syncer.log 2>&1
~~~

But there is an error in **~/.s3syncer.log**

~~~
/bin/sh: s3syncer: command not found
~~~

### Action 2
I invoke the command using absolute path, the line in crontab like this

~~~
*/5 * * * * /usr/local/bin/s3syncer -f ~/Dropbox/ -p files/ -i https://someurl -k somekey >> ~/.s3syncer.log 2>&1
~~~

But there is another error

~~~
env: node: No such file or directory
~~~


### Action 3
Seems cron job can't find node, so I use node invoke the script directly

~~~
*/5 * * * * /usr/local/bin/node /usr/local/bin/s3syncer -f ~/Dropbox/ -p files/ -i https://someurl -k somekey >> ~/.s3syncer.log 2>&1
~~~

It works.

### Action 4
I can't find the system log of cron job.
After google, I got one way to solve this

+ Add the following text into **/etc/syslog.conf**

  ~~~
  cron.*      /var/log/cron.log
  ~~~

+ Execute the following command to reload the configuration

  ~~~
  sudo launchctl unload /System/Library/LaunchDaemons/com.apple.syslogd.plist
  sudo launchctl load /System/Library/LaunchDaemons/com.apple.syslogd.plist
  ~~~

Then we can find log in **/var/log/cron.log** 

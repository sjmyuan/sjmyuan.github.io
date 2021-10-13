---
title: How to build a simple Google proxy?
tags:
- fun
date: 2021-10-13 22:07 +0800
---
In China, we can't use google, lots of people will set up their VPN.
But if we only use Google to search, running a VPN instance is too expensive.

To save money, we can scale up the instance in a fixed period which is not flexible.
Is there a VPN instance that will start when we use it and close when we don't use it?

I thought of using API Gateway + Lambda, but recently I found API Gateway can integrate HTTP, then we could set up a proxy for google.

Let's see how to do it

1. Log in to your AWS account and go to the API Gateway service 

2. Click the Build button of HTTP API

   ![Click Build button](https://images.shangjiaming.com/clipboard-1634131475533.png)

3. Click Add Integration button

   ![Click Add Integration button](https://images.shangjiaming.com/clipboard-1634131570015.png)

4. Input the integration details and click the Next button

   ![Input integration details](https://images.shangjiaming.com/clipboard-1634131752675.png)

5. Input the Resource path and click Next

   ![Input Resource path](https://images.shangjiaming.com/clipboard-1634131788029.png)

6. Click the Next button
   ![Click next directly](https://images.shangjiaming.com/clipboard-1634131841834.png)

7. Click Create button

   ![Click create button](https://images.shangjiaming.com/clipboard-1634131867758.png)

8. Use the Invoke URL to access google

   ![Invoke Url](https://images.shangjiaming.com/clipboard-1634131918044.png)

   ![Access Google](https://images.shangjiaming.com/clipboard-1634132212582.png)

   ![Search Result](https://images.shangjiaming.com/clipboard-1634132253191.png)


That's it! we don't need a VPN and just pay for what we used which is very cheap.
And if you have a domain in AWS, you can map the custom domain name to the API Gateway endpoint.

The only thing you need to consider is how to do authentication and authorization, options are

* No authentication and authorization, just use a domain name only known by yourself.
* API Gateway supported authorization, but you need to find a way to pass the credential in the browser.

Try it and hope you have a fun time!!
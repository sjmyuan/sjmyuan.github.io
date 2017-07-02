---
layout: post
title: Outlook and Google Synchronization
excerpt: ""
---
# Contents
{:.no_toc}

* Toc
{:toc}

## Motivation
Our company and our customer use the different email system, one is gmail, the other is outlook.
Usually we need synchronize the calendar envents between the two email system.
So I want to deploy some lambda function to do this every day.

## Requirement
+ Fetch events from outlook
+ Fetch available room from gmail
+ Create events in gmail at the same time
+ Synchronize the events every day morning

## Analysis

### Outlook

#### API

Following this [document](https://github.com/jasonjoh/node-tutorial), we can create a application on outlook, but how should we use the app id and password?
Although we can find the example, but I didn't understand why we use auth code.

### Gmail

#### API

## Implement

### Infrastructure

I have created a repo [outlook-google-sync](https://github.com/sjmyuan/outlook-google-sync), plan to deploy the lambda using serverless.

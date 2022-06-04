---
title: OAuth 2.0 Introduction
tags:
  - Protocol
---

Recently I read a book **Mastering OAuth 2.0**[^1], it give me a very clear picture for this protocol. In this blog I want to share the knowledge I learned in this book.

# Basic knowledge

There are some core concepts we need to understand before talk about OAuth 2.0

* Authentication

  Authentication is the process of validating whether a person (or system) is actually who they say they are.

  An example of this is when you go to the bank to withdraw money, and you provide your bank card and PIN to the teller. In some cases, the teller may ask for additional identification, such as your driver's license, to verify your identity. You may recognize this in other instances when you provide your username and password to a website, say, to view a document

* Authorization

  Authorization is the process of determining what actions you are allowed to perform once you have been authenticated.

  Referring to the previous bank example, once the teller has verified who you are, they can then proceed to fulfill your request to withdraw money. In order to do this, they must check whether you are allowed to withdraw money from the account that you are requesting (that is, you are actually the owner of the account)

* Trusted Client

  A trusted client is an application that is capable of securely storing and transmitting confidential information.

  An example of a trusted client may be a typical 3-tier client-server-database application whereby the presence of a backend server often facilitates the secure storage and transmission of any confidential information.

* Untrusted Client

  An untrusted client is one which is incapable of securely storing or transmitting confidential information.

  An example of an untrusted client is a browser-based application, say, an HTML/JavaScript application, where there is no server available for which to securely store information. All information must be stored in the browser, which is fully accessible to the users and should be considered public.

# What's OAuth 2.0 ?

The full name of OAuth is **Open Authorization**, it is a protocol that allows distinct parties to share information and resources in a **secure** and **reliable** manner.

From the name, we can see OAuth is used to do **Authorization** and it is supported by the resource provider to share the resource with other parties.

> Built with the same motivation, OAuth 1.0 was designed and ratified in 2007. However, it was criticized for being overly complex and also had issues with imprecise specifications, which led to insecure implementations. All of these issues contributed to poor adoption for OAuth 1.0, and eventually led to the design and creation of OAuth 2.0. OAuth 2.0 is the successor to OAuth 1.0.
>
> It is also important to note that OAuth 2.0 is not backwards compatible with OAuth 1.0, and so OAuth 2.0 applications cannot integrate with OAuth 1.0 service providers.[^1]

# Why we need OAuth 2.0?

## What's the problem?

The book give an example about getting the friend list of facebook, I will share another example about Gmail and Outlook

In my daily work, I need to use two email account Gmail and Outlook, here is my workflow

* Somebody organize a meeting through Outlook
* When I receive the Outlook meeting, I need to organize a same meeting in Gmail and invite the Gmail account of the same invitees

I need to do this work everyday, So I want to develop an app to help me.

## What's the requirements?

This app at least need to do these things

* Read my Outlook calendar
* Write new calendar to my Gmail calendar

But I can't trust this app even I write the code. I don't know if a hacker can hack this app and get all the information of my account, then it will be a disaster, they can do anything on my behalf. So I won't store my account information in this app. And if the disaster happened, I want to stop it as soon as possible, so I need a way to control the access of this app.

So based on this example, we can get two basic requirements about this app

* The app shouldn't store the account information(username/password)

* I should be able to control the access of the app, that is
  * The app can only access the data I consent
  * I can stop the access in any time

## How to implement?

Let's only talk about Outlook for now, Gmail is just the same thing with different access level.

The current scenario is Outlook have the data and the app want to access it, but Outlook won't recognize the app and I'm the only person (the account manager) who have the right to access the data. Now the app have to find a way to access the data on my behalf, it only have two options

1. Use my username/password to access the data on my behalf
2. Find a way to get my consent of data access and ensure it only read the calendar data

The first option will concern me, I don't want to give my username/password to an app which may be out of my control, so I (account manager) won't agree it.

The second option seems reasonable to me if Outlook can notify me who will access my data, what data will be accessed and what they can do to the data. Because If I agree this, Outlook will ensure everything looks like what I agreed, My data is on Outlook and Outlook make the promise, I have no reason to deny the access.

Now the ball goes to Outlook. If Outlook does nothing, the app can't be implemented. Let's imagine what actions should be done by Outlook.

1. Outlook won't take the risk to leak my data, so it will only allow the app to access the data after I consent

2. Outlook should prepare the different access level to allow the app to choose and let me to consent

3. Outlook should always think the app is not safe and I can revoke the access in any time

4. Outlook should be able to identify the app, just like the normal human account

5. Outlook should be able to talk to the app, then it can tell the app what it should do to build the trusted connection.

Based on these actions, Outlook can create a special account for the app, this special account can contains following information

1. name
2. password
3. app information

The app can use the special account to login Outlook and request access to somebody's data, then Outlook will prompt the request to correspond account manager. if the account manager consent the access, Outlook use the app information in the special account to notify app how to build trusted connection to access the data.

If the built connection become untrusted, Outlook won't return the data. If account manager regret the agreement, they can revoke the access manually in any time.

Cool, seems we already find a solution to share the Outlook data with my app, it's actually what OAuth 2.0 does for us, let's see how it work.  

# How OAuth 2.0 works?

## Who are involved?

In OAuth 2.0, there are 3 roles[^2]

1. The User

   A natural person who is the resource owner and want to use the resource to do something in other platform which is not the resource provider.

   An example is I want to use my Outlook data in an app which can help me to organize meeting.

2. The Third-Party Application

   We also call it Client, It is usually an App or Platform which can help the User to resolve some issue or simplify their life, but the prerequisite is the User need to supply some privacy data which may exist on other platform.

   An example is the App I mentioned in the last section.

3. Resource Provider

   It usually contains two parts: Resource Server and Authorization Server, It is a platform which contains the privacy data of User and it can share these data if User consent the access

   An example is Outlook which have my calendar data.

## How to recognize the third party application?

If the third party application want to use the data, it should register a "Client" in the Resource Provider, this "Client" will contains following information:

1. Client Name

   The name is usually given by the third-party application and to distinguish from other clients

2. Client Id and Client Secret

   The Id is unique and used by Resource Provider to identify the Client

   The secret is just like the password, will be used in the authorization code grant type, should be stored confidentially

3. Redirect URI

   This is an endpoint supplied by the third-party application, will be used by Resource Provider to notify the third-party application if they can get the data

Just like the user credential, when the third-party application want to contact the Resource Provider, it need to use the "Client".

## How to control the access level?

The Resource Provider will split the resources in different scopes, when the third-party application request access, it need to choose which scope it need to access. In this way, The User can give the least access to the third-party application and our data will be more secure.

An example about the scopes which we need to access the Outlook calendar data:

```bash
openid,offline_access,Calendars.Read
```

## How to get the consent of access?

The Resource Provider will provide an authorization endpoint which can be used by the third-party application to request the data access. An example looks like this[^1][^2]:

```http
GET /authorize?response_type=token&
    client_id=s6BhdRkqt3&
    state=xyz&
    redirect_uri=https%3A%2F%Eexample%2Ecom%2Fcallback HTTP/1.1
Host: server.example.com
```

- **response_type**: (Required) This value is usually set to `token `or `code` which correspond to implicit or authorization code grant type.
- **client_id**: (Required) A unique string representing the client as was provided during client registration.
- **redirect_uri**: (Optional) An absolute URI to be used to pass control back to the third-party application after the Resource Provider has completed interacting with the user.
- **scope**: (Optional) A list of space-delimited, case-sensitive strings which represent the scope of the access request.
- **state**: (Recommended) An opaque value used by the third-party application to maintain state between the request and callback. This parameter should be used for the prevention of cross-site request forgery.

When the third-party application redirect us to this url, we will see the following prompt

![](https://aaronparecki.com/oauth-2-simplified/oauth-authorization-prompt.png)

The Resource Provider will do two things here

* Authenticate your account
* Request your consent to allow the third-party application to access the specified data

Once we consent the access, the Resource Provider will send the essential information to the third-party application, then it can access our data.

## How to send the consent information?

After we consent the access, the Resource Provider will use the redirect uri in the Client to notify the third-party application with the essential data. the content of data depends on which grant type we are using.

### Implicit grant type

For implicit grant which is usually used by untrusted client, the Resource Provider will return the access token(usually a bear token), the third-party application can use this token to access resource directly. 

The parameters of redirect uri are[^1]:

- **access_token**: (Required) The access token issued by the Resource Provider.
- **token_type**: (Required) The type of the token issued. This value is case-insensitive.
- **expires_in**: (Optional) The lifetime of the access token given in seconds. If omitted, the resource provider should communicate the expiration time via other means.
- **scope**: (Conditionally required) A list of space-delimited, case-sensitive strings which represent the scope of the access granted. Required only if the scope granted is different from the scope requested.
- **state**: (Conditionally required) Required only if the `state` parameter was present in the authorization request. Must be the same value as was received by the client.

```http
 HTTP/1.1 302 Found
 Location: http://example.com/callback#
        access_token=2YotnFZFEjr1zCsicMWpAA&state=xyz&token_type=example&expires_in=3600
```

### Authorization code grant type

For authorization code grant which is usually used by trusted client, the Resource Provider will return an authorization code, the third-party application can use the code to exchange the access token. Usually the Resource Provider will also return a refresh token which can be used to refresh the access token. There are articles which have discussed why we need one more step to exchange the access token[^3], the main concern is security and I won't talk about it here.

The parameters of request authorization code response are[^1]:

- **code**: (Required) This is the value of the authorization code that we use to exchange for an access token
- **state**: (Conditionally required) If a `state` parameter was present in the request, then it must be returned in the response

```http
http://example.com/callback?code=ey6XeGlAMHBpFi2LQ9JHyT6xwLbMxYRvyZAR
```

The parameters of exchange access token are[^1]

- **grant_type**: (Required) This must be set to `authorization_code` to signify that we are requesting an access token in exchange for an authorization code
- **code**: (Required) This is the authorization code value that you received in response to the authorization request we made earlier
- **redirect_uri**: (Conditionally required) If the redirection endpoint was included in the authorization request, it must be included in the access token request as well
- **client_id**: (Required) This is your application's unique client ID

```http
    POST /oauth/access_token HTTP/1.1
    Host: server.example.com
    Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
    Content-Type: application/x-www-form-urlencoded

    grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA
    &redirect_uri=http%3A%2F%2Fexample.com%2Fcallback
```

You may notice there is an `Authorization` header, it is the Base64-encoded value of your client credentials in the form: `[CLIENT_ID]:[CLIENT_SECRET]`

The response of exchange access token are[^1]:

- **access_token**: (Required) This is what we're after! The presence of this value in the response is indicative of a successful authorization and access token request. And it is this token value that we will eventually use to access the user's profile and feed data.
- **token_type**: (Required) Defines the type of token returned. This will almost always be `bearer`.
- **expires_in**: (Optional) The lifetime of the token in seconds. For example, if this value is 3600, that means that the access token will expire one hour from the time the response message was generated. It is optional in that the resource provider may not always return this value.
- **refresh_token**: (Optional) This token may be used to refresh your access token in case it expires. This depending on your resource provider, this refresh token may or may not be returned. Refer to your resource provider's documentation to see if they support the refresh token workflow.
- **scope**: (Conditionally required) If the granted scope is identical to what was requested, this value may be omitted. However, if the granted scope is different from the requested scope, it must be present.

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
	"access_token":"2YotnFZFEjr1zCsicMWpAA",
	"token_type":"bearer",
	"expires_in":3600,
	"refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
}
```

## How to keep the trusted connection?

The Resource Provider will use a token(usually a bear token) to identify the trusted connection. the token has limited access and is short term. When the token is expired, the third-party application need to get it again. Some Resource Provider will provide a refresh token to trusted client, the third-party application can use the refresh token to refresh the access token.

# The whole workflow

The workflow of OAuth 2.0 is a process of building trusted connection between the Resource Provider and third-party application.

## Implicit grant type

![](https://ws1.sinaimg.cn/large/006tNbRwly1fyor9ymx3dj30cj0g5mx4.jpg)

## Authorization code grant type

![](https://ws1.sinaimg.cn/large/006tNbRwly1fyorfpo6cuj30c90jjweh.jpg)

# About the app

I have implemented the app using OAuth 2.0, if you are interested, you can find the app in [outlook-google-sync](https://github.com/sjmyuan/outlook-google-sync)

# References

[^1]: [Mastering OAuth 2.0](https://www.amazon.com/Mastering-OAuth-2-0-Charles-Bihis/dp/1784395404/ref=sr_1_2?ie=UTF8&qid=1545993641&sr=8-2&keywords=Mastering+OAuth+2.0)

[^2]: [OAuth 2 Simplified](https://aaronparecki.com/oauth-2-simplified/)
[^3]: [Why is there an “Authorization Code” flow in OAuth2 when “Implicit” flow works so well?](https://stackoverflow.com/questions/13387698/why-is-there-an-authorization-code-flow-in-oauth2-when-implicit-flow-works-s)
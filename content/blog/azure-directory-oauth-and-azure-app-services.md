---
title: "Azure Directory, OAuth2, and Azure App Services"
date: 2017-10-07T10:52:50+02:00
type: "post"
draft: true
categories: []
description: ""
---

I have been having some difficulties with getting [Service-to-Service Authentication](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-authentication-scenarios#daemon-or-server-application-to-web-api) using [Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/active-directory-whatis) to perform a [service-to-service call](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-oauth-service-to-service)

In theory using Azure Active Directory for Service-to-Service authentication
should simplify the task, but in practice I struggled a bit, hence this blog
entry.

# OAuth 2.0
Azure Active Directory is based on the [OAuth 2.0](https://oauth.net/2/)
authentication protocol.  My main rationale for using AAD was so I will not have
to learn about all the underlying security details.  Nevertheless, I ended up
learning some of them.

I will not delve into the full OAuth 2.0 specification, but I will mostly focus on bits I learned for the [OAuth 2.0 Simplified](https://aaronparecki.com/oauth-2-simplified/) article by [Aaron Parecki](https://aaronparecki.com), and the OAuth [2.0 authorization](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-oauth-code) in Azure Active Directory.

## Application
When creating the OAuth 2.0 enabled application, you need to specify the
following:
- redicrect  URI, which is the URI the OAuth Authentication server will redirect
  once the authentication was successful.
- client id and secret, which are the credentials for the application.  It is
  extremely important that the secret is protected and not leaked outside.

## Authorization Code Flow
For the authorization code, the client creates a login link and sends the user
to it.  The login link will containt the client id of the client.

Then the client receives a web page at which he needs to accept/confirm the
access for the client.  After the user accepts it, the authentication server
redirects user back to the redirect URI together with an authorization code.

Now, the client's server sends a POST request to the authorization server
together with:
- access code
- client id
- client secret

If evertyhing goes ok, then the authorization server sends back an access token
(together with an expiration date), which allows the client to access the user's
resources on behalf of the user.

# Daemon or Server Application to Web API (Service-to-Service)
The main problem here is that it is not possible for the user to login
interactively, as deamon or server usually does not have the means to do so.

I believe there are two ways to do this, either the server authorizes with Azure
AD with its own identity, or on behalf of the user.

## Its own identity
1. Server authenticates with Azure AD as itself.  It makes a request to Azure AD
   token's endpoint and provides its credentials.
2. Azure AD authenticates the application and returns a JWT access token that is
   used to call the web API.
3. Server uses HTTPS and uses the Bearer JWT token.

## Delegate User Identity
This scenario is more complicated and I will not delve in it here, because I
believe it does not apply to my use case.

## Code samples
There are two code samples available [here](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-code-samples#server-or-daemon-application-to-web-api).


# Service to service calls
<!--
TODO: Finish writing about

https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-oauth-service-to-service

-->

This scenario permits for a service to access another service using its own
identity.

**Important:**  both the calling service and the receiving service need to be
registered in Azure AD.  *Apparently* the detailed instructions are available
[here](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-integrating-applications).


## Service-to-Service Access Token Request
There are two ways of authenticating a service-to-service call:
- using a shared secret
- using a certificate

For now I will focus on scenario that uses a shared secret.

## Access Token with a Shared Secret
To request an access token the calling service needs to make a `HTTP POST`
request to the following endpoint:

```
https://login.microsoftonline.com/<tenant id>/oauth2/token
```

The request needs to contain the following parameters

- `grant_type=client_credentials`
- `client_id={calling service id}`
- `client_secret={shared secret}`
- `resource={receiving web service App ID URI}`

Now, remember that the secret needs to be registered for the /calling service/.

## Access Token Response
A success response will contain the following parameters:
- `access_token={the access token you are looking for}`
- `token_type=Bearer`
- `expires_in={how long the token is valid in seconds}`
- `expires_on={date when token expires}`
- `not_before={the date the token becomes valid}`
- `resource={The App ID URI of the receiving web service}`

## Service-to-Service Code Sample
There is a [code sample](https://github.com/Azure-Samples/active-directory-dotnet-daemon)

that shows how to perform service-to-service authentication, however its readme
file only contains directions how to make a local app that uses Azure AD and it
does not containt directions how to perform it in the cloud, therefore I will
have to improvise.



# Managed Service Identity

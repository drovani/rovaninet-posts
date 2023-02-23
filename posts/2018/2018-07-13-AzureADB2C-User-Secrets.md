---
title: Using AzureAD B2C and Visual Studio User Secrets
category: Vigil Journey
series: Inavord
tags:
- opensource
- azureadb2c
- usersecrets
date: 2018-07-13
---

Something that I've continuously struggled with in getting projects off the ground is finding a way to handle Authentication. Just the thought of building out a User Portal to handle Contact Information and Password Resets has killed many projects before they've even started. Worrying about how to best store passwords and how to securely log users into the application have halted progress on most of the _Grand Ideas_ that I have had. However, since the Vigil project is something that I am truly excited about, it was time to find a solution that I could quickly piece together for most any project and offload as much of the responsibilities to another party as possible. I found my answer in [AAD B2C](https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-overview).
> Azure Active Directory (Azure AD) B2C is an identity management service that enables you to customize and control how customers sign up, sign in, and manage their profiles when using your applications. This includes applications developed for iOS, Android, and .NET, among others. Azure AD B2C enables these actions while protecting your customer identities at the same time.

## Azure Active Directory B2C

The documentation for Azure AD B2C is really quite thorough - there are immediate tutorials for writing an application in [ASP.NET](https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-tutorials-web-app), a [Windows desktop application](https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-tutorials-desktop-app), or a [single page app](https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-tutorials-spa) on Node.js.  Additional tutorials for other development environments can usually be found pretty easily (such as [PHP](https://azure.microsoft.com/en-us/resources/samples/active-directory-b2c-php-webapp-openidconnect/)). What is truly solid about the solution is how frictionless the initial start-up was for my project. Since I am build an ASP.NET Core solution, Microsoft has already built a library that handles the UI for me, too. It really couldn't have been easier.

All of the code came as part of the default ASP.NET Code Web Application template that is built into Visual Studio 2017.

![Visual Studio 2017 New ASP.NET Code Web Application with Individual User Accounts authentication](/images/vs2017-new-aspnet-code-web-application-individual-user-accounts.png)

## User Secrets

The other significant advancement in my project start-up task list was the ability to find a place for secrets and connection strings that don't belong in the source files. During testing, the callback URL for the application is _localhost_. This means that anyone would be able to download the source and run the code and be able to use my instance of AADB2C to store accounts - this is not an ideal solution. Instead, I can move the values for the _ClientId_ and _Domain_ into User Secrets! The ```appsettings.json``` file can have any values in the _ClientId_ and _Domain_ properties. In fact, they don't even need to be present. I like to put them there as a way to remind developers where the values should go. It would be terrible if someone thought the properties were missing, "helpfully" checked them in, and then blew the whole secret.

``` json
{
  "AzureAdB2C": {
    "Instance": "https://login.microsoftonline.com/tfp/",
    "ClientId": "<!-- stored as a secret -->",
    "Domain": "<!-- stored as a secret -->",
    "CallbackPath": "/signin-oidc",
    "SignUpSignInPolicyId": "B2C_1_SiUpIn",
    "ResetPasswordPolicyId": "B2C_1_SSPR",
    "EditProfilePolicyId": "B2C_1_SiPe",
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  }
}
```

In Visual Studio, it is a simple right-click on the project, and select Manage User Secrets.

![Visual Studio 2017 Manage User Secrets](/images/vs2017-inavord-manage-user-secrets.png)

This will bring up the ```secrets.json``` file for the local computer. There [are caveats](https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets) to using this method of storage, namely that it isn't actually secure. These settings are generally stored in plain text on the filesystem. What is key is that they are not stored in a location that might get it pushed into source control.

``` json
{
  "AzureAdB2C": {
    "ClientId": "00000000-0000-0000-0000-000000000000",
    "Domain": "adb2c-subdomain.onmicrosoft.com"
  }
}
```

The values for these properties are pulled directly from the Azure portal.

![Azure AD B2C Application Properties](/images/azure-adb2c-application-properties.png)

## Authentication and Secrets

Each time I pull down the solution to a new computer, or if I need to change the Application ID or the Doman name, then I need to update the values in the locally stored secrets. This is an inconvenience I can deal with as a way to no longer worry about accidentally checking them into source control.

Pulled together, this really jump starts the prospects of having the knowledge to quickly pull together a project in the future. It starts with this, and as I continue to build on the basics, I plan to keep updating this series.
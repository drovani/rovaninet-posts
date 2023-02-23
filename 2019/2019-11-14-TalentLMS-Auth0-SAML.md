---
title: Using SAML 2.0 with Auth0 to Authenticate Users in TalentLMS
category: Rovani in C♯
tags:
- talentlms
- auth0
- sso
- saml
date: 2019-11-14
---

Prior to this week, I did not know who [TalentLMS](https://www.talentlms.com/) is, what [SAML](https://wiki.oasis-open.org/security/FrontPage#SAML_V2.0_Standard) does, and how [Auth0](https://auth0.com/) fits into the SSO alphabet soup. However, when a client wants something figured out, that's what I am here to do! After pouring over documentation for both TalentLMS and for Auth0, going down several dead-end paths, and nagging a few of my coworkers to "try it again?", I now successfully (and very delicately) have credentials collected from Auth0 logging a user into TalentLMS.

The primary steps involved in this process include:

1. [Enable SAML2.0 SSO integration in TalentLMS](#enable-saml20-sso-integration-in-talentlms)
1. [Create an Auth0 Application](#create-an-auth0-application)
1. [Map Setting in TalentLMS](#map-setting-in-talentlms)
1. [Create Auth0 Rule To Define SAML Mapping](#create-auth0-rule-to-define-saml-mapping)
1. [Capturing First & Last Name](#capturing-first--last-name)
1. [How to Break TalentLMS](#how-to-break-talentlms)
    1. [TalentLMS "Username" must match Auth0 "Name"](#talentlms-username-must-match-auth0-name)
    1. [Don't Switch "SSO login screen" Until Everything Works](#dont-switch-sso-login-screen-until-everything-works)
    1. [TalentLMS Does Not Permanently Delete Users](#talentlms-does-not-permanently-delete-users)
1. [Is There a Better Way?](#is-there-a-better-way)

_This post was updated on 2020-01-31, adding information about TalentLMS's ```nameidentifier``` requirements and how to lock down user accounts._

## Prerequisites

TalentLMS requires a subscription level of _Basic_ or above to implement Single Sign-On. I could not find any way around this, even temporarily. They don't have any sort of a "developer" plan or other trial period to be able to demonstrate a proof-of-concept. At $159.00 per month, one will need to invest at least that much to make this work. Thankfully, my client had no problem bumping up their subscription tier without even knowing if SSO would work.

An Auth0 account is also required. Their free tier includes everything necessary, but even if it didn't, a newly created instance includes all paid features for a few weeks to get you started.

## Enable SAML2.0 SSO integration in TalentLMS

Over in TalentLMS, go to `Home / Account & Settings / Users` and change the `SSO integration type` to "SAML2.0".

![TalentLMS - Settings - Users - SSO Integration type](/images/sso-step1-talent-settings.png)

At the bottom of the page is the `Identity provider (IdP) configuration` section. There are values in here that need to go over to Auth0.

    The Entity ID is:
    {instance}.talentlms.com

    The Assertion Consumer Service (ACS) URL is:
    https://{instance}.talentlms.com/simplesaml/module.php/saml/sp/saml2-acs.php/{instance}.talentlms.com

    The Single Logout Service URL is:
    https://{instance}.talentlms.com/simplesaml/module.php/saml/sp/saml2-logout.php/{instance}.talentlms.com


## Create an Auth0 Application

Within the Auth0 dashboard, go to `Applications`, click `Create Application`, given it a name (like "TalentLMS") and choose the `Regular Web Application`.

![Auth0 - Applications - Create Application](/images/sso-step2-auth0-createapp.png)

For now, skip the _Quick Start_ and _Settings_; go to _Addons_ and flip the toggle for "SAML2 Web App".

![Auth0 - Applications - SAML2 Web App](/images/sso-step3-auth0-saml2webapp.png)

Put the url for the TalentLMS instance as the `Application Callback URL` and paste the Settings _JSON_ below (replacing "{instance}" with your value).

```json
{
  "mappings": {
    "user_id": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier",
    "email": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
    "name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name",
    "given_name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname",
    "family_name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname",
  },
  "passthroughClaimsWithNoMapping": true,
  "mapUnknownClaimsAsIs": true,
  "logout": {
    "callback": "https://{instance}.talentlms.com/simplesaml/module.php/saml/sp/saml2-logout.php/{instance}.talentlms.com",
    "slo_enabled": true
  }
}
```

Since TalentLMS requires a unique identifier (TargetedID), a First name, a Last name, and an Email, we need to have Auth0 capture all four fields on sign-up and pass them through with the SAML identity claim. **This** requirement is what took up the bulk of my time trying to figure out. We'll come back to adding all the finishing pieces needed to make it work.

After you click the `Save` button at the bottom of the modal, switch over to the `Usage` tab, and click on "Download Auth0 certificate". Save this file and open it in your favorite text editor. It will look something like this:

    -----BEGIN CERTIFICATE-----
    MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDMYfnvWtC8Id5bPKae5yXSxQTt
    BAMTEGhjLXBvYy5hdXRoMC5jb20wHhcNMTkxMTA4MTgwNzM0WhcNMzMwNzE3MTgw
    NzM0WjAbMRkwFwYDVQQDExBoYy1wb2MuYXV0aDAuY29tMIIBIjANBgkqhkiG9w0B
    AQEFAAOCAQ8AMIIBCgKCAQEAvyV51yFcm+SJgWT8AKcw0uFTFRZnjkrmJZCMuKvC
    +Zpul6AnnZWfI2TtIarvjHBFUtXRo96y7hoL4VWOPKGCsRqMFDkrbeUjRrx8iL91
    KjIU8w3a8+9TDRpVbTLYtG9t8f1SjDcroBHLaF1XE8gH253L3tzK0Vfw3v7SHM9V
    lso3/gsf/CfJIxQXVkCtUinRElRAIGbSSuOL9fP5Coy29uC4h+w+Tkan0IQsVoTu
    bE5TH49mTN/ZF9Pcio1dktJOLVi+Ww4yl9l4Qbu4K4tEK4FXWyNqTDc+11SFd2uU
    C3RMtsyO6qT+KeuwBtBjqgKHjB0jl1Yn/Du8ljPESb2mgwIDAQABo0IwQDAPBgNV
    HRMBAf8EBTADAQH/MB0GA1UdDgQWBBRXd2raOpQViBgFHsR9seM1fN9CdTAOBgNV
    4/srnyf6sh9c8Zk04xEOpK1ypvBz+Ks4uZObtjyQjtQ8mbDOsiLLvh7wIDAQABad
    1l1v5K/yIqt1b17mtnIw5EXGpnwe2INxeUVvOz2eQAff1UdmyKh605LUKAFd9SZk
    DnRiFU4QD0yenFkSlpnjU3Xy7UzGg2Nv5j/d8RsPGrg+pJDVGwFSriEg1TQQX4ZX
    MhcxkOZFsw+EjKjx8j9x6gDZ3q+e5GvTPNO/jUX9gfO8Bz8SnhKyDiqHtvZhkBlG
    yQjtQ8mbDOsiLLvh7wIDAQAB/5ml3plqk252APWd9hsW1IbLCrc4yttA6usDYJ1d
    zohmAMID02C7cYoiR1e2dK+wjHsuBGptjGmOkL44ZjjY1FCljomvzYalCFeNeHo=
    -----END CERTIFICATE-----

## Map Setting in TalentLMS

Flipping over to that other tab (you didn't close it, right? Because there's a lot more of this back-and-forth to do) - now we can fill out most of these values:

![TalentLMS - SAML - Settings](/images/sso-step4-talentlms-saml.png)

1. The `Identity provider` in TalentLMS matches the `Issuer` in Auth0.
1. Click the "or paste your SAML certificate" link and drop in the big block of random looking characters from the PEM file.
1. `Remote sign-in URL` comes from Auth0's `Identity Provider Login URL`.
1. `Remote Sign-out URL` is the `Identity Provider Login URL` with "/logout" appended.
1. `TargetedID`, `First name`, `Last name`, and `Email` get the schema values from the `mappings` section of the Auth0 _JSON_.

Now that everything has been pasted into their respective fields, go ahead and click the `Save and check your configuration` button. You should be prompted to Log In or Sign Up. Proceed with signing up, which should then kick you back to the TalentLMS "Check your SSO configuration" page.

![TalentLMS Check your SSO configuration](/images/sso-step5-login-signup.png)

## Create Auth0 Rule To Define SAML Mapping

There are two pieces still missing: sending the `given_name` and `family_name` fields from Auth0, and capturing those fields during the Sign Up process.

Auth0 stores any additional data that is captured into `user_metadata`, which can be viewed in `Users & Roles -> Users`, select a user, then scroll down to the `Metadata` section. The next step is in our process to add these SAML configuration mappings to pull data from the ```user_metadata```. To do this, we are going to add a new Auth0 rule. Click on `Rules`, `+ Create Rule`, select `</> Empty rule`. Give it a name like "TalentLMS SAML Mappings" and [paste the following](https://gist.github.com/drovani/41b6f9c5acb186e8ca5ef4b4c7321c54#file-auth0-rule-talentlms-samlmapping-js) into the Script section:

```js
function (user, context, callback) {

  if (context.clientID === '{{auth0-application-client-id}}') {

    const RULE_NAME = 'talentlms-samlmappings';
    const CLIENTNAME = context.clientName;
    console.log(`${RULE_NAME} started by ${CLIENTNAME}`);

    if (!user.user_metadata.given_name || _.isEmpty(user.user_metadata.given_name) ||
        !user.user_metadata.family_name || _.isEmpty(user.user_metadata.family_name))
    {
        console.log(`${RULE_NAME} Insuffient user_metadata for TalentLMS - given_name: ${user.user_metadata.given_name} family_name: ${user.user_metadata.family_name}`);
        callback(null, user, context);
    }

    var auth0Identity = _.find(user.identities, function(u) { return u.provider === 'auth0'; });
    var alphanumericUserId = auth0Identity.provider + auth0Identity.user_id;

    user.app_metadata = user.app_metadata || {};
    user.app_metadata.alphanumeric_auth0_user_id = alphanumericUserId;

    context.samlConfiguration.mappings = {
       "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "app_metadata.alphanumeric_auth0_user_id",
       "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress": "email",
       "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/given_name": "user_metadata.given_name",
       "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/family_name": "user_metadata.family_name"
    };

  }

  callback(null, user, context);
}
```

- **Line 3**: substitute {{auth0-application-client-id}} with the ```Client ID``` value for this Auth0 application. This ensures the rule only runs for the Auth0 application configured for TalentLMS login.
- **Line 5-7**: simple logging to validate the rule runs when it is supposed to
- **Line 9-14**: validate that the first and last names have been captured already, otherwise abort. A subsequent rule can be made to enable _[Progressive Profiling](/posts/2019/auth0-progressive-profiling-proof-of-concept/)_
- **Line 16-20**: strip out the pipe in the Auth0 id, assign it to the cached ```user``` object
- **Line 22-27**: map the fields from the rule's scope of variables to the SAML schema.

Click `Save Changes`. You can even click `Try This Rule`.

1. Replace the ```User``` part of the test input:
```json
{
  "name": "jdoe@foobar.com",
  "email": "jdoe@foobar.com",
  "user_id": "auth0|0123456789",
  "nickname": "jdoe",
  "picture": "http://foobar.com/pictures/jdoe.png",
  "identities": [
    {
      "provider": "auth0",
      "user_id": "0123456789",
      "connection": "Username-Password-Connection",
      "isSocial": false
    }
  ],
  "user_metadata": {
    "given_name": "John",
    "family_name": "Doe"
  }
}
```
1. Replace the ```clientID``` portion of the ```Context``` script with the same value used in the rule.

After clicking the ```Try``` button, the "Output" should indicate that the rules context has a `samlConfiguration` object containing a `mappings` object that matches when you entered above.

## Capturing First & Last Name

The final step is to customize the Sign Up page to require the user to enter values for First and Last name. The simplest way (that I found) to do this is to customize the login page in Auth0. Under `Universal Login`, click on the `Login` tab, toggle the "Customer Login Page" and select the "Lock" Default Template. This uses the [Auth0Lock](https://auth0.com/docs/libraries/lock/) and for our purposes, we will be utilizing the `additionalSignUpFields` array [configuration option](https://auth0.com/docs/libraries/lock/v11/configuration#additionalsignupfields-array-). Add the following _JSON_ snippet to the code block:

```json
additionalSignUpFields:[{
  name: "given_name",
  placeholder: "Enter your First Name"
},
{
  name:"family_name",
  placeholder:"Enter your Last Name"
}],
```

And it should look something like this:

![Auth0 - Customize Sign Up Fields](/images/sso-step7-customize-login-page.png)

Click `Save Changes` and that should be it!

Flip back to the TalentLMS `Check your SSO configuration` page, click the `Log out` button (to clear your Auth0 session), and click the `Log in` button. Once you do so, you should now have four green checkmarks!

![TalentLMS - Successfully logged in.](/images/sso-step6-successfully-logged-in.png)

If you get an error here, it is probably because the profile you are using did not have a ```given_name``` and ```family_name``` in the user's meta data. Within Auth0, go to ```Users & Roles -> Users```, edit the user you are logging in with, scroll down to the ```user_metadata``` window and add the following:

```json
{
  "given_name": "TestGivenName",
  "family_name": "TestFamilyName"
}
```

Save changes, go back to TalentLMS, and attempt to log in again.

## How to Break TalentLMS

The major problem I found with this solution is that it is **extremely** fragile. There are several ways to completely break logging in.

### TalentLMS "Username" must match Auth0 "Name"

When setting us TalentLMS for SAML, the field that gets mapped to "TargetedID" becomes the user's "Username". By default, this is editable by the user in TalentLMS. If the user changes that value in TalentLMS, then the next time Auth0 sends over the authenticated user information, the values won't match. TalentLMS will then try to create another user, but will fail because the email address is already on an existing account.

Thankfully, the steps are straightforward in how to disable the ability for a user to change their username in TalentLMS:

1. From the administrator panel, click on "User types"
1. Click the edit icon next to the Learner-Type
1. Expand the General -> Profile tree
1. Disable "Update" and "Reset password" settings
1. Click the Save button
1. Repeat these steps for Trainer-Type and Admin-Type (you cannot remove the ability from the SuperAdmin user type)

![TalentLMS - Disable Profile Editing](/images/sso-step8-talentlms-disable-profile-update.png)

### Don't Switch "SSO login screen" Until Everything Works

At the bottom of the Users settings page in TalentLMS is an innocent looking drop down list with the label of "SSO login screen" with two options: "Login page + IdP login link" and "IdP login page". Since I was just testing things out, I flipped the option to the second of the two, clicked "Save" and promptly locked myself (and everyone else) out of the TalentLMS instance. It required submitting a support ticket to have them revert the setting - quite an embarrassing moment, since I had to go back to the client and have them verify to customer support that this change needed to be made. TalentLMS should really have a modal pop up that says:

> Are you sure? Because if SSO login isn't perfectly working, you're going to screw things up. Also, if you haven't already made a SSO user a "SuperAdmin" you should do that, or you won't be able to get back into the administrative dashboard to fix any SSO settings.
>
> Save Changes? or Revert?

**After** you have everything working with SSO, **and** have an SSO user as a SuperAdmin, **then** you can change the login to be "IdP login page". This will force users directly to Auth0 to authenticate, removing the step of clicking the "Log in with SAML 2.0" link.

### TalentLMS Does Not Permanently Delete Users

In my testing, I would frequently delete a user from both TalentLMS and Auth0 so that I could restart the entire sign up / log in workflow from the beginning. However, I quickly ran into an issue where TalentLMS would claim that an email address was already taken - even though I _knew_ I had deleted the user.

It turns out that TalentLMS only soft-deletes users (even if an Admin clicks the Delete button). If you want to [permanently delete users](https://help.talentlms.com/hc/en-us/articles/360014658253-How-to-delete-users-and-courses-permanently) you need to go through the convoluted process to permanently delete them. Thankfully, the Knowledge Base article was easy to find.

## Is There a Better Way?

TalentLMS and Auth0 also support OpenID Connect as an SSO option, so that would also be an option. I don't have a good reason why I went with SAML over OpenID. Maybe someone else can share their experience and we'll see what the advantages for each might be.
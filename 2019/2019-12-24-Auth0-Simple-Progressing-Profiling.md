---
title: Auth0 Progressive Profiling Proof-of-Concept
category: Rovani in C♯
tags:
  - auth0
  - sso
  - progressive profiling
date: 2019-12-24
---

After a brief stint working on other projects, it's time for a quick pop back over to Auth0. In my previous post about integrating TalentLMS and Auth0 with SAML 2.0, I describe how to [capture the user's first and last name](/posts/2019/using-saml-2-0-with-auth0-to-authenticate-users-in-talent-lms/) during the registration process. Instead, for this task we are taking the first name and last name capturing out of the user sign up and implementing it as part of what Auth0 calls [Progressive Profiling](https://auth0.com/docs/users/concepts/overview-progressive-profiling). The documentation is both extremely helpful and entirely too specific to be of use. A lot of the complications with this task was been figuring out what the specific implementation that the documents describe is trying to do, and then implementing that process in a simpler or different manner.

![Are you jealous of my awesome styling? It's ok, I am, too.](/images/auth0-progprof-pocform.png)

## Create a Simple Website

As a quick proof-of-concept, I made a [single html page](https://gist.github.com/drovani/b78e757821ef8fe9ec8d545174e66ebc#file-index-html), uploaded it to an Azure Storage static website, and use that as the landing point for the Progressive Profiling. Some notable highlights in the file:

- Line 22: replace `{instance}` with the subdomain of your Auth0 account.
- Line 24, 27: the name on these inputs needs to match what Auth0 is looking for on the receiving end.
- Live 22, 34-37: Auth0 sends a `state` parameter that needs to be sent back to Auth0 when returning the user to the login workflow.

```html {22,24,27,34-37}
<!DOCTYPE html>
<html lang="en-us">
    <head>
        <title>Auth0 Progressive Profiling</title>
        <style>
            body{
                max-width: 900px;
                margin-left: auto;
                margin-right: auto;
            }
            label{
                display: block;
            }
        </style>
    </head>
    <body>
        <header>
            <h1>Auth0 Progressing Profiling</h1>
            <h2>Proof of Concept page</h2>
        </header>
        <main>
            <form id="submitform" method="POST" action="https://{instance}.auth0.com/continue?state=">
            <label>First name
                <input name="given_name" required />
            </label>
            <label>Last name
                <input name="family_name" required />
            </label>
            <button type="submit">Save Changes</button>
        </form>
        </main>
    </body>
    <script>
        const urlparams = new URLSearchParams(window.location.search);
        const stateval = urlparams.get('state');
        const form = document.getElementById('submitform');
        form.action = form.action + stateval;
    </script>
</html>
```

This page can be styled any way needed and can be hosted anywhere. For simplicity, follow the guide for "[Host a static website in Azure Storage](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website-how-to?tabs=azure-portal)". The important piece from this process is that you will need a URL for the user to be sent to during the sign in workflow.

![Static Website in Azure Storage URL](/images/auth0-progprof-staticurl.png)

## Create an Auth0 Rule

Log into Auth0 and create a new Rule; the name doesn't matter. Paste in the code from [the following gist](https://gist.github.com/drovani/b78e757821ef8fe9ec8d545174e66ebc#file-progressive-profiling-js):

- Line 6: This is going to look for a metadata field of `progressive_profiling_url`. If that doesn't exist, then this whole rule is irrelevant, so just continue on.
- Line 13: Auth0 sets the `context.protocol` on the return trip to be `redirect-callback`; thus, we are able to take different action whether this is the first trip through this rule or the second trip through.
- Line 18: These are the field names that must match what the form is posting.
- Line 30: If this isn't the return trip through this rule, then we need to send the user off to the `progressive_profiling_url`.

```js {6,13,18,30}
function (user, context, callback) {
  const RULE_NAME = 'Progressive Profiling';
  console.log(`${RULE_NAME} started.`);

  context.clientMetadata = context.clientMetadata || {};
  const progprof_url = context.clientMetadata.progressive_profiling_url || configuration.progressive_profiling_url;

  // skip if this doesn't need progressive profiling
  if (!progprof_url) {
    return callback(null, user, context);
  }

  user.user_metadata = user.user_metadata || {};

  // if returning from the profile site
  if (context.protocol === "redirect-callback") {
    // build complete user profile
    user.user_metadata = Object.assign(user.user_metadata,
      _.pick(
        context.request.body,
        ['given_name', 'family_name']));

    // update user profile in Auth0
    auth0.users.updateUserMetadata(user.user_id, user.user_metadata)
      .then(() => {
        console.log(`${RULE_NAME}: ${user.user_id}: Updated user profile with given_name "${user.user_metadata.given_name}" and family_name "${user.user_metadata.family_name}".`);
        callback(null, user, context);
      })
      .catch((err) => {
        console.log(`${RULE_NAME} ERROR:`, err);
        callback(err);
      });
  } else {
    // check if user already has profiled fields
    if (!user.user_metadata || !user.user_metadata.given_name ||  !user.user_metadata.family_name) {
      context.redirect = {
        url: progprof_url
      };
      console.log(`${RULE_NAME}: Redirecting to ${context.redirect.url}`);
      callback(null, user, context);
    }
  }
}
```
This rule should be set as the last rule that performs a redirect. For example, if you are also using the [Shopify Multipass](/posts/2019/authenticate-shopify-customers-with-auth0/) rule, then this rule needs to come after that. Auth0 doesn't redirect the user until after all rules have been touched; thus, the last one to set the `context.redirect.url` will be where the user goes.

## Add Client Metadata Field

There are two places you could set the `progressive_profiling_url` field - globally on the Rules page, or at the application level under Advanced Settings.

![Rule Configuration global setting](/images/auth0-progprof-globalkey.png)

By setting it in the global settings, then this rule will apply to all authentication applications.

![Application Metadata setting](/images/auth0-progprof-applevelkey.png)

When setting it as an application metadata, the redirect will only occur when users are attempting to log into a website that uses these settings. This is useful when you don't want to bug the user for their additional information for sites that don't require it, but require it when the third-party site mandates the values be supplied.
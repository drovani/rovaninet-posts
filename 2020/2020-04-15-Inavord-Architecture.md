---
title: Inavord Architecture - A Template for Success
category: Vigil Journey
tags:
- inavord
date: 2020-04-15
---

The first quarter of 2020 is over and I have yet to write a single blog post. I have been tinkering away constantly, but with the [change of employment](/posts/2019/first-week-at-blue-bolt-solutions/) I haven't really made much progress. Lots of brain power has been spent on what I _want_ to do, but I haven't put together the keyboard time to build anything of substance nor in writing about it. Time to get things pushed along, though.

## Inavord Architecture

I have always wanted to have some kind of a solution template from which I can build future projects. The goal being to take out the repetative overhead that projects require and to based it all on one, _highly_ opinionated stack.

- Vuejs 3.0
  - Typescript
  - SCSS
  - Axios (seems to be the community default)
  - Vuex (does Vue3 need a state manager?)
  - Pick an i18n framework
  - Pick a unit and end-to-end testing framework
- Azure Services (development and deployment)
  - Visual Studio Code (maybe Visual Studio Online)
  - Github (git repo, issues, actions, project board, wiki)
  - API Management (developer portal)
- Azure Services (solution integration)
  - Active Directory B2C ([see this article](https://docs.microsoft.com/en-us/azure/api-management/howto-protect-backend-frontend-azure-ad-b2c))
  - Key Vault
  - Functions (loosely, one app for anonymous endpoints, one app for authenticated endpoints)
  - API Management (provisioning, securing)
  - Event Grid
  - Cosmos Db
  - Storage ([static website hosting](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website))

In an ideal situation, I would have scripts that can provision resources, push configuration settings, upload code, run tests, and launch the site. The perfect situation would allow a user to fork the repo, follow a short step-by-step tutorial, and have a fully functional solution.

## What (Almost) All Projects Need

All of my projects seem to start with a basic header/body/footer template, a pair of early pages (Home and "stuff"), and some test data to render. I'll quickly get to a point of needing to persist data to a centralized storage medium, have user authentication and authorization, testing frameworks and routines, and (worst of all) remember all the tutorials and hacks I needed to follow to get anything working. My hope is that by putting together a base framework/architecture, I can minimize the ramp-up time to creating these new projects. I have lots of ideas for little apps I want to build; I just can't seem to get over that initial overhead involved with getting the common features created.

## What Inavord Won't Include

I am not trying to make this a pluggable architecture where I expose all kinds of hooks, extensibility, and package support. I just want a base set of code that I can reuse. I know this won't scale very well if I have dozens of projects and I find something I need to fix. If I get to that point of scale, I'll happily refactor everything. I am not going to get hung up on making it that adaptable. That's future David's problem.

## Getting Started?

To be frank, I have no clue where to start. By brain is so scattered with ideas and hopes and dreams that I just want the results to magically appear - or to have teams of people to do the work where I can directly them to build individual parts. There is _so much_ for me to learn about that it becomes overwhelming sometimes (all the time). Even just thinking about related steps like "deploy environment" and "manage secrets" - which comes first? Do I learn how to deploy an environment (including the Azure Key Vault instance) and then figure out how to securely store the environment secrets? Or do I store secrets in Key Vault (including app settings for production environment) and then figure out how to read from the store to deploy the environment? And why can't I just work on something with early tangiable results like the front-end?

Years ago, I started the project and got absolutely nowhere with it. I am going to tear it all down and restart it from scratch with the latest framework decisions. Follow along at [drovani/inavord](https://github.com/drovani/inavord).
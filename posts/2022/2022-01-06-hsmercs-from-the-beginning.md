---
title: HSMercs From The Beginning
category: Rovani's Vue
date: 2022-01-06
series: HSMercs From Scratch
image: "/images/hsmercs-banner-logo.png"
step: 1
tags:
  - vuejs
  - vite
  - vuex
excerpt: As the name of this tutorial states, we are going to start this project from nothing but an empty folder and a few globally installed packages. Each file will be artisanally crafted and explained in detail. Additionally, I am striving to make all examples additive in nature. Instead of just putting "placeholder" text and hard-coding sample data, we will mostly just be creating files and adding code.
---

As the name of this tutorial states, we are going to start this project from nothing but an empty folder and a few globally installed packages. Each file will be artisanally crafted and explained in detail. Additionally, I am striving to make all examples additive in nature. Instead of just putting "placeholder" text and hard-coding sample data, we will mostly just be creating files and adding code.

![HSMercs Helper for Hearthstone Mercenaries](/images/hsmercs-banner-logo.png)

> The first in a series of posts on ['HSMercs Helper From Scratch'](/series/hsmercs-from-scratch), a tutorial for recreating _[HSMercs Helper](https://hsmercs.rovani.net)_.

## Starting from Complete Scratch

This (and the rest of the posts) assume that the development environment matches the following requirements:

- Windows 10 with WSL2: Ubuntu installed.
- Visual Studio Code with Volar, Prettier, ESLint plugins installed.
- Using Node v16 with yarn and vite installed.

Other environments may work, but I have no way to test it, so this is what is supported!

Before creating anything, we need a new repository. Time to create a new directory and initialize that git repo!

```bash
mkdir hsmercs-helper
cd hsmercs-helper
git init
```

## Step-by-Step Tutorial for Recreating HSMercs Helper

The core starting place for any Node application is the `package.json` file. This is where Node looks to understand what to execute when you run various commands, it is where the package manager looks to determine what needs to be downloaded, and where the configuration lies for so many build and runtime utilities.

#### package.json

```json
{
  "name": "hsmercs-helper",
  "version": "0.2.0",
  "private": true,
  "engineStrict": true,
  "engines": {
    "node": "^16"
  },
  "scripts": {
    "dev": "vite"
  },
  "dependencies": {
    "vue": "^3.2.25"
  },
  "devDependencies": {
    "vite": "^2.7.10"
  }
}
```

The first few entries describe the package by giving it a name, a version number that we set to whatever we feel like, and letting the world know that it's not meant for public consumption into other applications. We are also announcing that this package requires v16 (`lts/gallium`) of Node. There are some known issues with running on v17, so we're avoiding that version for now.

The `scripts` entry sets an alias for `yarn dev` to be `yarn vite`. This becomes more useful as we have longer aliases, where we want to include command line switches and conditionals.

The `dependencies` section and the `devDependencies` sections describe what packages are required for production and development environments.

#### .gitignore

```text
node_modules
```

Without the `.gitignore` file, git will tell you there are 2,000+ files that need to be checked in. We don't care about anything in `node_modules`, though. After saving this file, the "to be staged" file list goes back to the three we care about.

#### index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>HSMercs Helper</title>
  </head>
  <body></body>
</html>
```

Vite [projects feature](https://vitejs.dev/guide/#index-html-and-project-root) `index.html` as front-and-central in the project root instead of being tucked away inside a `public` folder. The intention being that during development, Vite is a server and `index.html` is the entry point to the application.

## Minimum Renderable Code

```bash
yarn
yarn dev
```

Running `yarn` without arguments tells it to examine the `package.json` file and install all dependencies. The second command says to execute the `dev` script. Vite will create a web server, typically running on port 3000, and can now serve the empty home page.

![HSMercs Helper on localhost:3000](/images/vite-localhost3000.png)

Sure, it's boring, but this is where everything starts. Now time to get these files added to the repo, committed to the tree, and pushed up to system of truth.

```bash
git add -A  # add the four files we created
git commit -m "minimum renderable code" # commit them to the repo with a comment
git remote add origin https://github.com/{username}/{repo} # add a github repo as the origin
git push --set-upstream-branch origin main # push the commit to a branch named main at the origin
```

## Minimum Renderable Vue

We'll start by creating our first Vue component. By convention, we name the entrypoint to the whole application as `App.vue` and place it in our `src` folder.

#### src/App.vue

```vue
<script setup lang="ts"></script>

<template>
<<<<<<< HEAD
  <div>HSMercs Helper</div>
=======
  <header>
    <div>HSMercs Helper</div>
    <div>A set of tools for Hearthstone Mercenaries players.
  </header>
>>>>>>> main
</template>
```

Also by convention, the Typescript file that serves as the initial setup for the entire application is `main.ts` in the same `src` folder.

#### src/main.ts

```typescript
import { createApp } from "vue";
import App from "./App.vue";

createApp(App).mount("#app");
```

This first imports the `createApp` function from the `vue` library and the `App` component we just created. Then it tells Vue to initialize with our component and mount it to the element with id `app`.

#### index.html

```diff
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8" />
      <meta name="viewport" content="width=device-width, initial-scale=1.0" />
      <title>HSMercs Helper</title>
    </head>
    <body>
+     <div id="app"></div>
+     <script type="module" src="/src/main.ts"></script>
    </body>
  </html>
```

Adding in the div where the `App` component should be mounted and running the `main.ts` file. By specifying the type as "module", we are telling the browser that this script should be executed in "strict" mode. The browser will also wait to execute the code until the DOM is ready.

### Updating the Tools

If you try to run `yarn dev` right now, you'll receive an internal server error.

![Internal server error: Failed to parse source for import analysis because the content contains invalid JS syntax. Install @vitejs/plugin-vue to handle .vue files.](/images/vite-installplugin-vue.png)

#### package.json

```diff
  {
    "name": "hsmercs-helper",
    "version": "0.2.0",
    "private": true,
    "engineStrict": true,
    "engines": {
      "node": "^16"
    },
    "scripts": {
      "dev": "vite"
    },
    "dependencies": {
      "vue": "^3.2.25"
    },
    "devDependencies": {
+     "@vitejs/plugin-vue": "^2.0.1",
      "vite": "^2.7.10"
    }
  }
```

Update the `yarn.lock` file (and the node_modules folder) by telling yarn to double-check the packages

#### vite.config.ts

```typescript
import vue from "@vitejs/plugin-vue";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [vue()],
});
```

We're not starting to add configuration options to the `vite` server. This is telling `vite` how to handle `vue`.

#### /src/env.d.ts

```typescript
/// <reference types="vite/client" />

declare module '*.vue' {
    import { DefineComponent } from 'vue'
    // eslint-disable-next-line @typescript-eslint/no-explicit-any, @typescript-eslint/ban-types
    const component: DefineComponent<{}, {}, any>
    export default component
  }
```

One little bit of tooling configuration before we wrap up this first step of the tutorial. If you check out the `src/main.ts` file, you'll see VSCode displaying red squiggles with the error "Cannot find module './App.vue' or its corresponding type declarations. ts(2307)". We need to add a shim to tell the Typescript compiler how to handle `vue` files.

```bash
yarn     # recheck dependencies, specifically install @vitejs/plugin-vue
yarn dev # run vite to start the server!
```

!["HSMercs Helper" heading](/images/vite-localhost3000vue.png)

### Final Housekeeping

In order to have this code hosted somewhere, you'll want to run the build command. It does all kinds of magic to take the various bits and pieces we've written, combine it with the Vue library, and make it as small as possible, and dump it into a `dist` folder. By convention, the script call is `build`.

#### package.json
```diff
  {
    "name": "hsmercs-helper",
    "version": "0.2.0",
    "private": true,
    "engineStrict": true,
    "engines": {
      "node": "^16"
    },
    "scripts": {
      "dev": "vite",
+     "build": "vite build"
    },
    "dependencies": {
      "vue": "^3.2.25"
    },
    "devDependencies": {
     "@vitejs/plugin-vue": "^2.0.1",
      "vite": "^2.7.10"
    }
  }
```

If you run the command `yarn build` (which is an alias for `yarn vite build`), git will think it needs to include the `dist` folder in the repository. We want to ignore this folder.

#### .gitignore
```diff
  node_modules
+ dist
```

## Step 2: [Storing State & Unit Testing](/posts/2022/storing-state-and-unit-testing)

In the interest of keeping hardcoded sample data out of the project, the next step will be to create a couple TypeScript interfaces, scaffold a `Vuex` store, and write the first unit tests.

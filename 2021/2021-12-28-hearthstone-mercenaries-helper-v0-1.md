---
title: Hearthstone Mercenaries Helper v0.1
category: Rovani's Vue
date: 2021-12-28T03:54:34.913Z
tags:
  - hearthstone
  - vuejs
  - tailwind
excerpt: "In an effort to get some in-depth experience with Vuejs and as a project to replace my Hearthstone Mercenaries Helper [spreadsheet](https://docs.google.com/spreadsheets/d/19FBZWszfu286zdRNZ43JvUD2bUvxLfrYTLmO1qSJmEM/edit?usp=sharing), I took the time to create an initial version of a suite of tools to aid in tracking mercenary progression in Hearthstone. Because my styling (this is applicable to most of my life, btw) is terribly basic, I pulled in Tailwind CSS for a bit of color and uniformity. I also later found that I needed to bring in a state tracker, so I learned a bit about Vuex."
---

In an effort to get some in-depth experience with Vuejs and as a project to replace my Hearthstone Mercenaries Helper [spreadsheet](https://docs.google.com/spreadsheets/d/19FBZWszfu286zdRNZ43JvUD2bUvxLfrYTLmO1qSJmEM/edit?usp=sharing), I took the time to create an initial version of a suite of tools to aid in tracking mercenary progression in Hearthstone. Because my styling (this is applicable to most of my life, btw) is terribly basic, I pulled in Tailwind CSS for a bit of color and uniformity. I also later found that I needed to bring in a state tracker, so I learned a bit about Vuex.

![Rathorian's details card in HSMercs Helper v0.1](/images/hsmercs-v01.png)

I built the site out a little backwards. First, I tried to think of how I could duplicate my spreadsheet, but that got quickly bypassed as I tried to make the layout closer to what the in-game details page looks like. Since I was doing it haphazardly, one feature at a time, I stumbled into a look. It's not a great look; it's serviceable.

## What Do I Like?

- The data is (nearly) complete. There are [some known issues](https://github.com/drovani/rovaninet/labels/good%20first%20issue) with missing data, but it's all sugar coating. The core tracking and mostly-accurate display of text is complete.
- The data is succinct and avoids duplication. For example, if a "description" for an ability's tiers only has one number that changes, then the data only needs to have that integer known, instead of each tier having its own string.
- The details component is _generally_ laid out like it is in-game. This makes it easier to update pieces and view information roughly where a player would find in the interface they are used to.
- The Equipment interacts with appropriate Abilities and stats (Attack and Health). When a user equips an item and changes the tier of abilities or equipment, the text and number values update to match.
- Selection choices persist to the browser's _local storage_, so interactions are retained between user sessions.

## Areas of Improvement

- I should have made HSMercs Helper its own repository, Netlify build pipeline, and host it via a separate subdomain. This way, the HSMercs project wouldn't be tied to the version restrictions that Gridsome has; and I would be a little more confident pointing others to the code without them having to wade through my personal blog.
- The colors are atrocious. Tailwind CSS 3 has a [much broader default color palette](https://tailwindcss.com/docs/customizing-colors) that I am looking forward to utilizing. I will try to better match the in-game colors with the site.
- I should have put more focus on the mini-cards and allow for manipulation of tasks and ranks there. As I have been playing that game, having to go into the details view just to up a rank or mark a task as completed has been tedious.
- Implement sorting and filtering. To start, adding a Group-By-Role and then order alphabetically.

## Future Features

There are three other useful features in my spreadsheet that I would like to add to this site:

1. Highlight mercenaries that are close to getting another item. When a mercenary's next task is either Task 2 or Task 7, highlight that mercenary. This is what initially got me to create the spreadsheet, because when I was presented with the Mysterious Stranger, I couldn't remember which Mercs were already passed Level 7, which were close, and where where in some limbo between unlocking the third item.
1. "Bounty Rewards" that lists all of the weighted reward possibilities. It fades the bounties where one or more of the mercenaries are fully upgraded and highlights bounties for mercenaries missing my collection. Additionally, I can select multiple mercenaries that I want to focus on their coin collection.
1. Highlight synergies - finding a good UX experience to highlight other mercenaries that can trigger synergies like combos (i.e. "Fire combo:"), race dependencies (i.e. "if you control another Demon"), and enemy state (i.e. "if it's frozen").
1. Some light user accounts might be nice, to allow for persistence of collections beyond a single machine. I might use the [Battle.net OAuth](https://develop.battle.net/documentation/guides/using-oauth). As a stretch goal, I might also use the [GitHub OAuth](https://docs.github.com/en/developers/apps/building-oauth-apps/authorizing-oauth-apps)

## What's Next?

First thing's first - and that's to do what I should have done from the beginning: create a new repository for this project. Once the rebuild is at feature parity, the new home will be [https://hsmercs.rovani.net](https://hsmercs.rovani.net). I'll set up redirects once that process is complete.

Second step is to do a better job breaking down the user interface into components, instead of dumping everything into a few components and only extracting them when I stepped back and thought "whoa, that's too big". I am also undecided about how to reconcile my desire to have data models ("everything is a class!") with Vue handling data manipulation right in the SFC.

As I build Hearthstone Mercenaries Helper v0.2, I'll be writing step-by-step tutorials as a way to also teach someone else how to build in Vue. Then, and this is a _huge_ stretch goal, I'll do it all over again in React.

---
title: Stumbling Into Design Patterns
category: Vigil Journey
tags:
- vigil
- cqrs
- eventsourcing
- factory
- repository
- service
- ddd
date: 2016-09-14
---

As I have been working on the Vigil project, I have been accidentally stumbling onto various patterns. I will go in search of a way to solve some particular problem, and end up losing several hours while I research some reference the author made about a concept (or, usually, an acronym) at the end of a post as some kind of _a posteriori_. The two largest design patterns that I am most excited about are Command Query Responsibility Segregation (CQRS) and Event Sourcing (ES).

## Accidentally Implementing CQRS

I found that I wasn't enjoying how all of my database logic was being tied up first inside of my Controller. I refactored, pulling all of the database logic out and putting it into its own class (someone had used the word _Repository_ somewhere, so I called it that). I have always liked pulling code out of the Web Application project and putting it into its own project - so this new Repository got moved there. However, it meant that the View Models that I was using in the Controller could no longer be access by the Repository. Thus, the View Models got moved to the (what is now called) Domain project. But wait - now I have to modify a class in the Domain project if I want to change some data passed to the View. This didn't look right, so the View Models went back into the Web Project and a new set of classes were made - I called these Models. Ok, but now I need a way to turn the View Models into Models - or more accurately, extract the data from one and reconstruct it in the other. For this, I invented Mappers, which just handled shuttling data between my (what I found out were called) POCO classes. It turns out that Mappers can roughly be translated into Command Handlers, if I think of the Factory as a bus, and the POCO as my Command.

Thus, my commands and queries are separated out. Queries are happening in the Repositories, and Commands are happening in the Factories. Event Sourcing is also half way to being implemented, because my commands get pushed up to and object that then handles what to do with them. This means all I am really missing is separating the validation of a command from the actual execution of the command, and I still need to add Event Sourcing.

## Where Am I Going With This?

There are lots of examples of implementing little pieces of CQRS and ES here and there, spread throughout the web. What I could not find was a single series of posts about taking a completely greenfield project and building up each component of the code, including unit testing. From there, it would be a set of posts about integration and doing integration testing. I think that is where I want to take the Vigil Project. I will keep a branch with "snapshots" of where the code is at each blog post, and I can provide a post detailing each piece of the project. This will include my thoughts for each direction I took, places where I changed my mind (either voluntarily or by force), and the actual code that was generated.

We'll see how this goes. Any ideas on a rough template for what this series might look like? Let me know below.
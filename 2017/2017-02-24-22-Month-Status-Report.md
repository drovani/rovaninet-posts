---
title: 22 Month Status Report
category: Vigil Journey
treeid: Vigil/tree/dc688a9322bb3467b39c8d66a8c8883482e0a6f7
tags:
- vigil
- statusreport
date: 2017-02-24
---

As I was writing the [previous post](/posts/2017/introducing-vigil-web-api/), it came as a shock how far behind my posts on the progress of the Vigil Donor Relationship Management System had gotten. I was going too deep into the code and making too many changes that I didn't pause to write a post about my progress. My posts are 22 commits behind, and there are quite a few changes I've made to the workflow and to the application. As a way to catch up on all the missed posts that I should have written, I am putting this one together as a status report of the project/journey/solution.

In terms of workflow, I have embraced the [GitHub Flow workflow](https://guides.github.com/introduction/flow/) (both for Vigil and for this blog). For my little project, since there is no discussion and no deployments (as of yet), the workflow is as simple as Branch, Commit(s), Pull, Merge, Delete. I like the way that a merge can squash all of the commits made on a branch, so that I have one summarized collection of everything that I did. This way, I can easily differentiate the content of a post by doing one per Pull Request / Merge. Hopefully, this process will now keep the posts coming regularly, help me be more concise about what should be in a post, and better contain runaway developing.

<aside class="float-right">
    <img src="/images/vigil-project-22-months.png" alt="Vigil Projects after 22 Months" />
</aside>

## Quantity of code

As I look at the total row count of all of the code, I am actually surprised by how many lines of code I have produced. It is a pittance compared to what I could output if I had been working on this project full time for the last two years. However, for something that I have only been poking at every once in a while, I still find myself astonished that all of the simple "Create this" and "Update that" have turned into this much actionable code.

## Vigil.Domain

The Domain project contains a lot of interfaces and a few abstract classes. The key functionality that has been created here are the structure for what it takes to build a messaging and event sourcing system. More will probably be added here as a way to keep shared functionality across different implementations - be in for `Patron` entities or an Ordering and Inventory system or possibly a job taks/queue system.

The reading that I have done with regards to Domain Driven Design (DDD) talk about establishing a _Domain Language_. My interpretation is to enforce that in a shared project that creates the core concept of how the _domain_ should be structured and implemented. The rest of the application is just extending these interfaces and base classes and providing for meaning over the abstractions. There was a lot of flux in the earliest stages of the `Vigil.Domain` project, but it has already settled into a stable state. My assumption is that over time, this will be the project that undergoes the least amount of change, but that changes to it will have broad reaching effects.

### Vigil.Domain.Tests

The consensus for [testing abstract classes](http://stackoverflow.com/questions/243274/how-to-unit-test-abstract-classes-extend-with-stubs) seems to be to either never use abstract classes or to create extremely simple inherited classes and run the tests on those. I have chosen the later option (obviously) and so far the tests seem comprehensive. Though, as with most of this project, I have constant feelings of imposter syndrome where someone smart is going to come along and tell me how awful this whole thing is and how I'm doing it all wrong.

## Vigil.Patrons

As is the only concrete implementation of the _Domain Language_, this project is a semi-functional proof-of-concept. I keep trying to focus myself on considering it as production code, thus holding myself to the same requirements to unit test everything and keep functionality as a [complex web of small pieces](https://www.youtube.com/watch?v=R2Aa4PivG0g). Commands available handle the simplest of actions - `CreatePatron`, `UpdatePatronHeader`, `DeletePatron`. The `PatronCommandHandler` knows what to do with the commands, which is to just create the applicable event and push it to the Event Bus. The `PatronEventHandler` methods only update the `Patron` entity that was persisted using an `IPatronContext`.

Upon close examination, the Patron project is barely less of an abstraction than the Domain project from which it derives most of its functionality. It is  disheartening to know that even the after writing roughly 300 lines of code, there is very little that I can do with the code. However, I can also celebrate in knowing that it takes only 300 lines of code to be able to implement the core functionality fo what it takes to Create, Read, Update, and Delete a `Patron`. The only thing left (after I unit tested everything) was to wire the Patron project to a real implementation of all of the interfaces it was depending on.

## Vigil.Sql

I talked about this project in detail in _[Illusions of Queues and Buses](/posts/2017/illusions-of-queues-and-buses/)_. The short of it is that I wanted to put together an extremely simple implementation of the `ICommandQueue` and `IEventBus` so that I could use it in the `Vigil.WebApi` project.

## Vigil.WebApi

The [post previous to this](/posts/2017/introducing-vigil-web-api/) explains why I decided to venture into the API layer of the application, instead of continuing to implement more of the Patron project or begin working on another subsystem of the application. It certain has been gratifying to see everything come together and be properly integrated and _just work_.

## Future Tangents

There are a number of quality-of-life improvements that could easily cause me to stray down a long tangent of work. It would be nice if my project templates included all of the settings that I apply to each new project. These include:

- Set the "Author" to "drovani"
- Set "RepositoryType" to "git"
- Set "RepositoryUrl" to "https://github.com/drovani/Vigil"
- Clear the "Description"
- Set "SignAssembly" to "true"
- Set the "AssemblyOriginatorKeyFile" to the `vigil.snk` in the root solution folder
- For Test projects:
  - Set the "RootNamespace" to be the same as the project being tested.
  - Add `xunit.runner.json` and set the "methodDisplay" to "method"
  - Include the `Moq`, `xunit`, and `xunit.runner.visualstudio` NuGet packages

Other side projects would include putting together all of the shiny automated build routines that all of the big projects have. Being able to have Pull Requests kick off builds to make sure nothing breaks - that would be cool. Turning all of the projects into NuGet packages and making them a little more portable would be an interesting experiment.

My problem is that each of these tangents only delay my progress on creating productive code that leads the project to a useful state.
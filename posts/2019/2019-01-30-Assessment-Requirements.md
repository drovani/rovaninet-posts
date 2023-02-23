---
title: Technical Assessment - Requirements and First Pass
series: Technical Assessment
category: Rovani in C♯
treeid: techassessment-basic/tree/528a447383b5ce2037b042138b7f5b2a3ac76476
tags:
- career
- angular
- webapi
date: 2019-01-30
---

Yesterday, I did a phone interview with a company offering the possibility for a rather exciting position. The person I spoke with seemed very intelligent, articulate, passionate, and sold the organization well. It's a small organization that is growing, and is looking to consolidate all of their development work into their Chicago office instead of relying on developers in foreign countries. This feels very similar to [what I did](/about) with _Falkor Group_ and again with _The Fellowship_ — start with a small team, map out a strategy for the next several sprints, then mentor and grow the team from there. Of course, the interview process is a two-way sales process, and they need to make sure that I can live up to expectations. This leads to why I am doing a Technical Assessment.

> The first in a four-part series of posts on ['An Assessment of a Technical Assessment'](/series/technical-assessment-series).

The requirements are fairly straightforward.

> **Technical Assessment**
>
> This assessment is intended to give us a better idea of your technical ability and code style as a developer. It will also give you an opportunity to learn about and utilize the tools we work with at the company.
>
> **Instructions**
>
> Create a dynamic blogging application that meets the following requirements:
>
> - Allow the user to view blog posts in chronological order
> - Allow the user to create a blog post
> - Allow the user to remove a blog post
>
> **Technologies**
>
> You must use the following technologies to complete this assessment:
>
> - .NET 4.6+ / .NET Core
> - Angular 5+
> - ngrx/store
> - Entity Framework
> - SQL Server
>
> **Notes**
>
> - You do not have to implement user authentication into your solution
> - Your code should follow SOLID principles and make use of modern design patterns
> - Be prepared to explain your solution in depth

## Visual Studio -> New Solution

I set off right away to get working - this should be easy, right? I've never used ```Angular``` before (Knockout is the JavaScript framework of choice at _The Fellowship_), and I've never used ```ngrx/store``` (client state isn't that complicated in _KesherNet_). But Visual Studio has a template for creating a new ASP.NET Web Application using ```.NET Core``` and ```Angular``` - this should be super easy!

![New ASP.NET Core Web Application](/images/techass-new-core-web-application.png)

1. By the power vested in me, I command thee to build!
1. Oh, this project requires ```Node.js``` to build; so let's install that.
1. ... lots of progress bars.
1. Great - now let's build and run this barebones project!
1. Internal Server Error? huh - well let's do a search for this error message "The NPM script 'start' exited without indicating that the Angular CLI was listening for requests."
1. [This Stack Overflow answer looks promising](https://stackoverflow.com/a/53162998), I'll just do that.
1. ... more progress bars. "MSBUILD : error MSB3428: Could not load the Visual C++ component "VCBuild.exe"." That can't be good.
1. [Another Stack Overflow answer](https://stackoverflow.com/a/39235952)
1. ... more progress bars. But it's all green text - so that should be good.
1. New let's build and run this barebones project!

## Hello, World

![Hello, World](/images/techass-hello-world.png)

Well that worked out well enough - time to break this template down and make it do what I want.

The sample comes with three components - Home (display static content), Counter (show ability to modify non-persistent state on the page), and Fetch Data (demonstrate calling a Web API endpoint). I deleted the Home and Counter components, then renamed everything "Fetch Data" to "Home".

The initial object is a ```WeatherForecast```, which I repurposed to be my ```BlogPost``` interface. The ```component.html``` is easy enough to repurpose to spit out the data coming from the controller. The new ```HomeController``` was refactored to return a collection of ```BlogPost```. I added a second project to hold a ```BlogPostService``` which will handle the data retrieval. Later, this can be refactored to use EF and SQL Server to store and retrieve the data.

After throwing together some quick static seed data, the project was ready to go. I had a few typos and case sensitivities that needed correction. Once I got all of that straightened out - I finally had the initial pass at a "Recent Blog Posts" application.

![Recent Blog Posts](/images/techass-recent-blog-posts.png)
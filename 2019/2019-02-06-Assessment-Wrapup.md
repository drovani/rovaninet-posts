---
title: Technical Assessment - Final Notes
series: Technical Assessment
category: Rovani in C♯
tags:
- career
- angular
- entityframework
date: 2019-02-06
---

The primary piece of learning for me with this assessment was everything related to the front-end of the application. Creating a Web API in .NET and using Entity Framework is where my core developer strengths lie. While _The Fellowship_ doesn't currently use .NET Core anywhere, my side projects and code experiments have all been in the new libraries as I prepare the team to make the shift. This was a fun little activity as a way to quickly get the basics of how Angular works - creating components, routing requests, abstracting communication with the server, and binding with the user input.

> The final post in a four-part series of posts on ['An Assessment of a Technical Assessment'](/series/technical-assessment-series).

Framing this mini-project as a look into whether this is the "best I can do", it certainly falls short. In the interest of time, there were several features that I had to cut.

- There is no meaningful user input validation
- I never found the right place to use `ngrx/store` in the application.
- I did not implement any server-side data validation
- I did not create/implement a SQL Server database, instead just using the `InMemoryDatabase`
- I did not roll out a SQL migration script or do any custom model building
- There is no retry policy on persisting requests to the database
- Error logging is non-existent - I could have easily dropped in Application Insights, but it seemed overkill for this project
- There are no unit tests, integration tests, or other automated testing to ensure the code is working as expected
- There is minimal dependency injection, only for passing the data context around.
- Least importantly, the UI looks terrible. But I never claim to be a designer, so I'm ok with that.

This project serves well as a Proof of Concept. It shows that I can quickly learn the basics of a new (to me) JavaScript framework, put together a simple solution to prove I can do it, and check it all into GitHub for others to examine.
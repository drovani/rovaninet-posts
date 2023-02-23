---
title: Vigil Base Solution
category: Vigil Journey
treeid: Vigil/tree/8c41b3dc782066e2c03d51633dc7e65e3df3d924
tags:
- vigil
- unittest
- codecoverage
- xunit
- moq
date: 2015-09-25
---

I've finally done it; I finally ripped, twisted, and otherwise cajoled the default MVC template, Identity code, and a few other tweaks I wanted to make into something I am semi-confident will be a good starting point for future projects. Which is really to say ? I'm at a stopping point and I'll probably burn it all to the ground, tomorrow.

## The Best Things In Life Are Passed Tests

Sometimes, the best thing that can happen in my day is that all of the unit tests pass. Not just once, due to some odd coincidence with timing, but because they actually, legitimately pass. I am sure that veterans of Test Driven Development will scoff at a lot of these tests. However, it was fun to learn how to use [xUnit](https://xunit.github.io/) and [Moq](http://www.moqthis.com/). Previously, I had been determined that I was going to be able to write my tests without using external libraries. I was just going to extend the different classes that depend on external resources and change them all to work in memory. This was stupid, time consuming, and generally fraught with errors. My tipping point was when I realized I was writing tests for these Test classes.

How awful is that?

I chose xUnit and Moq because they seemed to be the most popular. The primary difficulty that I have with mocking at this point are extension methods. Since I cannot mock a non-virtual member, mocking the extension classes is impossible. Thus, I need to find the source code, figure out exactly what the extension method is doing, and properly mock whatever method is used internally. It was a frustrating set of investigations, but I did enjoy digging into the internals of Identity and [Katana](https://katanaproject.codeplex.com/).

## Tests that Cover Code

It feels like half of my tests are in place just to improve my Code Coverage percentages. Even with that, there are two sticking points that I am having trouble getting over. The first are some the set portion of automatic properties in my [Identity](https://github.com/drovani/Vigil/blob/VigilBaseSolution/Vigil.Data/Vigil.Data.Core/Identity.cs) and [TypeBase](https://github.com/drovani/Vigil/blob/VigilBaseSolution/Vigil.Data/Vigil.Data.Core/TypeBase.cs) abstract classes. I have searched high and low and cannot find a solution to this. I am hoping that as I inherit from these classes, tests on those new classes will happen to start covering these properties.

The second part is managing to get the MoveNext() method to register as covered. It's a pain to get [Code Coverage with Async Await](http://blogs.msdn.com/b/dwayneneed/archive/2014/11/17/code-coverage-with-async-await.aspx), but Dwayne Need's blog post was very informative. It will be a while before I worry about getting that in depth with the code coverage that I will need to put all this work together for what amounts to no (seemingly) real gain.

Another "close enough" for Code Analysis

Most issues that the utility would flag were ones that I was able to fix. Rules like "CA1063 Implement IDisposable correctly" and "CA2000 Dispose object before losing scope" are easy to correct. The two rules that I completely turned off were "CA1056 URI properties should not be strings" and "CA1062 Validate arguments of public methods". The former, because it was a monumental pain to convert everything in my code to URIs, but return them back to strings when sending them to different frameworks; the later, because it did not [account for Code Contracts](http://geekswithblogs.net/terje/archive/2010/10/14/making-static-code-analysis-and-code-contracts-work-together-or.aspx).

The final block of rules that I am ignoring for now are "CA1020 Avoid namespaces with few types" and "CA1704 Identifiers should be spelled correctly". CA1020 is a valid notice, but the project is still in its infancy and namespaces are going to be sparce. CA1704 is flagging the words "Owin" and "POST", even though both of them are in my CustomDictionary.xml file, located in each project. I'm sure I'll stumble on a fix for this at some point.

> Code Coverage is down to only missing the .MoveNext() method calls.
>
> Code Analysis is reduced to just "POST" and "Owin" identifier issues and Avoid namespaces with few types.
>
> All tests are passing. Stamping this as the first official branch.
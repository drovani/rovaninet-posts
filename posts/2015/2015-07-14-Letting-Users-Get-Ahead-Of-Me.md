---
title: Letting Users Get Ahead Of Me
category: Vigil Journey
treeid: Vigil/tree/69a7825ee77eea429e6e60ff3c6534b9febabed1
tags:
- vigil
- aspnetidentity
- boundedcontext
date: 2015-07-14
---

In the quest to get to a Minimum Viable Product, I wanted to be able to wrap together a set of projects that would act as a complete framework for future projects. Every (nearly every) project requires users and a way for them to register and login. I had already tied myself to the [IdentityDbContext](https://msdn.microsoft.com/en-us/library/dn613255.aspx) for user and role management. Initially, I had my base context ([VigilContext](https://github.com/drovani/Vigil/blob/master/Vigil.Data/Vigil.Data.Modeling/VigilContext.cs)) inherit from IdentityDbContext. I had trouble coming to grips with having every project then required to bring in the Microsoft.AspNet.Identity.EntityFramework assembly. As I was explaining a completely unrelated problem to a coworker, it dawned on me that the whole point of [Bounded Contexts](https://msdn.microsoft.com/en-us/magazine/jj883952.aspx) was to have multiple contexts, and limit them to only a narrow scope. VigilContext was not going to be used everywhere; its purpose is for database initialization and future migrations.


## Enter the 'Vigil' Solution

This is a terrible idea, and I am probably going to regret it later, but I do not have a better way to organize everything just yet. I started a new Solution that will be all-encompassing, containing all of the projects, neatly organized into Solution Folders. This will get unwieldy, eventually, but for now I know that everything compiles, all tests pass, and code analysis returns acceptable results. Creating new solutions down the line for specific sets of projects should be rather trivial.

## Vigil Identity

The primary usage that requires a class that inherits from IdentityDbContext is to use the rest of the Identity library and the [OWIN authentication](http://coding.abel.nu/2014/05/whats-this-owin-stuff-about/), including the [RoleStore](https://msdn.microsoft.com/en-us/library/dn613257.aspx), [UserStore](https://msdn.microsoft.com/en-US/library/dn613259.aspx), [SignInManager](https://msdn.microsoft.com/en-us/library/dn896559.aspx). Therefore, a new project, created with the same naming pattern in the based modeling project, has a context that inherits from IdentityDbContext and all of the customized classes that inherit from the Identiy and Owin generics. The most common way of managing users is through a web application. Which means that any ancillary projects that do not actually manage users will be able to have bounded contexts that do not include the Identity assembly.

## Got Ahead of Myself - Vigil.Web

I was so excited about being able to add and edit users that I jumped ahead and created a new web project utilizing the Owin context. What a terrible mistake that was. I spent too much time focused on converting the default template to utilize all of my Vigil classes, and then suddenly stressed over not having tests for this new web project. Then I got myself buried under needing to figure out how to mock a Role Store, User Manager, and User Store in memory; and then how to start a whole test Owin context; which lead to attempting to create a mock Owin server; which lead to despair and two months since my last post.

## Paying Down Debt

Like missing a payment on a credit card, I need to get this debit under control right away. I cannot just make the minimum payment and figure out later how to pay off the balance and exploding interest. I am going to target just the unit testing surrounding the Identity Model project. Much of this will come from looking at the source code for the Identity Framework, and learning a lot from that.

That is my update - not much to say except for reminding myself to post more often, and make a meaningful, complete contribution to the project before spinning out of control.

> Moved files around to better keep a uniform folder structure. Reduced the solutions down to one, though this will probably go back up once executables need creating.
>
> Started adding testing framework for the OWIN and app server to be able to test Identity controllers.
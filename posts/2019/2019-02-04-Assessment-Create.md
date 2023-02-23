---
title: Technical Assessment - Adding Create Functionality
series: Technical Assessment
category: Rovani in C♯
tags:
- career
- angular
date: 2019-02-04
---

I am starting to make some real progress with this. I have a better idea of how components, service injection, and input binding works in Angular. These are all concepts that I am very familiar with in my extensive use of [Knockout](https://knockoutjs.com/), but anytime one is learning a new framework, there will be a learning curve in figuring out how _this particular framework_ implements various features. Next on the requirements list is the ability to create a new blog post.

> The third in a four-part series of posts on ['An Assessment of a Technical Assessment'](/series/technical-assessment-series).

## Creating a Form

Back to the Angular tutorial, this piece took several steps to get to where I needed to be to start writing code. Unfortunately, the opening tutorial ([Add a new hero](https://angular.io/tutorial/toh-pt6#add-a-new-hero)) only shows how to declare an input's id and then how to wire a button to pass the value of that element back to the component. This works fine for a single value, but I need the ability for a user to edit multiple fields and send them all to component. I now needed to jump into the "Fundamentals" section of the docs; specifically, the "[Forms](https://angular.io/guide/forms-overview)" section.

I decided to do a quick shuffle of files, and move the `blogpost` model into the "blogposts" folder. This is one of those decisions that I couldn't tell if it jives with convention, but it felt a little dirty keeping component-specific files in the root "app" folder. It's decisions like these that I'm excited to learn more about as I dig into larger Angular projects.

Creating the form, adding some styling to it, and binding it to a model in the component was all pretty straight forward. This is another case where knowing how one MV* framework (Knockout) informed me on what I was looking for, and the tutorial did an adequate job of showing how to put that concept into practice. After a bit of trial and error, I assembled the necessary controls to display a half-way decent form.

![Create New Blog Post](/images/techass-create-new-blog-post.png)

Working through everything, now was the time for the big test - would it actually submit my data to the server?

![POST Request Body](/images/techass-post-request-body.png)

Huzzah! This part works - now to the API controller to see if it's receiving the data.

![Controller Model Binding](/images/techass-modelbound.png)

Ok - well, it wasn't really that easy. I had to do a bit of refactoring from the default template. They were all expected changes, though. This part of the project was what I was most confident that I could get working fairly quickly.

I created a `BloggingContext` to handle the ORM side of things, and to make it easier, I just mapped it to an `InMemoryDbContext`. Changing the connection to a SQL database or an Azure Cosmos Db account would be fairly trivial.

## I Think I'm Done

At this point, the chief requirements are completed.

1. <i class="fas fa-check-circle" style="color: green;"></i>Allow the user to view blogs posts in chronological order
1. <i class="fas fa-check-circle" style="color: green;"></i>Allow the user to create a blog post
1. <i class="fas fa-check-circle" style="color: green;"></i>Allow the user to remove a blog post

I'll write one more post about where I would take the solution from here if I wanted to really put together a complete software package.
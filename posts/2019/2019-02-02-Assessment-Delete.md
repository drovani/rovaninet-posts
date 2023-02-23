---
title: Technical Assessment - Adding Delete Functionality
series: Technical Assessment
category: Rovani in C♯
treeid: techassessment-basic/tree/590d44659fd286391e44d12fbce753963d71328a
tags:
- career
- angular
date: 2019-02-02
---

Now that I have a static list, the next step is to make it slightly more dynamic. This means that I need to learn how to send something back to the server, instead of just retrieving data from the server. My guess is that a delete method should be easier than an insert of a new record, though I expect them both to be about equal. The first piece I recognized that I needed to add is an identifier for indicating _which_ record to delete. I am a big fan of using a `Guid` as the datatype for a "primary key", so I quickly added a new field to the `BlogPost` and added that to my seed data. But how to I make a button in `Angular` and then make a `DELETE` call to the controller with the id of the selected post?

> The second in a four-part series of posts on ['An Assessment of a Technical Assessment'](/series/technical-assessment-series).

It was time for me to turn to the [tutorial](https://angular.io/tutorial). The _Tour of Heroes_ tutorial on the Angular website certainly looked to be what I needed. I tried jumping straight to ["Delete a hero"](https://angular.io/tutorial/toh-pt6#delete-a-hero), but it's talking about services and delegates and... whoa. Ok, time to take it back to the top.

## Refactoring the Template

I zipped back to the top of the tutorial and quickly made sense of a lot of the work that the initial Visual Studio template created for me. Much of what the early parts of the tutorial teach the user were refactors that I thought would be necessary, but I didn't know how to do it yet.

- Refactor the `BlogPost` interface out to its own class.
- Refactor the data retrieval into a `BlogPostService` and inject it into the component.
- Rename the `home` component to be `blogpost` - it's still that I'm still using the generic name.
- Refactor the `blogpost.component` to use the new service.

What do I get after all that refactoring? Exactly the same output - which is what I had hoped for. No new functionality has yet been added, but I feel better about where things are.

![Recent Blog Posts](/images/techass-recent-blog-posts.png)

## Delete a ~~hero~~ Blog Post

1. Add a delete button and have it call a delete method.
1. Create a delete method and have it call the delete service.
1. Create a delete method in the service, and have it call the API server's delete method.

![HTTP404: NOT FOUND on DELETE](/images/techass-404-delete.png)

Oh, and I tried all kinds of things to get it to work. I tried explicitly setting the route in the attribute. I tried explicitly setting the action in the route. I tried using different action names in the URL. I tried turning it into a `GET` method. NOTHING WORKED!!! What was I doing so very wrong???

After a much longer time than I wish to admit, the problem was that I needed to put the id parameter in the attribute. Clearly I needed more coffee this evening.

``` csharp
[HttpDelete("{id}")]
public void DeleteBlogPost(Guid id)
{
    new BlogPostService().DeleteBlogPost(id);
}
```

From there, I was able to verify that the call made it into the `BlogPostService` that I had previously created. It properly removed the item from the original list and returned a `200:OK` response. All is well, and I am going to call this step complete. Once I get the entire front-end working, that's when I will handle persisting these changes to some storage medium. This part reminds me of my [User* Can Create**** New Patron*****](/posts/2016/user-can-create-new-patron/) post from 2016.
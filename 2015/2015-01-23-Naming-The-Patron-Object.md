---
title: Naming the Patron Object
category: Vigil Journey
tags:
- vigil
date: 2015-01-23
---

> There are only two hard things in Computer Science: cache invalidation and naming things.
>
> -Phil Karlton

To any seasoned developer, this is entirely too true, and [many](http://www.quora.com/Why-is-naming-things-hard-in-computer-science-and-how-can-it-can-be-made-easier)
[people](http://blog.stackoverflow.com/2009/03/it-stack-overflow-update-naming-is-hard/) have [written](http://seesparkbox.com/foundry/naming_css_stuff_is_really_hard)
extensively on why this is so hard. For this particular exercise, I have to decide what to the call the core object that represents a (potentially) giving unit in Vigil. Trolling through existing applications, here some options I have come across, and why I rejected them:

- Client - feels too impersonal for a DRM, though I know there are non-profits that use this as their preferred term.
- Constituent - the term is mostly used with voting system. Also, I am really loathe to type out "Constituent" thousands of times.
- Customer - this term is associated more with a point-of-sale system than a donation tracking system.
- Donor - not all names in the database are going to be actual donors. Some will just be people that purchased something, many will not even have given (in a system I recent worked on, around 55% of the names were non-donors).
- Entity - this conflicts heavily with Entity Framework, where the Entity object is used everywhere.
- Patron - this is a compelling term, since it has the dual definition of being both a person who gives financially, and a customer to a store.


The other part of the name is that it needs to pair with all of the auxilary tables that will be prefixed with:

- ______Address
- ______Attachment
- ______BankAccount
- ______Code
- ______ContactBase
- ______CreditCard
- ______Date
- ______Email
- ______Identifier
- ______Note
- ______Number
- ______Payment
- ______Person
- ______Phone
- ______Relationship

And that's just the preliminary list!

I had initially gone with Entity, since that it is what our current system uses. However, I quickly tired of running into the namespace conflicts with EF, and thus went searching for alternatives. I had almost settled on Constituent, until I went to start the re-factoring and after the twelfth time of typing it, realized it was going to cause [CTS](http://www.ninds.nih.gov/disorders/carpal_tunnel/detail_carpal_tunnel.htm). Patron was a late find that I stumbled upon. Thesauruses are my friends. So far, I am really digging it. I figure I get one re-factoring before I hit the point of no return. I am hoping this is the right one.
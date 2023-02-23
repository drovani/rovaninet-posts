---
title: Preemptively Structuring the Chaos
category: Vigil Journey
tags:
- vigil
date: 2015-05-13
---

I have found that it is better to put structure in place around a project before going hog wild on implementation. While I recognize that some practices grow organically, a healthy amount of structure up-front can save a project from technical bankruptcy down the line. The two tools that I use early and often are interfaces and contract classes.


## Interfaces and Contract Classes

There are a collection of interfaces that I regularly utilize throughout most projects.

- ICreated
  - VigilUser CreatedBy
  - DateTime CreatedOn
- IModified : ICreated
  - VigilUser ModifiedBy
  - DateTime? ModifiedOn
  - bool MarkModified(VigilUser, DateTime)
- IDeleted
  - VigilUser DeletedBy
  - DateTime? DeletedOn
  - bool MarkDeleted(VigilUser, DateTime)
- IOrdered
  - int Ordinal
- IEffective
  - DateTime EffectiveOn
  - bool SetEffectiveOn(DateTime)
- IEffectiveRange : IEffective
  - DateTime EffectiveUntil
  - bool SetEffectiveRange(DateTime, DateTime)

## The Obvious — ICreated, IModified, IDeleted, IOrdered

These four interfaces serve to ensure that all properties are identically named throughout the solution. They also serve as a guide to future developers about what restrictions should be placed on the class. Every class that persists data to storage should implement ICreated — everything was created by someone and at some time.  An object that doesn?t implement IModified should never actually be modified. A class with IDeleted should not have the records actually removed from storage, but just fill in these fields to flag them as deleted. IOrdered means we know to set a default sort on the Ordinal field.

## Instead of Deleting It — Effective It!

For values where we need historical data to be easily accessible, such as changing schedules or payment information, an interface is created to track the EffectiveOn date. Starting at that specific date and time, the new record becomes effective. There needs to be some way to group the different records, which is usually performed with a header type table. By definition, there can only be one effective record at any given date.

An example of its use is storing the payment information for a recurring transaction. The historical data for payment information is useful when looking into the past, and frequently the patron may want to change the payment option effective some future date. However, at no point will two or more payment methods be active. By implementing the IEffective and ICreated, the details of who made the change (and when) fully replace any functionality that might be lost by not implementing IModified.

### Need Multiple, Overlapping Effectives — Range It!

When multiple records need to have their history saved or allow for changes effective in the future, the IEffectiveRange comes into utility. For IEffective records, the end of their effectiveness is the beginning of the next record?s effective date. However, for classes where there can be multiple records that come and go, an explicit range is required. This is useful for line items on a gift ? do see when any given detail was the authoritative record, especially useful after the gift has been posted and receipted.

### Contract Classes For

I battled back and forth with whether to include the shell for code contracts with every interface I write, or to only build it out when needed. I quickly found that I was including, at minimum, a contract invariant method on each interface, so I decided to just make it a standard part of every interface. Even if the Contract Class contains no Contracts, at least the shell is there, and I can easily add them later, as needed.

A quick standard I have put together is that all Contract Classes are contained in the same file as the interface, in a child namespace of ?Contracts,? and named after the interface. Keeping the classes as internal and abstract means it can only be called within the assembly,

### Minimum Viable Framework

I finally feel that I am at a stage where I have the minimum of a framework on which I can start to build an actual product. From here, I will slowly add and expand modules, slowly drifting towards the user interface, and tool with refactoring, templating, scripting, and finding other ways that I can reduce the complexity of complex tasks.

> Added generic interfaces, and began cleaning up some Code Analysis rules from the ?All Rules? playlist.
>
> —[Commit 4010e840a8b168a1ab65466fa5c61cc342b56d8e]()

> Added new Code Analysis rule sets to eliminate rules that I would be ignoring anyway.
> Resolved several Code Contracts and Code Analysis issues.
>
> —[Commit 2b1ff8a55711cb585750d4241df9c97895b7edfc]()

> Decided I didn?t like having fields for the context, testuser, and now. Put them back as variables in each test method, and added a ContractVerification(false) attribute to the whole class. I think I?m going to find myself adding that attribute to a lot of testing classes.
>
> —[Commit df9e61081f22471546dd1fd1132c8ecfecc3a26c]()

> Added Contract Classes for each of the Interfaces.
> Created the Identity and TypeBase classes to expand the foundation of classes before working on POCO?s.
> Need to add more unit tests to fill out the Code Coverage.
>
> —[Commit 716da101f6da28158ab0ed52aaba037b4c027676]()

> Added ExcludeFromCodeCoverage attributes to the ObjectInvariant methods in the interface contract classes.
> Added tests for the TypeBase class (via inheritance through a TestTypeBase class).
> Filled out the ChangeLogTests tests.
>
> —[Commit bcc51aeaa46dc8780ffcd9c33f88d8ecb1c2fffd]()

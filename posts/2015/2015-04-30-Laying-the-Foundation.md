---
title: Laying the Foundation
category: Vigil Journey
tags:
- vigil
- aspnetidentity
date: 2015-04-30
---

Two key library decisions needed to be made this week: choosing the object-relational Mapping (ORM) framework, and choosing a membership provider. To cut right to the end, Entity Framework and ASP.NET Identity were the solutions that I have chosen to utilize throughout the software. This week's commit has been getting the initial implementation created, customized, and tested.

- Create NotImplementedAttribute
- Extend Identity membership classes
- Create Interface for data context
- Create Data Context
- Create minimal tests
- Minor Clean ups


## NotImplementedAttribute

Sort of like a TODO in the code, I pepper classes that still need work, or are just a placeholder shell with this attribute. This makes it so I can easily add a Code Analysis rule to mark it as incomplete, or I can add an ObsoleteAttribute to the NotImplementedAttribute. The compiler will issue a warning and I can easily track down any places I left marked as Not Implemented.

## Vigil Identity Classes

By default, the Identity classes use a string as the primary key. Internally, it is a serialized Guid, but I could not find a good reason why this was masked. Also, I wanted to play with inheriting the classes ? so I did exactly that. For now, the "System" classes are just the Identity classes, with a Guid key, and table names explicitly set to match the class name (thus overriding the [IdentityDbContext's default](http://stackoverflow.com/questions/29904898/classes-inherited-from-identity-objects-not-included-in-code-first-migrations) of naming them "AspNet_____").

## Data Context

Now is when the fun gets to start ? time to start making some actual progress towards real data.

1. Create the IVigilContext interface, to expose the bare minimums.
1. Create the Database Context: VigilContext
1. Inherit it from the IdentityDbContext generic class (so I can use those beautiful Vigil____ classes I created).
1. Inherit it from the IVigilContext interface and implement the "AffectedBy" and "Now" properties.
1. Create three tests:
  2. Initialize and Create the database ? this runs Entity Framework model validation. I wish I could find a way to do this without actually having to create the database, but I was unable to find a way to expose the EF method that runs the validation.
  2. Verify that the explicit constructor is setting the AffectedBy and Now properties. It is a menial test, but it makes sure no one breaks it in the future.
  2. Assert that the Set method works for at least one model. This serves two purposes ? it makes sure that the Context is calling its base class's Set method, and it makes it so that code gets covered by a test, thus passing Code Coverage analysis.

## Minor Clean Up Duty

I misspelled the name of the core project, spelling it "Vigi.Data.Core". This meant renaming the project, updating the Assembly name, default Namespace, the folder name (which also caused a rename on all files in the project), the namespace in existing classes, and references to the project. Thankfully, I caught it before I was creating objects that referencing those elsewhere.

In order to get complete Code Coverage for unit tests, I created a test just to call the constructor for the NotImplemented Attribute (it throws a NotImplemented Exception). However, the last line in that test is an Assert.Fail, which should never be reached. This causes the Code Coverage to say that line was never executed, and thus not covered. Obviously, I do not care about Code Coverage in the Testing project, so I added a codeCoverage.runsettings file, which explicitly excludes the Vigil.Testing.Data assembly from analysis.

> Added Vigil Users and Roles implementation of the Identity framework, and added two quick tests for VigilContext.
>
> —[Commit 9bca8542c5a0f099032631c133215a5bd9c28aae](https://github.com/drovani/Vigil/commit/9bca8542c5a0f099032631c133215a5bd9c28aae)

> Added Entity Framework and Identity Framework.
> Customized the Identity objects with Vigil objects, including using a Guid as the primary key, and adding a RoleType enum. Added a .runsettings file to exclude the Vigil.Testing.Data project from Code Coverage reports. Corrected the "Vigi.Data.Core" misspelling to "Vigil.Data.Core".
>
> —[Commit 8be35473aa84138f6ee362d083dd424726345089](https://github.com/drovani/Vigil/commit/8be35473aa84138f6ee362d083dd424726345089)

> Added tests to validate that the Vigil* tables have the correct names, since there was a problem with them being overwritten by the IdentityDbContext.
>
> —[Commit 4b68465701f111a806a920e7ec4d12de697aeee7](https://github.com/drovani/Vigil/commit/4b68465701f111a806a920e7ec4d12de697aeee7)
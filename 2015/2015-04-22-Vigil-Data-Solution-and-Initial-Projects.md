---
title: Vigil.Data Solution and Initial Projects
category: Rovani in C♯
tags:
- codecontracts
- strongnaming
- vigil
date: 2015-04-22
---

After several false starts, the convention for how to organize a solution seems to have come together. This also includes how to change the default project settings, and how to keep everything tidy as the software grows.


### Vigil.Data Solution

This solution will contain all projects in the Vigil.Data namespace. This includes the central collection of Entity Framework data models, projects for DDD Bounded Contexts, and each project needed for services. There will be future solutions for the web projects, automated routine projects, importing pipelines, export processes, etc.

- Vigil.Data.Code: where the Entity Framework Code First data models exist, including necessary abstract classes, interfaces, and extension methods.
- Vigil.Data.Modeling: contains all of the Entity Framework specific code that pertains to modelling the data into the data store (database), including the DbContext, Database Migrations, DbInitializers, and interfaces. This does not include any bounded contexts that may exist in the future.
- Vigil.Data.Services: services (which handle the business rules and workflows), specific-use data models, data mappers, validation, and supporting interfaces will all reside in this project.
- Vigil.Testing.Data: all unit tests will be rooted in the namespace Vigil.Testing, with the sub-namespaces reflecting the project(s) they are testing. Thus, this will have all of the tests for the Data projects.

Once these projects are created, there are three property pages that I immediately change from the defaults.

### Assembly Information

- Check the Company and Copyright information to make sure they reflect the project that I am creating (and not the computer's defaults).
- Set the Assembly Version to 0.1.* (default the Build and Revision Numbers)
- Set the File version to 0.1.0.0

Later, as the project grows and meaningful features are added, I will manually increase the minor version number as appropriate. Maybe even increase the Major Version, too!

### Signing

> Benefits of strong naming your assembly first:
>
> - Strong naming your assembly allows you to include your assembly into the Global Assembly Cache (GAC). Thus it allows you to share it among multiple applications.
> - Strong naming guarantees a unique name for that assembly. Thus no one else can use the same assembly name.
> - Strong name protect the version lineage of an assembly. A strong name can ensure that no one is able to produce a subsequent version of your assembly. Application users are ensured that a version of the assembly they are loading come from the same publisher that created the version the application was built with.
> - Strong named assemblies are signed with a digital signature. This protects the assembly from modification. Any tampering causes the verification process that occurs at assembly load time to fail. An exception is generated and the assembly is not loaded.
>
> —Why use strong named assemblies? answer by Kyle Rozendo

### Code Contracts

- Set the Assembly Mode to "Standard Contract Requires"
- Enable Runtime Checking at "Full"
- Set the [Contract Reference Assembly](http://stackoverflow.com/a/17892803/28310) to "Build"

> Initial projects for the Core data models, data modeling objects, data services, and data testing.
>
> Added public assembly signing, code contracts, application versioning, and copyright notifications.
>
> —[Commit 527fabb81ff6383f8ece46848feb2e329c131201](https://github.com/drovani/Vigil/commit/527fabb81ff6383f8ece46848feb2e329c131201)

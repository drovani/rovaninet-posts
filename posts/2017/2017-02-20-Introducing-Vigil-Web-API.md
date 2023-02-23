---
title: Introducing Vigil Web API
category: Vigil Journey
treeid: Vigil/tree/5936f903b81eb202c63da493e3d0808679843ce1
tags:
- webapi
- journey
- mvccore
date: 2017-02-20
---

Thus far in the series, the Vigil Journey project has been able to [create a patron](/posts/2016/user-can-create-new-patron/), [update a patron](/posts/2016/user-can-update-a-patron/), and have those redefined as "[a patron can be created and updated](/posts/2016/patron-can-be-created-and-updated-again/)". However, as acknowledged in those posts, this was all smoke and mirrors. Nothing was being persisted, and once the unit tests were run, there was no lasting effect of the code. All of the tests passed and there were lots of pretty green check marks, but no residual digital substance. Additionally, there was no application that could be left running with which someone could interact.

I began getting antsy about needing to have some semblence of real progress. Something that I could launch and point to and say "see there; it works!" This was my primary motivator for quickly (and sloppily) writing the [`SqlCommandQueue` and `SqlEventBus`](/posts/2017/illusions-of-queues-and-buses/) classes. I wanted to be able to demonstrate _something_.


## Recent Observations

While my initial instinct was to spin up a complete user interface for this; what better way to show progress than to open a web browser and display grid views and form fields, thought I. However, I also had grand schemes of building a full API for the UI to utilize. This kept my ambitions in check, and after researching for ways to directly interact with an API, allowing me to reach a more attainable goal. Thus, the next  layer of the project - which is to instantiate the `CreatePatron` command and pass it to an `ICommandQueue`. Utilizing my `SqlCommandQueue`, I could wire that to a database, which would handle persistence of `Command` objects, passing those to a `CommandHandler`. These would, in turn, generate events and put them on the `SqlEventBus`; which would call the appropriate `EventHandler` classes, and thus persist the changes to the relational database.

An unexpected (though, in hindsight, entirely predictable) revalation I had while adding this additional layer to the application is that each piece seems to have a higher cost to initialize the first proof of concept. When all I was looking for was the ability to _create a patron_, I could claim that "it works" when the unit test proved that a short piece of code did what it was supposed to do. The `PatronCommandHandler` class required no external libraries &emdash; in fact, going up the dependencie chain, `System` is the only required library because of `DateTime` and `Guid`. However, implementing even a cheap Command Queue or Event Bus requires the `EntityFrameworkCore` and `System.Linq` NuGet packages. Adding the ability to stand up even a small Rest API requires `Microsoft.AspNetCore`, `Microsoft.AspNetCore.Mvc`, `Microsoft.AspNetCore.Routing`, and if I want to use a webserver to host the code, then it requires bringing one or more of the `Microsoft.AspNetCore.Server` packages. There is a lot more experimentation, debugging, and unit testing required for each step. This is something that I'm going to have to keep in mind anytime I get the grand ambitions to tack on another layer to the application.

## ASP.NET Core MVC as an API

Previous versions of ASP.NET MVC separated out the `Controller` base class from the WebAPI controller base class. With ASP.NET Core MVC, there is no difference between an API controller and a web controller. I see this as being a real advantage when it comes to migrating code that was all crammed together in a web application, when an easier way to manage it would be as two different applications. Since a core goal of this entire _Vigil Journey_ is to separate responsibilities wherever possible, I like having the responsibility of user presentation being entirely separate from the responsibility of the command and query part of the workflow.

The `PatronController` is going to start as a simple controller that implements the four basic tenents of data: Create, Read, Update, Delete - or CRUD. Each action has a different Verb, Model, and return values.

|        | Http Verb | Model              | Success        | Failure              |
|--------|-----------|--------------------|----------------|----------------------|
| Create | Post      | CreatePatron       | Accepted (202) | BadRequest (400)     |
| Read   | Get       | PatronId           | Ok (200)       | NotFound (404)       |
| Update | Put       | UpdatePatronHeader | Accepted       | BadRequest, NotFound |
| Delete | Delete    | PatronId           | Accepted       | NotFound             |

```csharp
using Microsoft.AspNetCore.Mvc;
using System;
using System.Linq;
using Vigil.Domain.Messaging;
using Vigil.Patrons.Commands;

namespace Vigil.WebApi.Controllers
{
    [Route("api/[controller]")]
    public class PatronController : Controller
    {
        private readonly ICommandQueue commandQueue;
        private readonly Func<VigilWebContext> contextFactory;

        public PatronController(ICommandQueue commandQueue,
            Func<VigilWebContext> contextFactory)
        {
            this.commandQueue = commandQueue;
            this.contextFactory = contextFactory;
        }

        [HttpGet]
        public IActionResult Get()
        {
            using (var context = contextFactory())
            {
                var patrons = context.Patrons.Where(p => p.DeletedOn == null);
                return Ok(patrons);
            }
        }

        [HttpGet("{id}")]
        public IActionResult Get(Guid id)
        {
            using (var context = contextFactory())
            {
                var patron = context.Patrons.Find(id);
                if (patron == null)
                {
                    return NotFound();
                }
                else
                {
                    return Ok(patron);
                }
            }
        }

        [HttpPost]
        public IActionResult Create([FromBody]CreatePatron command)
        {
            if (command == null || !ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }
            else
            {
                commandQueue.Publish(command);
                return Accepted(
                    Url.Action(nameof(Get), new { id = command.PatronId })
                );
            }
        }

        [HttpPut("{id}")]
        public IActionResult UpdateHeader(Guid id,
            [Bind(nameof(UpdatePatronHeader.DisplayName),
                  nameof(UpdatePatronHeader.IsAnonymous),
                  nameof(UpdatePatronHeader.PatronType))] UpdatePatronHeader command)
        {
            if (command == null || !ModelState.IsValid)
            {
                return BadRequest(command);
            }
            using (var context = contextFactory())
            {
                if (context.Patrons.Find(id) == null)
                {
                    command.PatronId = id;
                    commandQueue.Publish(command);
                    return Accepted(Url.Action(nameof(Get), new {
                        id = command.PatronId
                    }));
                }
                else
                {
                    return NotFound(id);
                }
            }
        }

        [HttpDelete("{id}")]
        public IActionResult Delete(Guid id)
        {
            using (var context = contextFactory())
            {
                if (context.Patrons.Any(p => p.Id == id))
                {
                    commandQueue.Publish(
                        new DeletePatron(User.Identity.Name, DateTime.Now)
                        {
                            PatronId = id
                        }
                    );
                    return Accepted();
                }
                else
                {
                    return NotFound(id);
                }
            }
        }
    }
}
```

## Unit Tests before Functional testing

Before I attempted to launch Kestrel or IIS Integration, I needed to write a Unit Test. I got a little ahead of myself on this one. I wrote this first unit test, and then went off on a coding tangent, which caused a massive refactoring to occur. The unit tests never got completed for this version of the `PatronController`, but it does provide a quick overview as to how I handled the constructor requirements of the controller and checks for the return value.

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using Moq;
using System;
using Vigil.Domain.Messaging;
using Xunit;

namespace Vigil.WebApi.Controllers
{
    public class PatronControllerTest
    {
        public readonly ICommandQueue cmdQueue = Mock.Of<ICommandQueue>();
        public readonly Func<VigilWebContext> Context;

        public PatronControllerTest()
        {
            var serviceProvider = new ServiceCollection()
                .AddEntityFrameworkInMemoryDatabase()
                .BuildServiceProvider();
            var builder = new DbContextOptionsBuilder<VigilWebContext>()
                .UseInMemoryDatabase(databaseName: "TestHelper")
                .UseInternalServiceProvider(serviceProvider);

            Context = () => new VigilWebContext(builder.Options);
        }

        [Fact]
        public void Get_Returns_EmptyCollection_WhenThereAreNoPatrons()
        {
            var controller = new PatronController(cmdQueue, Context);

            var result = controller.Get() as OkObjectResult;

            Assert.NotNull(result);
            Assert.NotNull(result.Value);
        }
    }
}
```

If nothing else, this post reinforced that I need to write posts more often, and do a better job of isolating concepts into branches and merging small amounts of code at a time.
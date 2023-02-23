---
title: Hydrating A Patron
category: Vigil Journey
treeid: Vigil/tree/1b5152bcb40f3e372d13a86cc01685e17813dbf3
tags:
- vigil
- versionedevent
- eventsourcing
- unittests
date: 2017-01-23
---

There are two ways to retrieve the current values associated with an entity. The canonical source is the collection of events that describe how an entity came into existance and then was affected over time. The standard term for this seems to be "hydrating" the entity. Once the current state of the entity is found, it can then be persisted to a different medium for easier access. In the example of a `Patron`, there are few circumstances where the entire history of the entity is needed. Instead, most requests for the information will only be looking for the most current iteration - which is found in a snapshot database. One of the first challenges that I faced when building this initial prototype was proving that my code could rehydrate a `Patron` from only a list of events.

Looking into the details of what the process is when hydrating a `Patron`, the best place to start is with the constructor. The prelude to calling the constructor, of course, is that something has gone and fetched a collection of `IVersionedEvent` entities with a given `PatronId`.


```csharp
using System;
using System.Collections.Generic;
using Vigil.Domain.EventSourcing;
using Vigil.Patrons.Events;

public class Patron : EventSourced
{
    public Patron(Guid patronId, IEnumerable<IVersionedEvent> history) : this(patronId)
    {
        LoadFrom(history);
    }

    protected Patron(Guid patronId) : base(patronId)
    {
        Handles<PatronCreated>(OnPatronCreated);
        Handles<PatronHeaderChanged>(OnPatronHeaderChanged);
        Handles<PatronDeleted>(OnPatronDeleted);
    }
}

public abstract class EventSourced : IEventSourced
{
    private readonly Dictionary<Type, Action<IVersionedEvent>> handlers
        = new Dictionary<Type, Action<IVersionedEvent>>();

    protected void LoadFrom(IEnumerable<IVersionedEvent> pastEvents)
    {
        var orderedPastEvents = pastEvents.OrderBy(pe => pe.GeneratedOn);
        foreach (var e in orderedPastEvents)
        {
            handlers[e.GetType()].Invoke(e);
            Version = e.Version;
            events.Add(e);
        }
    }
}

```

I saw two options for where to put the actions that receive an event and cause an entity to be in the proper state. Either I could create an `EventHandler` that receives an entity and then conjures up a new instance of it with the resultant changes (thus making entities immutable), or I could have an entity know how to handle events that affect it (entities are thus mutable objects).

I liked the idea of keeping the logic of how an `EventSourced` object is affect consolidated to just the entity itself. This allows for an ORM (like [Entity Framework](https://docs.microsoft.com/en-us/ef/)) to directly assign data to the entity's properties, or for something else that has the collection of events to tell an entity to just _create itself_. Thus, the `Patron` constructor defines all of the events that it knows how to handle. When constructed with a collection of `IVersionEvent` objects, it calls the inherited `LoadFrom` method from `EventSourced`. My assumption is that the unit tests will be written to ensure that the `Patron` registers only events that are useful and that it handles all of the events that it should.

My initial goal was to create some sort of reflection assembly scanning that would find all methods that match the signature needed for an event and allow them to self-register. I might be able to create a custom attribute for that - perhaps a `HandlesEventAttribute`.

Anyway, each of the events, in creation order, get their appropriate method within the `Patron` object invoked, which updates the properties as appropriate. Unit testing is fairly straightforward.

```csharp
using System;
using Vigil.Domain.EventSourcing;
using Vigil.Patrons.Events;
using Xunit;

namespace Vigil.Patrons
{
    public class PatronTest
    {
        [Fact]
        public void Patron_Can_Be_Hydrated_From_Patron_Created()
        {
            var patronId = Guid.NewGuid();

            PatronCreated created = new PatronCreated("Create User", TestHelper.Now, Guid.NewGuid())
            {
                DisplayName = "Test Creation",
                IsAnonymous = false,
                PatronType = "Test Account",
                PatronId = patronId,
                Version = 0
            };
            Patron result = new Patron(patronId, new[] { created });

            Assert.Equal("Test Creation", result.DisplayName);
            Assert.False(result.IsAnonymous);
            Assert.Equal("Test Account", result.PatronType);
            Assert.Equal(patronId, result.Id);
            Assert.Equal(0, result.Version);
            Assert.Equal("Create User", result.CreatedBy);
            Assert.Equal(TestHelper.Now, result.CreatedOn);
            Assert.Null(result.ModifiedBy);
            Assert.Null(result.ModifiedOn);
            Assert.Null(result.DeletedBy);
            Assert.Null(result.DeletedOn);
        }

        [Fact]
        public void Patron_Can_Be_Hydrated_From_Patron_Created_And_Updated()
        {
            var patronId = Guid.NewGuid();
            var evnts = new VersionedEvent[] {
                new PatronCreated("Create User", TestHelper.Now, Guid.NewGuid())
                {
                    DisplayName = "Test Creation",
                    IsAnonymous = false,
                    PatronType = "Test Account",
                    PatronId = patronId,
                    Version = 0
                },
                new PatronHeaderChanged("Change User", TestHelper.Later, Guid.NewGuid())
                {
                    DisplayName = "Test Update",
                    IsAnonymous = true,
                    PatronType = "Test Updated",
                    PatronId = patronId,
                    Version = 1
                }
            };
            Patron result = new Patron(patronId, evnts);

            Assert.Equal("Test Update", result.DisplayName);
            Assert.True(result.IsAnonymous);
            Assert.Equal("Test Updated", result.PatronType);
            Assert.Equal(patronId, result.Id);
            Assert.Equal(1, result.Version);
            Assert.Equal("Create User", result.CreatedBy);
            Assert.Equal(TestHelper.Now, result.CreatedOn);
            Assert.Equal("Change User", result.ModifiedBy);
            Assert.Equal(TestHelper.Later, result.ModifiedOn);
            Assert.Null(result.DeletedBy);
            Assert.Null(result.DeletedOn);
        }

        [Fact]
        public void Patron_Can_Be_Hydrated_From_Patron_Created_And_Empty_Updated()
        {
            var patronId = Guid.NewGuid();
            var evnts = new VersionedEvent[] {
                new PatronCreated("Create User", TestHelper.Now, Guid.NewGuid())
                {
                    DisplayName = "Test Creation",
                    IsAnonymous = false,
                    PatronType = "Test Account",
                    PatronId = patronId,
                    Version = 0
                },
                new PatronHeaderChanged("Change User", TestHelper.Later, Guid.NewGuid())
                {
                    PatronId = patronId,
                    Version = 1
                }
            };
            Patron result = new Patron(patronId, evnts);

            Assert.Equal("Test Creation", result.DisplayName);
            Assert.False(result.IsAnonymous);
            Assert.Equal("Test Account", result.PatronType);
            Assert.Equal(patronId, result.Id);
            Assert.Equal(1, result.Version);
            Assert.Equal("Create User", result.CreatedBy);
            Assert.Equal(TestHelper.Now, result.CreatedOn);
            Assert.Equal("Change User", result.ModifiedBy);
            Assert.Equal(TestHelper.Later, result.ModifiedOn);
            Assert.Null(result.DeletedBy);
            Assert.Null(result.DeletedOn);
        }

        [Fact]
        public void Patron_Can_Be_Hydrated_From_Patron_Created_And_Deleted()
        {
            var patronId = Guid.NewGuid();
            var evnts = new VersionedEvent[] {
                new PatronCreated("Create User", TestHelper.Now, Guid.NewGuid())
                {
                    DisplayName = "Test Creation",
                    IsAnonymous = false,
                    PatronType = "Test Account",
                    PatronId = patronId,
                    Version = 0
                },
                new PatronDeleted("Delete User", TestHelper.Later, Guid.NewGuid())
                {
                    PatronId = patronId,
                    Version = 1
                }
            };
            Patron result = new Patron(patronId, evnts);

            Assert.Equal("Test Creation", result.DisplayName);
            Assert.False(result.IsAnonymous);
            Assert.Equal(patronId, result.Id);
            Assert.Equal(1, result.Version);
            Assert.Equal("Create User", result.CreatedBy);
            Assert.Equal(TestHelper.Now, result.CreatedOn);
            Assert.Equal("Delete User", result.ModifiedBy);
            Assert.Equal("Delete User", result.DeletedBy);
            Assert.Equal(TestHelper.Later, result.ModifiedOn);
            Assert.Equal(TestHelper.Later, result.DeletedOn);
        }
    }
}
```
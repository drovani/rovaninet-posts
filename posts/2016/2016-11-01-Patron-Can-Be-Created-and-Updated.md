---
title: Patron Can Be Created and Updated (Again)
category: Vigil Journey
treeid: Vigil/tree/63bbea9d206c6700efe58cb350390aac7a2aaeeb
tags:
- command
- commandhandler
- unittests
date: 2016-11-01
---

Now that I have revised my approach for what constitutes [creating and updating entities](/posts/2016/revisiting-creating-and-updating-entities/), I am going back to my `Patron` entity to rework the create and update commands. I have already scrapped the `Factory` class and its associated unit tests, and gone forward with code creation.

## Commands for Create and Update Patron

The first major change is that I have moved the Patron Commands into the `Vigil.Patrons` project; at this moment, I am keeping projects confined by domain. Later, I may find this to be in error, but for now it seems to make sense. I am keeping validation on the Command class and the unit tests for the validation scenarios does not need to change at all. The only change that I made was adding the `Id` property implementation for `ICommand`. Since I would like to track every part of the system, it makes sense to have unique identifiers for everything.

```csharp
using System;
using System.ComponentModel;
using System.ComponentModel.DataAnnotations;
using Vigil.Domain.Messaging;

namespace Vigil.Patrons.Commands
{
    public class CreatePatronCommand : ICommand
    {
        public Guid Id { get; set; } = Guid.NewGuid();

        [Required, StringLength(250)]
        public string DisplayName { get; set; }
        [DefaultValue(false)]
        public bool IsAnonymous { get; set; } = false;
        [Required, StringLength(250)]
        public string PatronType { get; set; }
    }
}
```

Along with some other shuffling around, I have dropped the notion of passing around `IKeyIdentity` and just using the `Guid`. I am never going to change the type of the key; but if I did, it would need to be a momumental undertaking anyway, since I would be changing _everything_. `Guid` is universally understood and unique, so I am just making life easier that way.

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using Vigil.Domain.Messaging;

namespace Vigil.Patrons.Commands
{
    public class UpdatePatronCommand : ICommand, IValidatableObject
    {
        public Guid Id { get; set; } = Guid.NewGuid();

        public Guid PatronId { get; set; }

        [StringLength(250)]
        public string DisplayName { get; set; }
        public bool? IsAnonymous { get; set; }
        [StringLength(250)]
        public string PatronType { get; set; }

        public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
        {
            if (PatronId == Guid.Empty)
            {
                yield return new ValidationResult("PatronId is a required field.", new string[] { nameof(PatronId) });
            }
        }
    }
}
```

## Patron Command Handler

Someone else will put a command in a message queue, and some other process will handle pulling that command off of the queue and instantiating the `PatronCommandHandler`. However, once all that work is taken care of (it's all part of the unit test's arrange step), it is time to turn the command into a series of events. While the current examples only show one event fired from the command, there could be many more. However, there are two (probably hard and fast) rules: #1 - there must be exactly one command handler per command (though one handler can handle many different commands), and #2 - every command must fire at least one event (if a command doesn't create an event, what's the point?).

```csharp
using System;
using Vigil.Domain.Messaging;
using Vigil.Patrons.Commands;
using Vigil.Patrons.Events;

namespace Vigil.Patrons
{
    public class PatronCommandHandler : ICommandHandler<CreatePatronCommand>
    {
        private readonly IEventBus eventBus;
        private readonly ICommandRepository repo;

        public PatronCommandHandler(IEventBus eventBus, ICommandRepository repo)
        {
            this.eventBus = eventBus;
            this.repo = repo;
        }

        public void Handle(CreatePatronCommand command)
        {
            var patronCreated = new PatronCreated
            {
                SourceId = command.Id,
                PatronId = Guid.NewGuid(),
                DisplayName = command.DisplayName,
                IsAnonymous = command.IsAnonymous,
                PatronType = command.PatronType
            };
            eventBus.Publish(patronCreated);
            repo.Save(command);
        }

        public void Handle(UpdatePatronCommand command)
        {
            var patronUpdated = new PatronUpdated
            {
                SourceId = command.Id,
                PatronId = command.PatronId,
                DisplayName = command.DisplayName,
                IsAnonymous = command.IsAnonymous,
                PatronType = command.PatronType
            };
            eventBus.Publish(patronUpdated);
            repo.Save(command);
        }
    }
}
```

## Command Handler Unit Tests

The purpose of a unit test for an `ICommandHandler` is to ensure the correct event(s) is/are put on the event bus, and that the command will be persisted through the repository. Later unit tests on the repository itself will determine if the entity was actually persisted, and later tests on the actual implementation of the event bus will determine if anything happens when an event is put on the bus. But for this unit test, the code is just checking to make sure the handler does what is expected.

```csharp
using Moq;
using System;
using Vigil.Domain.Messaging;
using Vigil.Patrons.Commands;
using Vigil.Patrons.Events;
using Xunit;

namespace Vigil.Patrons
{
    public class PatronCommandHandlerTest
    {
        [Fact]
        public void User_Can_Create_New_Patron()
        {
            CreatePatronCommand command = new CreatePatronCommand()
            {
                DisplayName = "Test Patron",
                IsAnonymous = false,
                PatronType = "Test Account"
            };

            var eventBus = new Mock<IEventBus>();
            eventBus.Setup(bus => bus.Publish(It.IsAny<PatronCreated>()))
                .Callback<PatronCreated>((@event) =>
                {
                    Assert.Equal(command.DisplayName, @event.DisplayName);
                    Assert.Equal(command.IsAnonymous, @event.IsAnonymous);
                    Assert.Equal(command.PatronType, @event.PatronType);
                    Assert.Equal(command.Id, @event.SourceId);
                    Assert.NotEqual(command.Id, @event.Id);
                    Assert.NotEqual(command.Id, @event.PatronId);
                    Assert.NotEqual(Guid.Empty, @event.PatronId);
                    Assert.NotEqual(Guid.Empty, @event.Id);
                }).Verifiable();
            var repo = new Mock<ICommandRepository>();
            repo.Setup(re => re.Save(It.Is<CreatePatronCommand>(cpc => cpc.Id == command.Id))).Verifiable();

            ICommandHandler<CreatePatronCommand> handler = new PatronCommandHandler(eventBus.Object, repo.Object);
            handler.Handle(command);

            Assert.NotEqual(Guid.Empty, command.Id);
            Mock.Verify(eventBus, repo);
        }

        [Fact]
        public void User_Can_Update_a_Patron()
        {
            UpdatePatronCommand command = new UpdatePatronCommand()
            {
                PatronId = Guid.NewGuid(),
                DisplayName = "Updated Test Patron"
            };

            var eventBus = new Mock<IEventBus>();
            eventBus.Setup(bus => bus.Publish(It.IsAny<PatronUpdated>()))
                .Callback<PatronUpdated>((@event) =>
                {
                    Assert.Equal(command.DisplayName, @event.DisplayName);
                    Assert.Equal(command.IsAnonymous, @event.IsAnonymous);
                    Assert.Equal(command.PatronType, @event.PatronType);
                    Assert.Equal(command.Id, @event.SourceId);
                    Assert.Equal(command.PatronId, @event.PatronId);
                    Assert.NotEqual(command.Id, @event.Id);
                    Assert.NotEqual(Guid.Empty, @event.Id);
                }).Verifiable();
            var repo = new Mock<ICommandRepository>();
            repo.Setup(re => re.Save(It.Is<UpdatePatronCommand>(cpc => cpc.Id == command.Id))).Verifiable();

            ICommandHandler<UpdatePatronCommand> handler = new PatronCommandHandler(eventBus.Object, repo.Object);
            handler.Handle(command);

            Assert.NotEqual(Guid.Empty, command.Id);
            Mock.Verify(eventBus, repo);
        }
    }
}
```
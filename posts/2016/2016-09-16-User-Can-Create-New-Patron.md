---
title: User* Can Create** New Patron***
category: Vigil Journey
treeid: Vigil/tree/cf213ec07134fbc4c7a1c3b58ce2cca027b0d099
tags:
- cqrs
- eventsourcing
- factory
- ddd
date: 2016-09-16
---

The first thing that I learned with Dependency Injection is that it encourages a mindset of "I'll figure that out that later." This allows me to put in the bare minimum for what might pass a test, even though I know it will have all kinds of complex integration somewhere down the line. To start this little experiment, I am going to put together a tiny amount of arguably functional code. I will start with the very simple definition of what a User is, what it means to Create an entity, and how a Patron is represented.


## *"User"

A "User" in this case is the arbitrary creator of a command and consumer of the factory. For now, the User is the test suite (powered by xUnit).

```csharp
using Moq;
using System;
using Vigil.Domain;
using Vigil.MessageQueue;
using Vigil.MessageQueue.Commands;
using Xunit;

namespace Vigil.Patrons
{
    public class PatronFactoryTest
    {
        [Fact]
        public void User_Can_Create_New_Patron()
        {
            var queue = new Mock<ICommandQueue>(MockBehavior.Strict);
            queue.Setup(q => q.QueueCommand(It.IsAny<ICommand>(), It.IsAny<IKeyIdentity>())).Verifiable();
            PatronFactory factory = new PatronFactory(queue.Object);

            IKeyIdentity result = factory.CreatePatron(new CreatePatronCommand()
            {
                DisplayName = "Test User",
                IsAnonymous = false,
                PatronType = "Test Account"
            });

            queue.VerifyAll();
            Assert.NotEqual(Guid.Empty, result.Id);
        }
    }
}
```

## **"Create"

At this point, I am going to define Create as issuing the command to _something else_ to instantiate and persist a new representation of the entity.

```csharp
using System.Diagnostics.Contracts;
using Vigil.Domain;
using Vigil.MessageQueue;
using Vigil.MessageQueue.Commands;

namespace Vigil.Patrons
{
    public class PatronFactory
    {
        protected readonly ICommandQueue _queue;

        public PatronFactory(ICommandQueue queue)
        {
            _queue = queue;
        }

        public IKeyIdentity CreatePatron(CreatePatronCommand command)
        {
            Contract.Requires(command != null);
            Contract.Ensures(Contract.Result<IKeyIdentity>() != null);

            var key = KeyIdentity.NewIdentity();

            _queue.QueueCommand(command, key);

            return key;
        }
    }
}
```

## ***"Patron"

A patron, this early in development, is an abstract representation of what should be created - it does not care about persistence or structure.

```csharp
using System.ComponentModel;
using System.ComponentModel.DataAnnotations;
using Vigil.Domain;

namespace Vigil.MessageQueue.Commands
{
    public class CreatePatronCommand : ICommand
    {
        [Required, StringLength(250)]
        public string DisplayName { get; set; }
        [DefaultValue(false)]
        public bool IsAnonymous { get; set; } = false;
        [Required, StringLength(250)]
        public string PatronType { get; set; }
    }
}
```

## ICommandQueue Interface

The most basic, simple interface with the bare minimum for what a message queue would need to implement.

```csharp
using Vigil.Domain;

namespace Vigil.MessageQueue
{
    public interface ICommandQueue
    {
        void QueueCommand(ICommand command, IKeyIdentity key);
    }
}
```

## ICommand Interface

This is just a simple way to identify commands and provide a way for future restrictions and contracts, when I need them.

```csharp
namespace Vigil.Domain
{
    public interface ICommand
    {
    }
}
```
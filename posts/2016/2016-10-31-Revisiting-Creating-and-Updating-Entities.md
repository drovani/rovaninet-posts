---
title: Revisiting Creating and Updating Entities
category: Vigil Journey
treeid: Vigil/tree/df1b703b2fef0de6c2e78b4e57a9c46637dc097e
tags:
- vigil
- cqrs
- eventsourcing
- commandhandler
- eventbus
date: 2016-10-31
---

Just when I thought I was in a good place to carry forward with retrieving a persisted entity and then modifying its values, I started looking intently into the [CQRS Journey](https://msdn.microsoft.com/en-us/library/jj554200.aspx) on MSDN. While reading through their narative, I thought I was really getting to understand how the code was structured. However, once I dug into the source code - well, I was naively mistaken. The notion of having [a Factory](/posts/2016/user-can-create-new-patron/) was entirely flawed, as that role of validating and placing Commands on the Event Bus was better suited in the user interface layer (be it a Web Controller, a console library, or some other set of code). I supposed that the Factory could fall under "some other set of code"; however, for the purposes of demonstrating the bare minimum needed to declare an action completed, the Factory was adding one too many steps. I realized that the unit test need not create a Factory to then pass a Command to the Bus. Instead, the Unit Test should _assume the command was already on the bus_.

If I go back to the notion of "User Can Create New Patron", I have realized that I did it entirely wrong. Since what I wanted to demonstrate was the power and ability of using `interface` design to better focus on what I want to accomplish, the statement I should have been "User Can Command A Patron To Be Created". This provides a better starting place for me to carry that command forward. The `CreatePatronCommand` has now already been created (_ipso facto_, it is in the message queue) and now needs an `ICommandHandler` to turn it into one or more events and to persist the command for historical referencing.


### The New Command Interfaces

With this refocus, I now have an `ICommand` that is handled by an `ICommandHandler` with access to a `ICommandRepository` for persistence needs. The narative now becomes:

A `Command` describes the action that needs to take place. The name of the command (`CreatePatronCommand`) indicates what should be peformed and the data in the command provides the necessary information to fully execute the command.

```csharp
using System;

namespace Vigil.Domain.Messaging
{
    public interface ICommand
    {
        /// <summary>Gets the command identifier.
        /// </summary>
        Guid Id { get; }
    }
}
```

The command, having been put on the bus, is then passed to a `CommandHandler` which creates one or more events, placing them on the event bus, and persists the command to some storage for archival purposes. Otherwise, the command is of no further use and could be discarded.

```csharp
namespace Vigil.Domain.Messaging
{
    public interface ICommandHandler
    {
    }

    public interface ICommandHandler<TCommand> : ICommandHandler
        where TCommand : ICommand
    {
        void Handle(TCommand command);
    }
}
```

In order to persist the command, a `CommandRepository` is passed to the command handler to presist the command. When needed, this could also fetch data if it is required to do some kind of look-up when creating events.

```csharp
namespace Vigil.Domain.Messaging
{
    public interface ICommandRepository
    {
        void Save<TCommand>(TCommand command) where TCommand : ICommand;
    }
}
```
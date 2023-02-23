---
title: Illusions of Queues and Buses
category: Vigil Journey
treeid: Vigil/tree/4bac8bfafb8ed7748b2d1958d98786d7fa07c047
tags:
- vigil
- sql
- commandqueue
- eventbus
date: 2017-01-26
---

There are many solutions on the market (including [free](https://github.com/zeromq/netmq) [open](https://github.com/pardahlman/RawRabbit) [source](https://www.rabbitmq.com/dotnet.html)) for creating both message queues and event buses. However, for example purposes - and as a simple _proof of concept_ - I just wanted a simple way to store the contents of a `Command`, run the `CommandHandler`, receive `Event` objects, call the `EventHandler` actions, and save all of the results. Everything runs synchronously and there are no retry policies, topics, or request/reply patterns. It is a simple, nearly useless set of code - but it serves its very specific purpose.


## SqlCommandQueue

Calling this a "queue" is a complete misnomer, since there is no actual queuing of the commands. All this does is persist the command to the database, call the command handler, and then declare the command as handled.

```csharp
using Microsoft.Extensions.DependencyInjection;
using Newtonsoft.Json;
using System;
using Vigil.Domain.Messaging;

namespace Vigil.Sql
{
    public class SqlCommandQueue : ICommandQueue
    {
        private readonly IServiceProvider _serviceProvider;
        private readonly Func<SqlMessageDbContext> _dbFactory;

        public SqlCommandQueue(IServiceProvider serviceProvider,
            Func<SqlMessageDbContext> dbFactory)
        {
            _serviceProvider = serviceProvider;
            _dbFactory = dbFactory;
        }

        public void Publish<TCommand>(TCommand command) where TCommand : ICommand
        {
            using (SqlMessageDbContext context = _dbFactory())
            {
                var newCmd = new Command()
                {
                    GeneratedBy = command.GeneratedBy,
                    GeneratedOn = command.GeneratedOn,
                    Id = command.Id,
                    SerializedCommand = JsonConvert.SerializeObject(command),
                    CommandType = typeof(TCommand).AssemblyQualifiedName,
                    DispatchedOn = DateTime.UtcNow
                };
                context.Commands.Add(newCmd);
                context.SaveChanges();
            }

            var handler = _serviceProvider
                .GetRequiredService<ICommandHandler<TCommand>>();
            handler.Handle(command);

            using (SqlMessageDbContext context = _dbFactory())
            {
                var cmd = context.Commands.Find(command.Id);
                cmd.HandledOn = DateTime.UtcNow;
                context.SaveChanges();
            }
        }
    }
}
```

A bit of magic happens on lines 28 and 29 — in order to retrieve an arbitrary object for later consumption, the command needs to be serialized. Additionally, to allow it to be deserialized later, the full AQN of the class should be kept handy. The actual work is done in lines 36 and 37, which get the command handler that has already been globally registered, and then calls the `Handle` method. Since nothing happens asynchronously, the queue is fully able to assume that returning control back to it means the command has been handled. Updating the `HandleOn` property closes that loop.

## SqlEventBus

Just as the "Command Queue" isn't a queue, the `SqlEventBus` isn't a bus, or a topic, or any kind of fancy messaging. What does it do? It persists the event to storage (a database), gets and calls all of the event handlers, then marks the persisted event as handled. The code looks quite similar to the `SqlCommandQueue`.

```csharp
using Microsoft.Extensions.DependencyInjection;
using Newtonsoft.Json;
using System;
using Vigil.Domain.Messaging;

namespace Vigil.Sql
{
    public class SqlEventBus : IEventBus
    {
        private readonly IServiceProvider _serviceProvider;
        private readonly Func<SqlMessageDbContext> _dbFactory;


        public SqlEventBus(IServiceProvider serviceProvider,
            Func<SqlMessageDbContext> dbFactory)
        {
            _serviceProvider = serviceProvider;
            _dbFactory = dbFactory;
        }

        public void Publish<TEvent>(TEvent evnt) where TEvent : IEvent
        {
            using (SqlMessageDbContext context = _dbFactory())
            {
                var newEvnt = new Event()
                {
                    GeneratedBy = evnt.GeneratedBy,
                    GeneratedOn = evnt.GeneratedOn,
                    Id = evnt.Id,
                    SourceId = evnt.SourceId,
                    EventType = typeof(TEvent).AssemblyQualifiedName,
                    SerializedEvent = JsonConvert.SerializeObject(evnt),
                    DispatchedOn = DateTime.UtcNow
                };
                context.Events.Add(newEvnt);
                context.SaveChanges();
            }

            var handlers = _serviceProvider.GetServices<IEventHandler<TEvent>>();
            foreach(IEventHandler<TEvent> handler in handlers)
            {
                handler.Handle(evnt);
            }

            using (SqlMessageDbContext context = _dbFactory())
            {
                var handled = context.Events.Find(evnt.Id);
                handled.HandledOn = DateTime.UtcNow;
                context.SaveChanges();
            }
        }
    }
}
```

The unit tests are very simple (so simple that I haven't even created them - and probably won't). It would just be a set of tests that verify it is doing exactly what I've told it to do. Since this code is only a demostration, and not meant for any kind of actual use, I'm not bother to fully unit test it. I doubt that this code will survive past the point where I have a real queue and bus living up in Azure.

Two other pieces that I did add, that may be something that carries into an Azure implementation. I created new `Command` and `Event` objects that are just POCO wrappers for the `Vigil.Domain.Command` and `Vigil.Domain.Event` entities that are being persisted. One little handy thing that I did create, solely for debugging, is a `SqlViewerController` that sets up a _/sql/_ route with a way to view all persisted commands, all persisted events, and the ability to test rehydrating a `Patron`. It has been useful to have when I was testing out the `Vigil.WebApi` project — more on that in another post.
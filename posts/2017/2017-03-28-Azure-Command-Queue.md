---
title: Vigil's Azure Command Queue
category: Vigil Journey
treeid: Vigil/tree/cc860d1411b9c8b27e810437656128021fac5e05
tags:
- azure
- cqrs
- secrets
date: 2017-03-28
---

Command Query Responsibility Segregation ([CQRS](https://martinfowler.com/bliki/CQRS.html)) was not something that I easily understood. Martin Fowler did a thorough job of explaining it in often-linked post from 2011. However, I still did not quite grasp it at the time. I also wasn't working on any project large enough or complex enough that it made sense to use a pattern with that level of depth. When I came back to the concept many years later while working on a significantly more complex project, I could put together all of the puzzle pieces but I still couldn't quite see the whole picture. While researching the topic, I stumbled on a blog post by Cesar de la Torre from Microsoft - [CQRS BUS and Windows Azure technologies](https://blogs.msdn.microsoft.com/cesardelatorre/2012/02/22/cqrs-bus-and-windows-azure-technologies/). His picture was well worth the thousands of words that I had already read.

![CQRS - Basic patterns](/images/cqrs-diagram.jpg)

I am limiting the scope of this step of the project to just creating a Message Queue in Azure, writing a command to it, and triggering an appropriate `Command Handler`. As soon as I started working on the problem, though, I was sidetracked with an important and timely concern — how do I safely store connection information to a remote server? Considering that I work on this project on at least three different machines, I need a highly portable and secure way to store the connection information to my instance of Azure without exposing that connection information to anyone who may look at the code on [GitHub](https://github.com/drovani/Vigil). Now, instead of having three steps to this stage, I have four; maybe more.

## Storing Connection Information

![User Secrets in Context Menu](/images/vs2017-inavord-manage-user-secrets.png)

When this migrates to a production ready environment, I will probably use [Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/key-vault-whatis) or some similar service to manage these types of important secrets. However, for the limited amount of information that I need to keep, I am just going to use the built-in "User Secrets" that comes with Visual Studio 2017. Using the context menu on a Web Application project provides the option to "Manage User Secrets". This pulls up a `secrets.json` file that lives (unencrypted) on the local filesystem. There are plenty of caveats to its use, namely that it provides no security outside of making sure these settings exist entirely separate from your code. In order to store the information I needed, my `secrets.json` file looks similar to this.

```json
{
  "vigil-storage": "vigilstorage",
  "vigil-storage-queue": "commandqueue",
  // sample key - the real one is much longer
  "vigil-storage-key1": "ocJFUuUrYgVppKtO726cC1S..."
}
```

In order to access the user secrets to the configuration builder, a small piece is added to `Startup.cs`.

```csharp
public Startup(IHostingEnvironment env)
{
    var builder = new ConfigurationBuilder()
        .SetBasePath(env.ContentRootPath)
        .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
        .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
        .AddEnvironmentVariables();

    if (env.IsDevelopment())
    {
        builder.AddUserSecrets("6bf18fc4-44a1-4c9b-98ea-22cc92ed1c94");
    }

    Configuration = builder.Build();
}
```

Consuming the secrets is as simple as any other configuration value. Since I only need it right now for the `StorageCredentials` that get injected into the `AzureCommandQueue` constructor, I have the creation of the credentials as an extension method on the `IServiceCollection`. My `StartupExtensions` class can live anywhere; I have decided to put it in the same project that needs it, `Vigil.Azure`.

```csharp
public static class StartupExtensions
{
    public static IServiceCollection AddVigilAzureServices(this IServiceCollection services, IConfigurationRoot configuration)
    {
        services.AddTransient<IEventBus, AzureEventBus>()
                .AddTransient<ICommandQueue, AzureCommandQueue>()
                .AddTransient(srvProvider =>
                    {
                        var storageCredentials = new StorageCredentials(
                            configuration["vigil-storage"],
                            configuration["vigil-storage-key1"]
                        );
                        CloudStorageAccount storageAccount = new CloudStorageAccount(storageCredentials, true);
                        CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();
                        var commandQueue = queueClient.GetQueueReference(configuration["vigil-storage-queue"]);
                        return commandQueue;
                    });

        return services;
    }
}
```

## Create a Storage Account in Azure

This was the easiest step of them all, because there are many wonderful tutorials already created on how to do get started with [Azure Storage](https://docs.microsoft.com/en-us/azure/storage/storage-introduction). It comes down to three simple steps that can all be done in under a minute (once you know your way around).

1. Create a ![icon](/images/azure-resource-group-icon.png) Resource group
1. Create a ![icon](/images/azure-storage-account-icon.png) Storage Account
1. View ![icon](/images/azure-access-key-icon.png) Access keys

The name of the storage account (`vigilstorage`) and the value of one of the keys goes into the `User Secrets`, and _voilà_, the storage account is created. One item that tripped me up was that I expected to need to create the Queue within the portal. As I eventually figured out, that piece is performed exclusively in code.

## Sending a Message to Azure Queue Storage

After many hours of digging through tutorials, hunting down _dotnet core_ versions of the libraries, and trial and error - I finally got to a point where my code compiled and executed and I could test it with a [Postman](https://www.getpostman.com/) query. I was surprised by how little code it ended up requiring. This is the entirety of what the `AzureCommandQueue` class looks like:

```csharp
public class AzureCommandQueue : ICommandQueue
{
    private CloudQueue commandQueue;

    public AzureCommandQueue(CloudQueue commandQueue)
    {
        this.commandQueue = commandQueue;
    }

    public void Publish<TCommand>(TCommand command) where TCommand : ICommand
    {
        PublishAsync(command).Wait();
    }

    public async Task PublishAsync<TCommand>(TCommand command) where TCommand : ICommand
    {
        await commandQueue.CreateIfNotExistsAsync();

        var newCmd = new Command(command)
        {
            DispatchedOn = DateTime.UtcNow
        };
        var message = new CloudQueueMessage(JsonConvert.SerializeObject(newCmd));
        await commandQueue.AddMessageAsync(message);
    }
}
```

## HOLY SHIT I GOT IT TO WORK

![Data in an Azure Queue](/images/azure-queue-data.png)

This was one of those "really cool moments" when it comes together how fantastic my job is and how amazing technology truly is. I just had a tool on my computer simulate a user pushing a non-existant button that sends information to another tool on my computer that is pretending to be a web server which then executed code that I wrote to take that message and securely send it up to the internet and sit on some unknown computer where it will stay and wait for me to call upon it.

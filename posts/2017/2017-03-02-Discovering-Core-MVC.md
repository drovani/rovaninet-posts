---
title: Discovering MVC - The ASP.NET Core MVC Stack
category: Vigil Journey
tags:
- coremvc
date: 2017-03-02
---

Now that the Vigil Project has breached into the API layer, I started to learn all kinds of new information about what takes place before the Controller's Action method is called and what happens after that method returns. The MVC framework, especially while using Visual Studio, make it incredibly easy to forget about all of the heavy lifting that the framework does &ndash; long before my code is ever executed. It is only when I wanted to start changing how pieces worked that I really start to see the true depth and breadth of what Microsoft has created. As I begin to discover new pieces of the framework, I felt that the best way to retain this information is to repeat it; and since no one around me wants to hear about all of this, I'll just type it into a blog post.

## Upon First Introduction

My first introduction to MVC (through various tutorials) gave me the impression that the full stack looks basically like this:

![Initial Pipeline Assumptions](/images/mvcpipeline-initial.svg)

There's a lot of black box magic going on in that diagram. But you know what? It doesn't matter. When all I wanted to do was respond to a simple request, then all I had to do was create a `Controller` and an `Action` and then _magic happened_, and I got a functioning application. Do I have any clue about how the data got to the method? Or how the `Controller` was even instantiated in order for _something_ to have an object to call my method? __Nope__! And that's the point.

## The First Challenge - Explicit Constructor

The Create action for the `BaseController` is simple. It just take an instance of the `CreatePatron` command and publishes is to the `CommandQueue`. This is essentially the same as the unit test that proved that a "[patron can be created](/posts/2016/patron-can-be-created-and-updated-again/)".

```csharp
[HttpPost]
public IActionResult Create([FromBody]CreatePatron command)
{
    if (command == null || !ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }
    else
    {
        CommandQueue.Publish(command);
        return Accepted(Url.Action(nameof(Get),
            new { id = command.PatronId }));
    }
}
```

However, this quickly goes awry the first time I execute the code, because the `CreatePatron` command (as with all `PatronCommand` classes) has no default constructor. MVC _really_ wants a default constructor, but I _really_ want to guarantee that the `GeneratedBy` and `GeneratedOn` values are properly populated. "Surely," though I, "this should be an easy update and I'll be able to move onto something else."

I was wrong.

## What Creates an Instance of My Model

Early in the tutorials I was following, they allude to this magical _Binding Handler_ which takes data from the request and binds it to the model and passes it to the action. I now had an idea of _what_ I was looking for, with the assumption that the Binding Handler is what would create the instance of my model. I also assumed that whatever was doing this work was a result of some default setting.

This was the first time I started really digging into what happens during the `Startup` portion of the application. `Startup` is the default class that is created with any new MVC project, and it is wired into the `WebHostBuilder` during the execution of the `Main` event. I didn't dig into all of the different methods that automatically get generated, because I had a this specific action that I was trying to find.

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    // Add framework services.
    services.AddMvc()
}
```

As a bit of a tangent, this is where I am estatic that .NET Core is open source. For previous frameworks, I would have to scour the web for someone to have already done all of this digging, and decided it was worth it to create a post, and then I would have to manage to find their post, and then hope it was close enough for me to get the answers I needed. Now that everything is open source, I can just go [view the code itself](https://github.com/aspnet/mvc) to find out what happens.

```csharp
    public static class MvcServiceCollectionExtensions
    {
        public static IMvcBuilder AddMvc(this IServiceCollection services)
        {
            if (services == null)
            {
                throw new ArgumentNullException(nameof(services));
            }

            var builder = services.AddMvcCore();

            builder.AddApiExplorer();
            builder.AddAuthorization();

            AddDefaultFrameworkParts(builder.PartManager);

            // Order added affects options setup order

            // Default framework order
            builder.AddFormatterMappings();
            builder.AddViews();
            builder.AddRazorViewEngine();
            builder.AddRazorPages();
            builder.AddCacheTagHelper();

            // +1 order
            builder.AddDataAnnotations(); // +1 order

            // +10 order
            builder.AddJsonFormatters(); // <--- THIS LOOKS PROMISING!!!

            builder.AddCors();

            return new MvcBuilder(builder.Services, builder.PartManager);
        }
```

What does the `AddJsonFormatters()` extension method do? I can go [look that up](https://github.com/aspnet/Mvc/blob/dev/src/Microsoft.AspNetCore.Mvc.Formatters.Json/DependencyInjection/MvcJsonMvcCoreBuilderExtensions.cs). That method calls an internal method that adds Service Descriptor to the service collection to resolve an `IConfigureOptions<MvcOptions>` into a `MvcJsonMvcOptionsSetup`. That [source code](https://github.com/aspnet/Mvc/blob/dev/src/Microsoft.AspNetCore.Mvc.Formatters.Json/Internal/MvcJsonMvcOptionsSetup.cs) for that class indicates that it adds a `JsonInputFormatter` to the list of Input Formatters in the `MvcOptions` object that gets passed to it. Continuing to dive further into the [source code](https://github.com/aspnet/Mvc/blob/dev/src/Microsoft.AspNetCore.Mvc.Formatters.Json/JsonInputFormatter.cs), I finally found the magic line of code!

```csharp
namespace Microsoft.AspNetCore.Mvc.Formatters
{
    public class JsonInputFormatter : TextInputFormatter
    {
        /* snip the constructor */

        public override Task<InputFormatterResult> ReadRequestBodyAsync(
            InputFormatterContext context,
            Encoding encoding)
        {
                    /* snip a bunch of set-up code */
                    try
                    {
                        // HERE IT IS!!!
                        model = jsonSerializer.Deserialize(jsonReader, type);
                    }
                    /* snip clean-up code */
                }
            }
        }
        /* snip the rest of the class */
    }
}
```

## What Do I Know, Now?

![Getting the Input Formatter](/images/mvcpipeline-chooseinputformatter.svg)

During startup, using the `AddMvc` extension method, by default, will add a `JsonInputFormatter` to the list of Input Formatters. When reading the request body, the `ReadRequestBodyAsync` using the JSON serializer's [`Deserialize` method](http://www.newtonsoft.com/json/help/html/M_Newtonsoft_Json_JsonSerializer_Deserialize_1.htm). I don't see anyway to injet something to override just that one line - or even the object creation portion.

Zoomed in - this is the tiny piece that I learned of the whole pipeline. I have no idea if this is to scale or not, but it certainly feels like only a small step.

![Getting the Input Formatter](/images/mvcpipeline-chooseinputformatter.zoom.svg)

## What Do I Need To Do?

1. Inherit from `JsonInputFormatter` and override the `ReadRequestBodyAsync` method.
1. Create a new setup class to inject the new input formatter, implementing `IConfigureOptions<MvcOptions>`.
1. Create a new static class and an extension method on `IMvcBuilder` to add dependency injection for the setup class.
1. Crib unit tests from the Mvc code base.
1. Hope that it all really works.
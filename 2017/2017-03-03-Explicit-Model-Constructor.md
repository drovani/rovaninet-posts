---
title: Using an Explicit Model Constructor with a JsonInputFormatter
category: Vigil Journey
treeid: Vigil/tree/78af44f69d3ae477314d21ec2f76065f2a95267f
tags:
- vigil
- coremvc
- hastobeabetterway
date: 2017-03-03
---

I may have gone down a long and twisted rabbit hole trying to figure out this problem, but I learned a lot about how model binding along the way, so I consider the whole experiment a productive use of my time, even if I end up ripping it all out in a few weeks. However, since I thought this would be a good idea, I figure others might find a good use for this knowledge. Thus, this is how I am using an explicity constructor when the input is in a `JSON` format and parameter is bound using the request body.


```csharp
[HttpPost]
public IActionResult Create([FromBody]CreatePatron command)
```

An early lesson when writing Actions is that, in Core MVC, the `[FromBody]` attribute is required to use the `JsonInputFormatter`. Setting a new convention for controllers is possible ([instructions on how to](http://www.dotnetcurry.com/aspnet-mvc/1149/convert-aspnet-webapi2-aspnet5-mvc6)), but for now I am going to stick with the tedium of remembering to apply it to every parameter.

The purpose of this is that I want the `GeneratedBy` and the `GeneratedOn` parameters to be required for every instance of a `Command`. This ensures that it won't be set by the user (either the end user or another user in a consuming change) since the properties' set methods are `protected`.

```csharp
using System;

namespace Vigil.Domain.Messaging
{
    public abstract class Command : KeyIdentity, ICommand
    {
        public string GeneratedBy { get; protected set; }
        public DateTime GeneratedOn { get; protected set; }

        protected Command(string generatedBy, DateTime generatedOnUtc)
        {
            if (string.IsNullOrEmpty(generatedBy)) throw new ArgumentNullException(nameof(generatedBy));
            if (generatedOnUtc == default(DateTime)) throw new ArgumentException($"{nameof(generatedOnUtc)} requires a non-default value.", nameof(generatedOnUtc));
            if (generatedOnUtc.Kind != DateTimeKind.Utc) throw new ArgumentException($"{nameof(generatedOnUtc)} must be DateTimeKind.UTC.", nameof(generatedOnUtc));

            GeneratedBy = generatedBy;
            GeneratedOn = generatedOnUtc;
        }
    }
}
```

This poses a problem with using the `CreatePatron` command as a parameter. The default [`JsonInputFormatter` class](https://github.com/aspnet/Mvc/blob/dev/src/Microsoft.AspNetCore.Mvc.Formatters.Json/JsonInputFormatter.cs) uses the [`JsonSerializer.Deserialize(JsonTextReader, Type)` method](https://github.com/aspnet/Mvc/blob/dev/src/Microsoft.AspNetCore.Mvc.Formatters.Json/JsonInputFormatter.cs#L149) to create the instance of the model. This method uses the parameterless constructor, which my objects to not have. To solve this problem, I inheritted from the `JsonInputFormatter` (keeping as much functionality as I could) and proceeded from there.

## CommandInputFormatter Constructor

All of the fields that are passed to the `JsonInputFormatter` constructor are stored as private readonly fields. Therefore, I had to create my own private readonly copies of them to use in the methods that I needed to write.

```csharp
public class CommandInputFormatter : JsonInputFormatter
{
    private readonly IArrayPool<char> _charPool;
    private readonly ILogger _logger;
    private readonly ObjectPoolProvider _objectPoolProvider;

    public CommandInputFormatter(ILogger logger
        , JsonSerializerSettings serializerSettings
        , ArrayPool<char> charPool
        , ObjectPoolProvider objectPoolProvider)
        : base(logger, serializerSettings, charPool, objectPoolProvider)
    {
        if (charPool == null)
        {
            throw new ArgumentNullException(nameof(charPool));
        }
        _charPool = new JsonArrayPool<char>(charPool);

        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        _objectPoolProvider = objectPoolProvider ?? throw new ArgumentNullException(nameof(objectPoolProvider));
    }
}
```

## CommandInputFormatter.CanReadyType

Since I wanted this formatter to only work for these specific classes, I extended the functionality of the `CanReadType` method to also filter for just models that implement the `Command` abstract base class.

```csharp
protected override bool CanReadType(Type type)
{
    return base.CanReadType(type) && typeof(Command).IsAssignableFrom(type);
}
```

## CommandInputFormatter.ReadRequestBodyAsync

The simplest way for me to change the constructor that `JsonInputFormatter` uses to generate the model would have been if they had isolated that small piece of the `ReadRequestBodyAsync` method into its own protected virtual method. However, since they didn't, I have to override the entire thing. Because ASP.NET Core (including MVC) is all open source, I was able to copy the source code from the original and only replace the parts that I needed to.

```csharp
public override Task<InputFormatterResult> ReadRequestBodyAsync(InputFormatterContext context, Encoding encoding)
{
    if (context == null)
    {
        throw new ArgumentNullException(nameof(context));
    }
    if (encoding == null)
    {
        throw new ArgumentNullException(nameof(encoding));
    }


    using (var streamReader = context.ReaderFactory(context.HttpContext.Request.Body, encoding))
    {
        using (var jsonReader = new JsonTextReader(streamReader))
        {
            jsonReader.ArrayPool = _charPool;
            jsonReader.CloseInput = false;

            var successful = true;
            EventHandler<Newtonsoft.Json.Serialization.ErrorEventArgs> errorHandler = (sender, eventArgs) =>
            {
                successful = false;

                var exception = eventArgs.ErrorContext.Error;

                // Handle path combinations such as "" + "Property", "Parent" + "Property", or "Parent" + "[12]".
                var key = eventArgs.ErrorContext.Path;
                if (!string.IsNullOrEmpty(context.ModelName))
                {
                    if (string.IsNullOrEmpty(eventArgs.ErrorContext.Path))
                    {
                        key = context.ModelName;
                    }
                    else if (eventArgs.ErrorContext.Path[0] == '[')
                    {
                        key = context.ModelName + eventArgs.ErrorContext.Path;
                    }
                    else
                    {
                        key = context.ModelName + "." + eventArgs.ErrorContext.Path;
                    }
                }

                var metadata = GetPathMetadata(context.Metadata, eventArgs.ErrorContext.Path);
                context.ModelState.TryAddModelError(key, eventArgs.ErrorContext.Error, metadata);

                _logger.LogDebug(1, eventArgs.ErrorContext.Error, "Command Input Formatter threw an exception");

                // Error must always be marked as handled
                // Failure to do so can cause the exception to be rethrown at every recursive level and
                // overflow the stack for x64 CLR processes
                eventArgs.ErrorContext.Handled = true;
            };

            object model = GetModel(context);
            var type = context.ModelType;
            var jsonSerializer = CreateJsonSerializer();
            jsonSerializer.Error += errorHandler;
            try
            {
                jsonSerializer.Populate(jsonReader, model);
            }
            finally
            {
                // Clean up the error handler since CreateJsonSerializer() pools instances.
                jsonSerializer.Error -= errorHandler;
                ReleaseJsonSerializer(jsonSerializer);
            }

            if (successful)
            {
                return InputFormatterResult.SuccessAsync(model);
            }

            return InputFormatterResult.FailureAsync();
        }
    }
}
```

There are only two lines that differ between the formatters; I was surprised by how little I had to change.

| `JsonInputFormatter`                                | `CommandInputFormatter`                   |
|-----------------------------------------------------|-------------------------------------------|
|object model;                                        |object model = GetModel(context);          |
|model = jsonSerializer.Deserialize(jsonReader, type);|jsonSerializer.Populate(jsonReader, model);|

## CommandInputFormatter.GetModel

Of course, there was a little more that I had to add, but this piece is very specific to my application. This is the block of code that creates the appropriate `Command` object using the explicit constructor.

```csharp
private static readonly Dictionary<Type, Func<string, DateTime, Command>> _commandModelCreators = new Dictionary<Type, Func<string, DateTime, Command>>();
protected virtual object GetModel(InputFormatterContext context)
{
    if (!_commandModelCreators.ContainsKey(context.ModelType))
    {
        var generatedBy = Expression.Parameter(typeof(string), nameof(Command.GeneratedBy));
        var generatedOn = Expression.Parameter(typeof(DateTime), nameof(Command.GeneratedOn));
        var constructor = context.ModelType.GetConstructor(new Type[] { typeof(string), typeof(DateTime) });
        var newExpr = Expression.New(constructor, generatedBy, generatedOn);
        var lambda = Expression.Lambda<Func<string, DateTime, Command>>(newExpr, generatedBy, generatedOn);

        _commandModelCreators.Add(context.ModelType, lambda.Compile());
    }
    // @TODO Find a better way to get the current user's name
    return _commandModelCreators[context.ModelType](context?.HttpContext?.User?.Identity?.Name ?? "Anonymous User", DateTime.UtcNow);
}
```

I save the compiled expression in the formatter. There might be a better way to cache the results, but I can worry about that optimization later. I would also like to have a better way to pass in the identity of the user generating this command. This works for my testing purposes, but I need a better way to do this in the future. Additionally, it would be better if I could inject the `DateTime.UtcNow` value to be able to explicitly set it and test again in unit tests.

## Include the CommandInputFormatter

I was not expecting this piece to turn into a three step process. In order to add an an input formatter with parameters, it needs to be configured through an `IConfigureOptions<MvcOptions>`, which is added via a `ServiceDescriptor` to the `Services` collection on the `IMvcBuilder` in the `Startup` class. I learned all this by searching through the Github repository. Once I found the path through it all, I just duplicated most of it, adapting it to my use case.

The `IConfigureOptions<MvcOptions>` is easy to implement. The difficulty was in finding what dependency injections were available for the constructor.

```csharp
public class VigilMvcOptionsSetup : IConfigureOptions<MvcOptions>
{
    private readonly ILoggerFactory _loggerFactory;
    private readonly JsonSerializerSettings _jsonSerializerSettings;
    private readonly ArrayPool<char> _charPool;
    private readonly ObjectPoolProvider _objectPoolProvider;

    public VigilMvcOptionsSetup(
        ILoggerFactory loggerFactory,
        IOptions<MvcJsonOptions> jsonOptions,
        ArrayPool<char> charPool,
        ObjectPoolProvider objectPoolProvider)
    {
        if (jsonOptions == null)
        {
            throw new ArgumentNullException(nameof(jsonOptions));
        }

        _loggerFactory = loggerFactory ?? throw new ArgumentNullException(nameof(loggerFactory));
        _jsonSerializerSettings = jsonOptions.Value.SerializerSettings;
        _charPool = charPool ?? throw new ArgumentNullException(nameof(charPool));
        _objectPoolProvider = objectPoolProvider ?? throw new ArgumentNullException(nameof(objectPoolProvider));
    }

    public void Configure(MvcOptions options)
    {
        options.InputFormatters.Insert(0, new CommandInputFormatter(
            _loggerFactory.CreateLogger<CommandInputFormatter>(),
            _jsonSerializerSettings,
            _charPool,
            _objectPoolProvider
        ));
    }
}
```

From here, I could have easily just injected the `VigilMvcOptionsSetup` during startup config. I suspect that there will be more customization that I will want to attach to the `IMvcBuilder`, so chose to follow the lead from the MVC developers and create an extension method.

```csharp
public static class MvcBuilderExtensions
{
    public static IMvcBuilder AddCommandFormatter(this IMvcBuilder builder)
    {
        if (builder == null)
        {
            throw new ArgumentNullException(nameof(builder));
        }
        ServiceDescriptor descriptor = ServiceDescriptor.Transient<IConfigureOptions<MvcOptions>, VigilMvcOptionsSetup>();
        builder.Services.TryAddEnumerable(descriptor);
        return builder;
    }
}
```

The final step, to wire it all together, is to add a call to the extension method in the `Startup.ConfigureServices(IServiceCollection)` method. It is important that my extension method comes after the `AddMvc` method because I inject the `CommandInputFormatter` to the beginning of the `InputFormatters` collection. Reversing the order would put my input formatter behind all of the defaults, and the `JsonInputFormatter` would grab all of the requests before my input formatter was ever checked.

```csharp
// Add framework services.
services.AddMvc()
    .AddCommandFormatter();
```

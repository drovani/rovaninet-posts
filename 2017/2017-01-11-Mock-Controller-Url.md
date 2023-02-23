---
title: Mock Controller's User and Url Properties in ASP.NET Core MVC
category: Vigil Journey
treeid: Vigil/tree/96ccfc6f1e4326a7f2809976ee88c4388a624804
tags:
- moq
- mock
- unittests
- controller
date: 2017-01-11
---

I have come to the point where I am building out the initial proof-of-concept for the WebAPI portion of this project. This gives me a place to continue testing out features, and see if how I envisioned things would work can actually work. One of the first parts of unit testing the controller was to mock any of the properties that I need to use that the web server would ordinarily wire up. In a `Controller` that primarily entails the `ControllerContext`, which would also wire the `Controller.User`, `Controller.Url`, and `Controller.HttpContext` properties. So how do I do that?


### Mocking Controller.Url

I use Moq as my mocking framework, though this could be done with any library. The `Url` property returns an `IUrlHelper`, which makes it easy to mock.

```csharp
namespace Microsoft.AspNetCore.Mvc
{
    public interface IUrlHelper
    {
        ActionContext ActionContext { get; }

        string Action(UrlActionContext actionContext);
        string Content(string contentPath);
        bool IsLocalUrl(string url);
        string Link(string routeName, object values);
        string RouteUrl(UrlRouteContext routeContext);
    }
}
```

The `Content(string)`, `IsLocalUrl(string)`, and `Link(string, object)` are all trivial to overload. The piece that I stumbled across was when I wanted to mock `Url.Action(string, object)`, which is an extension method in the `UrlHelperExtensions` class. Moq is unable to mock extension methods. However, because `AspNetCore` is open source, I could happily go [look at the source](https://github.com/aspnet/Mvc/blob/master/src/Microsoft.AspNetCore.Mvc.Core/UrlHelperExtensions.cs) and see what the extension method was doing behind the scenes. All of the various extension methods just pass controll to the last (and longest) extension method, which calls `Url.Action(UrlActionContext)`. They all pass a `null` for values not collected.

```csharp
public static string Action(
    this IUrlHelper helper,
    string action,
    string controller,
    object values,
    string protocol,
    string host,
    string fragment)
{
    if (helper == null)
    {
        throw new ArgumentNullException(nameof(helper));
    }

    return helper.Action(new UrlActionContext()
    {
        Action = action,
        Controller = controller,
        Host = host,
        Values = values,
        Protocol = protocol,
        Fragment = fragment
    });
}

```

This all means that in order to "mock" the extension methods, I just need to provide a mock for the `IUrlHelper.Action(UrlActionContext)` method.

```csharp
[Fact]
public void Create_With_ValidCommand_IsPublished_AndReturnsAcceptedResult_WithLocation()
{
    CreatePatron cmd = new CreatePatron("Patron Web Test", TestHelper.Now)
    {
        DisplayName = "Patron Display Name",
        IsAnonymous = false,
        PatronType = "Test Patron"
    };

    var mockUrlHelper = new Mock<IUrlHelper>(MockBehavior.Strict);
    Expression<Func<IUrlHelper, string>> urlSetup
        = url => url.Action(It.Is<UrlActionContext>(uac => uac.Action == "Get" && GetId(uac.Values) != cmd.Id));
    mockUrlHelper.Setup(urlSetup).Returns("a/mock/url/for/testing").Verifiable();

    var controller = new PatronController()
    {
        Url = mockUrlHelper.Object
    };

    AcceptedResult result = controller.Create(cmd) as AcceptedResult;

    Assert.NotNull(result);
    mockUrlHelper.Verify(urlSetup, Times.Once());
    Assert.Equal("a/mock/url/for/testing", result.Location);
}

private Guid? GetId(object values)
{
    return values?.GetType().GetProperty("id")?.GetValue(values, null) as Guid?;
}
```

On my initial attempt, I was having the Url mock return an actual Url. However, I realized that since the string that returns from this shouldn't have any effect other than being a string, it _should_ be some meaningless value. This code is not attempting to mock the routing or url creation based on the values sent to the `IUrlHelper`. All this unit test is doing, and all it _should be doning_, is validating that the controller is asking for a string that comes from passing in the "Get" action and an object with a property of "id" that is a `Guid`.

Mocking the `UrlHelperExtensions.RouteUrl(this IUrlHelper, ...)` extension methods is done the same way, but by mocking the `IUrlHelper.RouteUrl(UrlRouteContext)` method. The `RouteUrl` extension methods also just pass along their values to a new instance of `UrlRouteContext` and then call the interface's method.

## Mocking Controller.User.Identity.Name

Mocking the User property on a Controller can be a little more difficult to figure out, because the `ControllerBase.User` property is read-only; `User` gets its value from the `HttpContext.User` property. The `Request` and `Response` properties on the `ControllerBase` also come from the `HttpContext`. Fortuitously, `ControllerBase.HttpContext` is settable and `HttpContext` is an abstract class. Therefore, a simple stub of the `User` property on a mock of the `HttpContext` takes care of whatever needs there are for accessing the claims information.

Since the most common usage for this is that I am trying to access the user's name, I created a quick private method that goes inside of my test class. Moq takes care of walking the expression tree and creating the necessary mocks of the `ClaimsPrincipal` and the `IIdentity` properties.

```csharp
private void SetupUser(Controller controller, string username)
{
    var mockContext = new Mock<HttpContext>(MockBehavior.Strict);
    mockContext.SetupGet(hc => hc.User.Identity.Name).Returns(username);
    controller.ControllerContext = new ControllerContext()
    {
        HttpContext = mockContext.Object
    };
}
```
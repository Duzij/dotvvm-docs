# Custom presenters

In most applications, there are some situations where you need to generate some XML, JSON or other type of a content (RSS feed etc.), or just compose the HTTP response yourself. 

In both ASP.NET Core and OWIN, you can write your own middleware which handles the communication, or you can use another framework for that (e.g. ASP.NET Web API for REST interfaces).

However, sometimes you need to integrate with various DotVVM services (e.g. file uploads, routing), or you just don't want to introduce additional libraries or frameworks in your project.

In such case, you can use a custom presenter. It is a lightweight alternative to a middleware which lets you to handle the HTTP request yourself, and flawlessly integrates with DotVVM routing.

## Define a custom presenter

To create a custom presenter, you need to create a class that implements the `IDotvvmPresenter` interface. In its `ProcessRequest` method, you can handle the request yourself.

The main advantage of custom presenter over the middleware is that you can use DotVVM routes, use route parameters, action filters and you also have the
 [Request context](~/pages/concepts/viewmodels/request-context) object.

# [ASP.NET Core](#tab/aspnetcore)

```CSHARP
public class RssPresenter : IDotvvmPresenter
{
    public Task ProcessRequest(IDotvvmRequestContext context)
    {
        context.GetAspNetCoreContext().Response.ContentType = "application/rss+xml";

        // load data
        var articles = ...;
        
        // generate RSS feed
        var items = articles
            .Select(a =>
            {
                var item = new SyndicationItem(a.Title, a.Description, a.Url)
                {
                    PublishDate = a.BeginDate
                };
                item.Authors.Add(new SyndicationPerson() { Name = a.AuthorName });
                return item;
            });

        // create feed
        var feed = new SyndicationFeed("Sample RSS Feed", "DotVVM Sample", new Uri("https://www.dotvvm.com"), items);

        // write the XML to the output
        var writer = XmlWriter.Create(context.GetAspNetCoreContext().Response.Body, new XmlWriterSettings() { Indent = true });
        feed.SaveAsRss20(writer);
        writer.Flush();

        return Task.FromResult(0);
    }
}
```

# [OWIN](#tab/owin)

```CSHARP
public class RssPresenter : IDotvvmPresenter
{
    public Task ProcessRequest(IDotvvmRequestContext context)
    {
        context.GetOwinContext().Response.ContentType = "application/rss+xml";

        // load data
        var articles = ...;
        
        // generate RSS feed
        var items = articles
            .Select(a =>
            {
                var item = new SyndicationItem(a.Title, a.Description, a.Url)
                {
                    PublishDate = a.BeginDate
                };
                item.Authors.Add(new SyndicationPerson() { Name = a.AuthorName });
                return item;
            });

        // create feed
        var feed = new SyndicationFeed("Sample RSS Feed", "DotVVM Sample", new Uri("https://www.dotvvm.com"), items);

        // write the XML to the output
        var writer = XmlWriter.Create(context.GetOwinContext().Response.Body, new XmlWriterSettings() { Indent = true });
        feed.SaveAsRss20(writer);
        writer.Flush();

        return Task.FromResult(0);
    }
}
```

***

## Register presenter in route table

Finally, you need to register the presenter in the `DotvvmStartup.cs` file:

```CSHARP
config.RouteTable.Add("t", "u", typeof(RssPresenter));
```

The `RssPresenter` must be registered in the `IServiceCollection`. See [Dependency injection](~/pages/concepts/configuration/dependency-injection/overview) for more information.

Alternatively, you can register your own factory method that will be used to create the instance of the presenter. In that case, the presented doesn't need to be registered in `IServiceCollection`:

```CSHARP
config.RouteTable.Add("t", "u", serviceProvider => new RssPresenter());
```

## Action filters on presenters

You can apply [Action filters](~/pages/concepts/viewmodels/filters/action-filters) or the [Authorize attribute](~/pages/concepts/security/authentication-and-authorization/overview) on the presenter class.

## See also

* [Parameters](~/pages/concepts/routing/parameters)
* [Route redirection](~/pages/concepts/routing/route-redirection)
* [Auto-discover routes](~/pages/concepts/routing/auto-discover-routes)
* [Dependency injection](~/pages/concepts/configuration/dependency-injection/overview)
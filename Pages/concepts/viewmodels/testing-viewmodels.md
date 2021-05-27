# Testing viewmodels

One of the benefits of using the MVVM patterns is that the viewmodels are testable. Because the viewmodel doesn't have any dependencies on the user interface components, it can be tested without the need to launch the browser.

You can just create an instance of the viewmodel class, set its property values, call a method, and verify that the results are correct.

```CSHARP
[TestMethod]
public void NormalTest()
{
    var viewModel = new SampleViewModel()
    {
        FirstName = "Humphrey",
        LastName = "Appleby"
    };

    viewModel.GenerateUserName();
    Assert.AreEqual("applebyh", viewModel.GeneratedUserName);
}
```

## Mocking the request context

**DotVVM** injects the `IDotvvmRequestContext` object in the viewmodels. If your viewmodel uses some features of this object, you need to mock this object to provide all services to the tested method.

DotVVM contains a prepared mock class `TestDotvvmRequestContext`. You can set all properties you need (`Configuration`, `Route`, `Parameters`, `Query`, `HttpContext`, `ModelState`, `ResourceManager` etc., but they are all optional).

In the test, you may need to verify e.g. whether the viewmodel method has redirected to some URL, or whether it returned a file, failed on the model validation etc. 

## Testing redirects and validation failures

In DotVVM, the methods `InterruptRequest`, `RedirectTo*`, `RedirectPermanentTo*`, `FailOnInvalidModelState` and `ReturnFile` throw the `DotvvmInterruptRequestExecutionException`. This allows to interrupt the execution of the HTTP request and pass it to the following middleware.

The `DotvvmInterruptRequestExecutionException` exception includes a property indication the reason why the request was interrupted and a `CustomData` parameter which holds e.g. the URL where the viewmodel action tried to redirect.

Here you can see a test case that verifies the viewmodel has built the proper route:

```CSHARP
[TestMethod]
public void NormalTest()
{
    var configuration = DotvvmConfiguration.CreateDefault();
    configuration.Routes.Add("ArticleDetail", "article/{Id}/{Title}", "article.dothtml", null);
    
    var viewModel = new SampleViewModel()
    {
        Context = new TestDotvvmRequestContext()
        {
            Configuration = configuration,
            ApplicationHostPath = "myApp"      // This tells the DotVVM that the application runs in a virtual directory, e.g. on the URL "http://somedomain.com/myApp/"
        }
    };
    
    var ex = Assert.ThrowsException<DotvvmInterruptRequestException>(() => 
    {
        var article = new ArticleDTO() { Id = 1, Title = "DotVVM is the best framework" };
        viewModel.RedirectToArticle(article);
    });
    
    Assert.AreEqual(InterruptReason.Redirect, ex.InterruptReason);
    Assert.AreEqual("/myApp/article/1/dotvvm-is-the-best-framework", ex.CustomData);
}
```

## See also

* [Viewmodels overview](overview)
* [Dependency injection](~/pages/concepts/configuration/dependency-injection/overview)

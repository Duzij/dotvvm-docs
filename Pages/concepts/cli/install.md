# Install the CLI

**DotVVM Command-Line tool** can be used for several purposes:

* Adding [pages](~/pages/concepts/dothtml-markup/overview), [master pages](~/pages/concepts/layout/master-pages), or [markup controls](~/pages/concepts/control-development/markup-controls) in the project
* Generating [REST API bindings](~/pages/concepts/respond-to-user-actions/rest-api-bindings/overview)

In the future, we plan to add additional features, like generating Selenium page objects for DotHTML pages, and more.

## Install the CLI tool

To install DotVVM CLI globally, run the following command in the terminal:

```
dotnet tool install -g DotVVM.CommandLine
```

## Remove the DotNetCliToolReference

If you've been using ASP.NET Core with .NET Core 2.2 or lower, you'll probably have the following entry in the `.csproj` file:

```
  <ItemGroup>
    <DotNetCliToolReference Include="DotVVM.CommandLine" Version="2.0.0" />
  </ItemGroup>
```

This can be safely removed, as the `DotNetCliToolReference` was deprecated, and installing command-line tools via `dotnet tool install` is now the preferred way.

## See also

* [Create pages and controls](create-pages-and-controls)
* [Generate REST API clients](generate-rest-api-clients)

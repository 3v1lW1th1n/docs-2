---
slug: openapi
title: OpenAPI
---

[OpenAPI](http://swagger.io/) is a specification and complete framework implementation for describing, producing, consuming, and visualizing RESTful web services. ServiceStack implements the 
[OpenAPI Spec](https://github.com/swagger-api/swagger-spec/blob/master/versions/2.0.md) back-end and embeds the Swagger UI front-end in a separate plugin which is available under [OpenAPI NuGet package](http://nuget.org/packages/ServiceStack.Api.OpenApi/):

    PM> Install-Package ServiceStack.Api.OpenApi

## Installation

You can enable Open API by registering the `OpenApiFeature` plugin in AppHost with:

```csharp
public override void Configure(Container container)
{
    ...
    Plugins.Add(new OpenApiFeature());

    // Uncomment CORS feature if it needs to be accessible from external sites 
    // Plugins.Add(new CorsFeature()); 
    ...
}
```

Then you will be able to view the Swagger UI from `/swagger-ui/`. A link to **Swagger UI** will also be available from your `/metadata` [Metadata Page](/metadata-page).

## OpenAPI Attributes

Each route could have a separate summary and description. You can set it with `Route` attribute:

```csharp
[Route("/hello", Summary = @"Default hello service.", 
    Notes = "Longer description for hello service.")]
```

You can set specific description for each HTTP method like shown below:

```csharp
[Route("/hello/{Name}", "GET", Summary="Says 'Hello' to provided Name", 
    Notes = "Longer description of the GET method which says 'Hello'")]
[Route("/hello/{Name}", "POST", Summary="Says 'Hello' to provided Name", 
    Notes = "Longer description of the POST method which says 'Hello'")]
```

You can further document your services in the OpenAPI with the new `[Api]` and `[ApiMember]` annotation attributes, e,g: Here's an example of a fully documented service:

```csharp
[Api("Service Description")]
[ApiResponse(HttpStatusCode.BadRequest, "Your request was not understood")]
[ApiResponse(HttpStatusCode.InternalServerError, "Oops, something broke")]
[Route("/swagger/{Name}", "GET", Summary = "GET Summary", Notes = "Notes")]
[Route("/swagger/{Name}", "POST", Summary = "POST Summary", Notes="Notes")]
public class MyRequestDto
{
    [ApiMember(Name="Name", Description = "Name Description",
        ParameterType = "path", DataType = "string", IsRequired = true)]
    [ApiAllowableValues("Name", typeof(Color))] //Enum
    public string Name { get; set; }
}
```

Please note, that if you used `ApiMember.DataType` for annotating `SwaggerFeature` then you need to change the types to OpenAPI type. For example, annotation of 
```csharp
[ApiMember(DataType="int")]
```
need to be changed to 
```csharp
[ApiMember(DataType="numeric" DataFormat="int32")]
```

Here is the table for type migration

| Swagger Type (DataType) | OpenAPI Type (DataType) | OpenAPI Format (DataFormat) |
|-------------------------|-------------------------|-----------------------------|
| Array                   | array                   |                             |
| boolean                 | boolean                 |                             |
| byte                    | integer                 | int                         |
| Date                    | string                  | date                        |
|                         | string                  | date-time                   |
| double                  | number                  | double                      |
| float                   | number                  | float                       |
| int                     | integer                 | int32                       |
| long                    | integer                 | int64                       |
| string                  | string                  |                             |

You can Exclude **properties** from being listed in OpenAPI with:

```csharp
[IgnoreDataMember]
```

Exclude **properties** from being listed in OpenAPI Schema Body with:

```csharp
[ApiMember(ExcludeInSchema=true)]
```
### Exclude Services from Metadata Pages

To exclude entire Services from showing up in OpenAPI or any other Metadata Services (i.e. Metadata Pages, Postman, NativeTypes, etc), annotate **Request DTO's** with:

```csharp
[Exclude(Feature.Metadata)]
public class MyRequestDto { ... }
```

### Swagger UI Route Summaries

The Swagger UI groups multiple routes under a single top-level route that covers multiple different 
services sharing the top-level route which can be specified using the `RouteSummary` dictionary of 
the `OpenApiFeature` plugin, e.g: 

```csharp
Plugins.Add(new OpenApiFeature {
    RouteSummary = {
        { "/top-level-path", "Route Summary" }
    }
});
```

### OpenAPI operation filters

You can override operation or parameter definitions by specifying the appropriate filter in plugin configuration:

```csharp
Plugins.Add(new OpenApiFeature
{
    OperationFilter = (verb, operation) => operation.Tags.Add("all operations")
});
```

Available configuration options:

```
ApiDeclarationFilter - allows to modify final result of returned OpenAPI json
OperationFilter - allows to modify operations
SchemaFilter - allows to modify OpenAPI schema for user types
SchemaPropertyFilter - allows to modify propery declarations in OpenAPI schema
```

### Properties naming conventions

You can control naming conventions of generated properties by following configuration options:

```
UseCamelCaseSchemaPropertyNames - generate camel case property names
UseLowercaseUnderscoreSchemaPropertyNames - generate underscored lower cased property names (to enable this feature UseCamelCaseModelPropertyNames must also be set) 
```

Example:

```csharp
Plugins.Add(new OpenApiFeature
{
    UseCamelCaseSchemaPropertyNames = true,
    UseLowercaseUnderscoreSchemaPropertyNames = true
});
```

### Change default Verbs

If left unspecified, the `[Route]` attribute allows Services to be called from any HTTP Verb which by default 
are listed in the Open API specification under the most popular HTTP Verbs, namely `GET`, `POST`, `PUT` and `DELETE`.

This can be modified with `AnyRouteVerbs` which will let you specify which Verbs should be generated 
for **ANY** Routes with unspecified verbs, e.g. we can restrict it to only emit routes for `GET` and `POST` Verbs with:

```csharp
Plugins.Add(new OpenApiFeature
{
    AnyRouteVerbs =  new List<string> { HttpMethods.Get, HttpMethods.Post }
});
```

### Miscellaneous configuration options

```
DisableAutoDtoInBodyParam - disables adding `body` parameter for request DTO to operations
LogoUrl - url of the logo image for Swagger UI
```

Example:

```csharp
Plugins.Add(new OpenApiFeature
{
    DisableAutoDtoInBodyParam = true
});
```

## Virtual File System

The docs on the Virtual File System shows how to override embedded resources:

### Overriding OpenAPI Embedded Resources

ServiceStack's [Virtual File System](/virtual-file-system) supports multiple file source locations where you can override OpenAPI's embedded files by including your own custom files in the same location as the existing embedded files. This lets you replace built-in ServiceStack embedded resources with your own by simply copying the [/swagger-ui](https://github.com/ServiceStack/ServiceStack/tree/master/src/ServiceStack.Api.OpenApi/swagger-ui) files you want to customize and placing them in your Website Directory at:

```
/swagger-ui
  /css
  /images
  /lib
  index.html
```

### Basic Auth in OpenAPI

Users can call protected Services using the Username and Password fields in Swagger UI. 
Swagger UI sends these credentials with every API request using HTTP Basic Auth, 
which can be enabled in your AppHost with:

```csharp
Plugins.Add(new AuthFeature(...,
      new IAuthProvider[] { 
        new BasicAuthProvider(), //Allow Sign-ins with HTTP Basic Auth
      }));
```

To login, you need to click "Authorize" button.

![](../images/openapi/1-swaggerui-authorize.png?raw=true)

And then enter username and password.

![](../images/openapi/2-swaggerui-password.png?raw=true)

Also you can click "Try it out" button on services, which requires authentication and browser will prompt a window with user/password field for entering basic auth credentials.

Alternatively users can login outside of Swagger UI, to access protected Services in Swagger UI.


## Generating Autorest client

You can use OpenAPI plugin to automatically generate client using [Autorest](https://guthub.com/Azure/Autorest). To do it, you need to install autorest first

    npm install -g autorest

After installing open powershell console and download api specification json from you solution.

    iwr http://your.domain/openapi -o openapi.json

Then generate client source code.

    autorest --latest-release -Input openapi.json -CodeGenerator CSharp -OutputDirectory AutorestClient -Namespace AutorestClient

It will generate directory with model types and REST operations, accessible throught the client. To use it you can write following code:

```csharp
using (var client = new SampleProjectAutorestClient("http://localhost:20000"))
{
    var dto = new SampleDto { /* .... */ }; 
    var result = client.SampleOperation.Post(body: dto);

    // process result
}
```

### Known issues

Autorest generated clients do not support `application/octet-stream` MIME type, which is used when service returns `byte[]` array. There is an [issue](https://github.com/Azure/autorest/issues/1932) you can track.

## Publish Azure Management API

Login to [Azure Portal](https://portal.azure.com) and search for `API management service`.

![](../images/azure-api-management/1-search.png?raw=true)

Choose `API management service`. In opened window click `Add` button.

![](../images/azure-api-management/2-add.png?raw=true)

Fill the creation form. Put your own values in `Name`, `Resource Group`, `Organization name` and `Administrator email`. When creation form will be ready, click `Create` button.

![](../images/azure-api-management/3-create.png?raw=true)

Wait while Management API will be activated. It can take more than forty minutes. When it ready click on created API management resource.

![](../images/azure-api-management/4-activating.png?raw=true)

In opened window click `APIs - PREVIEW` menu item on the left pane.

![](../images/azure-api-management/5-publisher-portal.png?raw=true)

Choose `OpenAPI specification` in `Add API` section.

![](../images/azure-api-management/6-add-api.png?raw=true)

Fill the url with location of you services, ended with `/openapi` or just click `Upload` button and upload OpenAPI json definition, which is available at `/openapi` path of your services.

![](../images/azure-api-management/7-create-api.png?raw=true)

After successfull import you should see list of available operations for your services

![](../images/azure-api-management/8-created.png?raw=true)

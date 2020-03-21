---
title: Custom Endpoints, System.Text.Json and IConfiguration
date: 2020-03-21
summary: I often include a custom route to my services to expose what the config looks like. Here's an update version using the new EndpointRouting and System.text.Json for serialization
tags: [dotnet]
---

Recently I've been doing a fair bit with "microservices". I will leave the discussion of what that term actually means &lt;sigh&gt; I'll leave that to those that would rather discuss dogma than getting sh*t done.

In practical terms there are a lot of small processes, maybe with an api, maybe an event consumer/producer, whatever...
A practical problem is that, with all these processes (maybe running as containers in a k8s cluster) it can be difficult to confirm if they are configured correctly. You may be **sure** that one of these is setup right. But is can be very reasuring to see what the process sees rather than making assumptions.

So, I will generally have some version of an http endpoint that just displays the contents of the `IConfiguration` as it has been loaded by the process. So, assuming you can get `kubectl proxy` to find your process, you can browse to `/config` and it will display the current `IConfiguration` as json.

I've usually done this with `Newtonsoft.Json` but I thought I'd try to get it to work with `System.Text.Json` and Endpoint Routing

First, to use it, simply add to the `UseEndpoints` section of Startup (or via the fluent builder style in `Program`). Here's a much abreviated `Startup`...
```csharp
    public class Startup
    {
        // ...

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            // ...

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapConfig();
                // ...
            });
        }
    }

```

I have been playing with conventions and naming. I like the idea of putting this type of helper in the Microsoft namespace that is required for the AspNet framework feature. If you "goto definition" on something like `MapControllers()` or `MapRazorPages()` you'll that MS uses the same idea. It greatly enhances discoverability as you don't need to *know* some special namespace to find what you need. Just dot your way to glory because it is already in scope.

The first new thing for me was the technique for adding an Endpoint route. Turns out it's pretty easy, see `MapConfig`. It's just a different way of mapping your existing Middleware.

The second was custom serialization using `System.Text.Json`. Because `IConfiguration` is actually a collection of Sources (ie Json, Environment, etc), you get a better result if you walk the sections, see `WriteJson`. The previous version using `Newtonsoft.Json` was a bit more consise but this turned out ok and it removes a dependency and some versioning hell.

Also, note the use of `await using`. Writing to the response body stream **must** be done with async. If not then it will throw an exception, complaining that sync is not allowed. Without `async using` it will use the "old" `IDisposable.Dispose()` which uses `Flush` rather than `FlushAsync`.

```csharp
namespace Microsoft.AspNetCore.Builder
{
    public static class ConfigurationEndpoint
    {
        public const string DefaultPath = "/config";

        /// <summary>
        /// expose current configuration via an endpoint
        /// </summary>
        public static IEndpointConventionBuilder MapConfig(this IEndpointRouteBuilder endpoints, string pattern = DefaultPath)
        {
            if (endpoints.ServiceProvider.GetService(typeof(IConfiguration)) == null)
                throw new InvalidOperationException($"Unable to find {nameof(IConfiguration)}");

            var pipeline = endpoints.CreateApplicationBuilder()
               .UseMiddleware<ConfigurationMiddleware>()
               .Build();

            return endpoints.Map(pattern, pipeline).WithDisplayName("Configuration");
        }

        private class ConfigurationMiddleware
        {
            private readonly IConfiguration config;

            public ConfigurationMiddleware(RequestDelegate next, IConfiguration config)
            {
                this.config = config ?? throw new ArgumentNullException(nameof(config));
            }

            public async Task Invoke(HttpContext context)
            {
                context.Response.StatusCode = 200;
                context.SetNoCacheHeaders();

                if (context.Request.ContentType == "text/plain")
                    await WriteAsText(context.Response);
                else
                    await WriteAsJson(context.Response);
            }

            private async Task WriteAsText(HttpResponse response)
            {
                response.ContentType = "text/plain";
                foreach (var kvp in config.AsEnumerable())
                    await response.WriteAsync($"{kvp.Key} = {kvp.Value}\n");
            }

            private async ValueTask WriteAsJson(HttpResponse response)
            {
                response.ContentType = "text/json";
                await using (var writer = new Utf8JsonWriter(response.Body))
                    Write(writer, config, new JsonSerializerOptions { WriteIndented = true });

                void Write(Utf8JsonWriter writer, IConfiguration value, JsonSerializerOptions options)
                {
                    if (value is IConfigurationSection section)
                    {
                        if (section.Value is null)
                            writer.WriteStartObject(section.Key);
                        else
                        {
                            writer.WriteString(section.Key, section.Value);
                            return;
                        }
                    }
                    else
                        writer.WriteStartObject();

                    foreach (var child in value.GetChildren())
                        Write(writer, child, options);

                    writer.WriteEndObject();
                }
            }
        }
    }
}
```

I have similar endpoints for Environment which shows some underlying info like versions, host name and os, environment variables, and lists version of our assemblies. Something like...
```
{
    Environment: "development",
    EntryAssemblyName: "EventHubService",
    EntryAssemblyVersion: "0.3.29+d57ddbd12b",
    FrameworkDescription: ".NETCoreApp,Version=v3.1",
    LocalTime: "2020-03-21T17:51:32.5289499Z",
    MachineName: "flintstone",
    OperatingSystemArchitecture: "X64",
    OperatingSystemPlatform: "WINDOWS",
    OperatingSystemVersion: "Microsoft Windows 10.0.18363",
    ProcessArchitecture: "X64",
    ProcessorCount: "24",
    Env: {
        ASPNETCORE_URLS: "https://localhost:5001;http://localhost:5000",
        ASPNETCORE_ENVIRONMENT: "Development",
        ASPNETCORE_HTTPS_PORT: "5001"
    },
    Versions: {
        MyCompany.SignalR: "0.3.29+d57ddbd12b",
        MyCompany.Hosting: "0.3.27+1a64bae350",
        MyCompany.Feature: "0.3.27+1a64bae350",
        MyCompany.Events: "0.3.23+5f1a1ba42d",
        MyCompany.Domain: "0.3.27+1a64bae350"
    }
}
```
Super useful to *prove* that the enironemtn is actually running what you think is running.
But that's for another day.
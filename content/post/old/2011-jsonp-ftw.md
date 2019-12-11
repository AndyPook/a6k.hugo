---
title: JSONP FTW
date: 2010-03-09T11:18:00Z
summary: Extending WebAPI to support jsonp
tags: [dotnet]
---

We’ve been looking at how to integrate our stuff with MS Dynamics CRM. Dynamics does not play nice with others. I won’t go into the details here but I think we’re going to end up using javascript as a proxy to get things done. As a result I’ve been looking at the new (ish) WebAPI bits from the WCF team (http://wcf.codeplex.com). The basic motivation behind this project is to make WCF talk HTTP like a native. Things like content format negotiation are baked in. So you can write a single service and have different clients receive differently formatted responses. So a JQuery ajax call will see JSON another client might see XML. You can even point a browser at your service and get HTML via Razor templates (very useful if you want to add an admin UI to your service.
The point of this post is about content format negotiation and dealing with JsonP.
A lot (errr most) of this is taken from [Alexander Zeitlers article](http://blog.alexonasp.net/post/2011/07/26/Look-Ma-I-can-handle-JSONP-(aka-Cross-Domain-JSON)-with-WCF-Web-API-and-jQuery!.aspx). He mentions part way through to grab a file from the WebAPI project. I’ve added a little of my own flavour to this part.

WebAPI has the concept of MediaTypeFormatters. When a request comes in it will have an “accept” header which tells the server which media types the client can handle. A JQuery ajax request would send “application/json”, a browser would send “text/html”.
The [accept](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html) header value is used to look up which formatter to use to format the response.
There are times, however, when you want to force the format. Testing via a browser is one. But more importantly when using JsonP the request has an accept header of “*/*”. In this case you always want the response in json.
In the ContactManager_Advanced project in the samples included in the codeplex project there is an example of a “MessageChannel” that inspects the uri and sets the accept header. I’ve customised this a little so that it also looks for a “format” parameter in the querystring. It also forces to json if the is a “callback” parameter in the querystring.
Lastly I’ve changed the fluent interface. It made little sense to me to have an extension method on HttpApplication.
Here’s the listing:

```csharp
public static class UriFormatExtensionMessageChannelExtensions
    {
        public static IHttpHostConfigurationBuilder AddUriFormatExtension(this IHttpHostConfigurationBuilder builder)
        {
            return builder.AddMessageHandlers(typeof(UriFormatExtensionMessageChannel));
        }
    }

    public class UriFormatExtensionMessageChannel : DelegatingChannel
    {
        public UriFormatExtensionMessageChannel(HttpMessageChannel handler) : base(handler) { }

        private static Dictionary<string, MediaTypeWithQualityHeaderValue> extensionMappings = new Dictionary<string, MediaTypeWithQualityHeaderValue>();

        public static FluentExtensionMappings SetUriExtensionMapping(string extension, string mediaType)
        {
            extensionMappings[extension] = new MediaTypeWithQualityHeaderValue(mediaType);
            return new FluentExtensionMappings();
        }

        protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
            if (!TryGetLastSegmentFormat(request))
                TryGetQSFormat(request);

            return base.SendAsync(request, cancellationToken);
        }

        /// <summary>
        /// Try to get the format from the last segment of the Uri
        /// </summary>
        /// <example>http://example.com/product/1/json</example>
        /// <param name="request"></param>
        /// <returns>true if a format was found</returns>
        private static bool TryGetLastSegmentFormat(HttpRequestMessage request)
        {
            var segments = request.RequestUri.Segments;
            var lastSegment = segments.LastOrDefault();

            MediaTypeWithQualityHeaderValue mediaType;
            if (extensionMappings.TryGetValue(lastSegment, out mediaType))
            {
                var newUri = request.RequestUri.OriginalString.Replace("/" + lastSegment, "");
                request.RequestUri = new Uri(newUri, UriKind.Absolute);
                request.Headers.Accept.Clear();
                request.Headers.Accept.Add(mediaType);
                return true;
            }

            return false;
        }

        /// <summary>
        /// Try to get the format from the query string of the Uri.
        /// If it's a JsonP callback then force to json
        /// </summary>
        /// <example>http://example.com/product/1?format=json</example>
        /// <param name="request"></param>
        /// <returns>true if a format was found</returns>
        private static bool TryGetQSFormat(HttpRequestMessage request)
        {
            var qsValues = HttpUtility.ParseQueryString(request.RequestUri.Query);
            var format = qsValues["format"];
            bool rebuildUri = false;
            if (!string.IsNullOrEmpty(format))
                rebuildUri = true;

            // if it's a JsonP callback then force to json
            if (!string.IsNullOrEmpty(qsValues["callback"]))
                format = "json";

            MediaTypeWithQualityHeaderValue mediaType;
            if (!string.IsNullOrEmpty(format) && extensionMappings.TryGetValue(format, out mediaType))
            {
                if (rebuildUri)
                {
                    var newUriBuilder = new UriBuilder(request.RequestUri);
                    qsValues.Remove("format");
                    newUriBuilder.Query = qsValues.ToString();
                    request.RequestUri = newUriBuilder.Uri;
                }

                request.Headers.Accept.Clear();
                request.Headers.Accept.Add(mediaType);
                return true;
            }

            return false;
        }

        protected override HttpResponseMessage Send(HttpRequestMessage request, CancellationToken cancellationToken)
        {
            throw new NotImplementedException();
        }

        public sealed class FluentExtensionMappings
        {
            public FluentExtensionMappings SetUriExtensionMapping(string extension, string mediaType)
            {
                extensionMappings[extension] = new MediaTypeWithQualityHeaderValue(mediaType);
                return this;
            }
        }
    }
```

So now your global.asax Application_Start will have a snippet like this.

```csharp
UriFormatExtensionMessageChannel
  .SetUriExtensionMapping("xml", "application/xml")
  .SetUriExtensionMapping("json", "application/json")
  .SetUriExtensionMapping("png", "image/png")
  .SetUriExtensionMapping("odata", "application/atom+xml");

var config = HttpHostConfiguration.Create()
  .AddUriFormatExtension()
  .AddJsonpHandler();
```

The AddJsonpHandler line is a simple extension method to wrap Alex’s [JsonpResponseHandler](http://blog.alexonasp.net/post/2011/07/26/Look-Ma-I-can-handle-JSONP-(aka-Cross-Domain-JSON)-with-WCF-Web-API-and-jQuery!.aspx).

```csharp
    public static class JsonpResponseHandlerExtensions
    {
        public static IHttpHostConfigurationBuilder AddJsonpHandler(this IHttpHostConfigurationBuilder builder)
        {
            return builder.AddResponseHandlers(c => c.Add(new JsonpResponseHandler()), (s, d) => true);            
        }
    }
```

Although this example is using IIS to host the service all of this is equally applicable to self hosted services.
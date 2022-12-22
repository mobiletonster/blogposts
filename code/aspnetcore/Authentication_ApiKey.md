## Securing WebApi's in ASP.NET Core with ApiKey
Securing WebApi's in ASP.NET Core is flexible and can be done in many ways. In this post, we will show you how to use the ApiKey authentication method.

### ApiKey authentication
Often times, a WebApi is secured with a simple ApiKey authentication. In ASP.NET Core MVC/WebApi (not minimal api at this time) the common approach is to use a special ApiKeyAttribute filter decorating the controller or action that requires authentication to inspect the request and check for the presence of an ApiKey header value.

```csharp
    [AttributeUsage(validOn: AttributeTargets.Class | AttributeTargets.Method)]
    public class ApiKeyAttribute : Attribute, IAsyncActionFilter
    {

        private const string APIKEYNAME = "ApiKey";
        public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
        {
            if (!context.HttpContext.Request.Headers.TryGetValue(APIKEYNAME, out var extractedApiKey))
            {
                context.Result = new ContentResult()
                {
                    StatusCode = 401,
                    Content = "Api Key was not provided"
                };
                return;
            }

            var appSettings = context.HttpContext.RequestServices.GetRequiredService<IConfiguration>();

            var apiKey = appSettings.GetValue<string>(APIKEYNAME);

            if (!apiKey.Equals(extractedApiKey))
            {
                context.Result = new ContentResult()
                {
                    StatusCode = 401,
                    Content = "Api Key is not valid"
                };
                return;
            }

            await next();
        }
    }
```

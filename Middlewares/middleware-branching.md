## Middleware Branching

Branching allows a request to **leave the main middleware pipeline and execute a separate pipeline** based on a path or condition. This is used to avoid unnecessary middleware execution and isolate special behaviors.

---

### Map Branching

`Map()` creates a **separate pipeline based on a request path**.  
If the path matches, the request enters the branch and **does not return to the main pipeline**.

Real-life example:  
A `/health` endpoint used by load balancers to check if the server is alive without passing through authentication, logging, or heavy middleware.

Example:

```csharp
app.Map("/health", branch =>
{
    branch.Run(async context =>
    {
        await context.Response.WriteAsync("Healthy");
    });
});
```

Request `/health` goes directly to this pipeline and skips the rest of the application.

---

### MapWhen Branching

`MapWhen()` creates a **separate pipeline based on any boolean condition**.  
If the condition is true, the request moves into that branch and **does not continue in the main pipeline**.

Real-life example:  
Blocking requests from certain IP addresses or countries before they reach the application.

Example idea:

```csharp
app.MapWhen(
    context => context.Connection.RemoteIpAddress?.ToString() == "blocked-ip",
    branch =>
    {
        branch.Run(async context =>
        {
            context.Response.StatusCode = 403;
            await context.Response.WriteAsync("Access denied");
        });
    });
```

Used for security filtering, bot blocking, or routing experiments.

---

### UseWhen Conditional Middleware

`UseWhen()` conditionally **executes a middleware but keeps the request inside the main pipeline**.  
After the middleware runs, the request continues normally.

Real-life example:  
Enable detailed request logging only for API routes without affecting other endpoints.

Example:

```csharp
app.UseWhen(
    context => context.Request.Path.StartsWithSegments("/api"),
    branch =>
    {
        branch.Use(async (context, next) =>
        {
            Console.WriteLine("API request detected");
            await next();
        });
    });
```

The request still continues through the rest of the main middleware pipeline.

---

## Key Difference

Map / MapWhen  
Request **leaves the main pipeline and enters another pipeline**.

UseWhen  
Request **temporarily enters a conditional middleware but returns to the main pipeline afterward**.

---

One subtle engineering insight worth remembering.

In large production systems, branching is often used very early in the pipeline for extremely frequent routes like:

```
/health
/metrics
/static
```

This keeps the main pipeline lighter and avoids executing expensive middleware (authentication, logging, database calls) for requests that do not need them.

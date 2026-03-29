![[Pasted image 20260313095115.png]]

- Responsible for matching incoming requests and dispatching them to endpoints

![[Pasted image 20260313095237.png]]

## Registering endpoints:
- Simply use `app.MapGet` or `app.MapControllers`
![[Pasted image 20260313101932.png]]

## **Manual Routing with `Map` / `MapWhen`**

**High-level routing:**

```csharp
app.UseRouting();
app.MapControllers(); // automatically maps /Controller/Action/{id?}
```

**Manual routing example (useful if you need middleware for certain requests):**

```csharp
//Usefull when we need a middleware on that enpoint
// Block certain IPs (e.g., internal or banned clients)
app.MapWhen(
    context => context.Connection.RemoteIpAddress?.ToString() == "192.168.1.100",
    branch =>
    {
        branch.Run(async context =>
        {
            context.Response.StatusCode = StatusCodes.Status403Forbidden;
            await context.Response.WriteAsync("Access denied for your IP");
        });
    }
);

// Custom health check endpoint without creating a controller
app.Map("/health", hBranch =>
{
    hBranch.Run(async context =>
    {
        await context.Response.WriteAsync("Healthy");
    });
});
```

**Key real-world use cases:**

1. Quick endpoints (health checks, maintenance mode) without a controller.
    
2. Conditional pipelines (VIP users, banned IPs, feature flags).
    
3. Skipping heavy middleware for lightweight requests (static files, ping endpoints).
    

**Summary:**

- Use `Map` → route directly to a path.
    
- Use `MapWhen` → route based on a **condition** in HTTP context.
    
- Avoid overusing; keep high-level routing for standard CRUD controllers.
    

[[Routing Template]]

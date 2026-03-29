
Middleware executes **in the order it is registered**.  
Each middleware decides whether to pass the request to the next component or terminate the pipeline.

Correct ordering ensures security, performance, and correct routing behavior.

---
![[Pasted image 20260306113610.png]]
### UseExceptionHandler

Captures unhandled exceptions thrown by downstream middleware and returns a safe response instead of crashing the request.

Real-life example:  
If a database failure occurs in a controller, this middleware catches the exception and returns a friendly `500` response instead of exposing stack traces.

This must appear **at the beginning** so it can catch exceptions from everything below.

---

### UseHsts

Forces browsers to always use HTTPS for future requests by sending the **Strict-Transport-Security header**.

Real-life example:  
After a user visits your site once over HTTPS, the browser will automatically refuse future HTTP connections to prevent downgrade attacks.

This is a **security protection for production environments**.

---

### UseHttpsRedirection

Redirects incoming HTTP requests to HTTPS.

Real-life example:  
If a user types:

```
http://myapp.com
```

the server automatically redirects to:

```
https://myapp.com
```

This ensures encrypted communication between client and server.

---

### UseStaticFiles

Serves static resources such as images, CSS, JavaScript, and fonts directly from disk.

Real-life example:  
A request for:

```
/images/logo.png
```

is returned immediately without executing authentication, routing, or controller logic.

This improves performance because static files are extremely common.

---

### UseRouting

Matches the incoming request to an endpoint defined in the application.

Real-life example:  
Mapping a request like:

```
GET /users/15
```

to the controller endpoint:

```
UserController.GetUser(int id)
```

Routing must run **before authentication and authorization** because those systems need to know which endpoint is being accessed.

---

### UseCors

Controls which external domains are allowed to access the API.

Real-life example:  
Allow requests from:

```
https://myfrontend.com
```

but block requests from unknown domains.

This prevents malicious websites from making requests to your API from a browser.

---

### UseAuthentication

Determines **who the user is** by validating credentials such as JWT tokens or cookies.

Real-life example:  
Reading a JWT token from the request header:

```
Authorization: Bearer <token>
```

and identifying the logged-in user.

---

### UseAuthorization

Determines **what the authenticated user is allowed to do**.

Real-life example:  
A user may be authenticated but still blocked from accessing an admin endpoint unless they have the `Admin` role.

Authentication answers **who you are**.  
Authorization answers **what you can do**.

---

### Custom Middleware

Application-specific logic that runs after security checks.

Real-life examples:

- request logging
    
- performance measurement
    
- feature flags
    
- tenant detection in multi-tenant systems
    

Example:

```csharp
app.Use(async (context, next) =>
{
    var start = DateTime.UtcNow;

    await next();

    var duration = DateTime.UtcNow - start;
    Console.WriteLine($"Request took {duration.TotalMilliseconds} ms");
});
```

---

### Endpoint Execution (Minimal APIs / Controllers)

The final stage where the request reaches the actual application logic.

Real-life example:

```csharp
app.MapGet("/users", () => GetUsers());
```

or a controller action like:

```
GET /orders/12
```

This is where business logic runs and the response is generated.

---

### Mental Model

Think of middleware ordering like **airport security checkpoints**:

1. Error handling (catch unexpected problems)
    
2. Secure transport enforcement
    
3. Static resource shortcut
    
4. Route identification
    
5. Cross-origin security
    
6. Identity verification
    
7. Permission verification
    
8. Application logic
    

Every request walks through these gates in sequence. A poorly ordered pipeline is like checking passports **after** passengers board the airplane.

---

One small architectural observation worth carrying forward.

In large enterprise APIs, the middleware pipeline is often used as a **system-wide policy engine**. Logging, rate limiting, security headers, tenant detection, request tracing, and performance metrics are all centralized here so that every endpoint inherits the same behavior automatically. This keeps controllers thin and predictable.
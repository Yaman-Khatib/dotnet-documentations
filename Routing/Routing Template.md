Tokens within /`{...}` defines route segment that are bound if the route is matched
![[Pasted image 20260313112600.png]]


## ASP.NET Core Routing Templates — Practical Notes

Routing templates define how **URL segments map to parameters** in your endpoint. Think of them as **URL patterns → method parameters**.

---

### 1. Basic Parameter

```csharp
app.MapGet("/products/{id}", (int id) =>
    $"Product id is: {id}");
```

Request:

```
GET /products/12
```

Result:

```
Product id is: 12
```

Pattern idea:  
`{id}` captures the value from the URL and binds it to the method parameter.

---

### 2. Multiple Parameters

```csharp
app.MapGet("/products/{year}/{month}/{day}",
(int year, int month, int day) =>
    $"Date is: {year}-{month}-{day}");
```

Request:

```
GET /products/13/2/2005
```

Result:

```
Date is: 13-2-2005
```

Useful when URLs encode **structured data**, like:

- `/reports/2025/03/10`
    
- `/archive/2024/11/05`
    

---

### 3. Default Value

```csharp
app.MapGet("/{controller=Home}",
(string controller) => $"Controller is: {controller}");
```

Requests:

```
GET /
GET /card
```

Results:

```
Controller is: Home
Controller is: card
```

Idea:  
If the segment is missing, the **default value is used**.

Common MVC example:

```
/{controller=Home}/{action=Index}/{id?}
```

---

### 4. Optional Parameter

```csharp
app.MapGet("/products/{id?}",
(int? id) =>
    (id is null) ? "All products" : $"Product id is: {id}");
```

Requests:

```
/products
/products/12
```

Results:

```
All products
Product id is: 12
```

Real use case:

```
/products       -> list
/products/12    -> details
```

---

### 5. Mixed Literal + Parameters

```csharp
app.MapGet("a{first}b{second}c",
(string first, string second) =>
    $"First: {first}, Second: {second}");
```

Requests:

```
/axbxc
/asooobxc
/aymnbsmrc
```

Matches pattern:

```
a{first}b{second}c
```

Example interpretation:

```
axbxc
a x b x c
```

Result:

```
First: x
Second: x
```

Rare in CRUD APIs but useful in **legacy URL formats or token parsing**.

---

### 6. Catch-All (Slug)

```csharp
app.MapGet("/{*slug}",
(string slug) => $"Slug is: {slug}");
```

Requests:

```
/home-page
/home/yaman/rare
```

Results:

```
Slug is: home-page
Slug is: home/yaman/rare
```

Real-world use cases:

**CMS pages**

```
/about
/pricing
/blog/my-first-post
```

**Documentation systems**

```
/docs/api/authentication
/docs/installation/linux
```

The slug captures **everything after `/`**.

---

### 7. Single `*` vs Double `**`

```
{*slug}
```

captures the path but **URL encoding may apply**.

```
{**slug}
```

preserves slashes exactly (better for file paths or nested content).

Example:

```
/docs/backend/auth/jwt
```

Slug:

```
docs/backend/auth/jwt
```

---

### Mental Model

```
URL Pattern → Parameters → Handler
```

Example:

```
/products/{id}
        ↓
/products/12
        ↓
id = 12
```

The router simply **parses the URL and injects values into the handler parameters**.

---

A curious twist of routing philosophy: URLs are not just addresses; they become a **language for describing resources**. Good APIs feel almost like reading sentences:

```
/users/42/orders/2025
```

The router just plays translator between **HTTP paths and executable code**.
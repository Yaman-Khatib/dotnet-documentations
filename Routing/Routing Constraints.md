Here’s a **clean, professional note** you can drop directly into your `.md` notebook — focused, practical, and structured for real use.

---

#  ASP.NET Core Routing Constraints (Minimal APIs)

##  What are Route Constraints?

Route constraints **restrict URL matching** based on rules.

> If a constraint fails → route does NOT match → **404 Not Found**

---

## Register Custom Constraints

```csharp
builder.Services.AddRouting(options =>
{
    options.ConstraintMap.Add("validEmail", typeof(EmailConstraint));
});
```

 `"validEmail"` becomes usable inside route templates.

---

##  Built-in Constraints (Practical Use)

###  Type Constraints

```csharp
app.MapGet("/products/int/{id:int}", (int id) => ...);
app.MapGet("/products/{id:Guid}", (Guid id) => ...);
app.MapGet("/orders/{date:DateTime}", (DateTime date) => ...);
```

 Ensures value **can be parsed** to the type.

---

### 🔹 Format Constraints

```csharp
app.MapGet("/products/{code:alpha}", (string code) => ...);
```

 `alpha` = letters only

---

###  Range Constraint

```csharp
app.MapGet("/users/{age:range(18,40)}", (int age) => ...);
```

👉 Limits numeric values

---

###  Required Constraint

```csharp
app.MapGet("/tasks/required/{task:required}", (string task) => ...);
```

 Ensures parameter exists and is not empty

---

## ⚠️ Ambiguity Problem (Important)

```csharp
// ❌ Avoid this
/products/{price:double}
/products/{price:decimal}
/products/{price:float}
```

 `133.31` matches all → **routing conflict**

✔ Solution:

- use one type only
    
- or differentiate routes
    

---

##  Regex Constraint (Inline)

```csharp
app.MapGet(@"/users/number/{phone:regex(^\+1-\d{3}-\d{3}-\d{4}$)}",
    (string phone) => ...);
```

👉 Validates format at routing level

---

##  Custom Constraint Usage

```csharp
app.MapGet("/users/email/{email:validEmail}", (string email) => ...);
```

 Uses your registered constraint

---

## ⚠️ Important Behavior

### ❗ Constraints DO NOT validate business rules

They only check:

- format
    
- convertibility
    

---

### ❗ Failure result

| Scenario            | Result          |
| ------------------- | --------------- |
| Constraint fails    | 404 Not Found   |
| Model binding fails | 400 Bad Request |

---

### ❗ Example

```csharp
{id:int}
```

- `"abc"` → ❌ 404 (route not matched)
    
- `"999999999999"` → ✔ matched (even if logically invalid)
    

---

##  Best Practices

- Prefer **built-in constraints** over regex
    
- Use constraints for:
    
    - format
        
    - disambiguation
        
- Avoid:
    
    - database checks
        
    - business logic
        

---

## Se also:
- [[regex]]
- 
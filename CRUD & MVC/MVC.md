Here’s a more **practical, code-focused MVC note** based on your journey, including GET/POST rules, form binding, and return routing:

---

## ASP.NET MVC Practical Notes – Extended

### 1. **Folder & Naming Conventions**

```text
Controllers/
    ProductsController.cs
Models/
    Product.cs
Views/
    Products/
        Index.cshtml
        Create.cshtml
        Edit.cshtml
        Details.cshtml
        Delete.cshtml
    Shared/
        _Layout.cshtml
_ViewStart.cshtml
_ViewImports.cshtml
```

- Controller name → `<Name>Controller` → folder in Views minus `Controller`.
    
- View name → action name (`Index` → `Index.cshtml`).
    
- Shared layout: `_Layout.cshtml`.
    

---

### 2. **_ViewStart.cshtml**

```csharp
@{ Layout = "~/Views/Shared/_Layout.cshtml"; }
```

- Applies layout to all views automatically.
    
- Saves from repeating `Layout` in every view.
    

---

### 3. **_ViewImports.cshtml**

```csharp
@using M01_MVC.Models
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

- Global imports for all Razor views.
    
- Enables `asp-action`, `asp-route-id`, and model references without per-view repetition.
    

---

### 4. **Controller Actions**

```csharp
public class ProductsController(ProductStore store) : Controller
{
    // GET → View
    [HttpGet]
    public IActionResult Index()
    {
        var products = store.GetAll();
        return View(products); // View folder = Products, view file = Index.cshtml
    }

    [HttpGet]
    public IActionResult Details(int id)
    {
        var product = store.Get(id);
        return product != null ? View(product) : NotFound();
    }

    [HttpGet]
    public IActionResult Create() => View();

    // POST → data creation/updating
    [HttpPost]
    public IActionResult Create(Product product)
    {
        product.Id = Guid.NewGuid();
        store.Add(product);
        return RedirectToAction(nameof(Index)); // safer than hardcoded "Index"
    }

    [HttpPost]
    public IActionResult Edit(Product product)
    {
        store.Update(product);
        return RedirectToAction(nameof(Index));
    }
}
```

- **GET** → render view, display form or page.
    
- **POST** → submit form, modify data.
    
- HTML forms cannot call PUT/DELETE directly (use JS/fetch for APIs).
    

---

### 5. **View Basics**

```csharp
@model ICollection<Product>
@{
    ViewData["Title"] = "All Products";
}

<table class="table table-striped table-bordered">
    <thead class="table-dark">
        <tr>
            <th>Name</th>
            <th class="text-end">Price</th>
            <th style="width:180px"></th>
        </tr>
    </thead>
    <tbody>
        @foreach(var p in Model)
        {
            <tr>
                <td>@p.Name</td>
                <td class="text-end">@p.Price.ToString("c")</td>
                <td>
                    <div class="btn-group btn-group-sm">
                        <a class="btn btn-secondary" asp-action="Edit" asp-route-id="@p.Id">Edit</a>
                        <a class="btn btn-info" asp-action="Details" asp-route-id="@p.Id">Details</a>
                        <a class="btn btn-danger" asp-action="Delete" asp-route-id="@p.Id">Delete</a>
                    </div>
                </td>
            </tr>
        }
    </tbody>
</table>
```

- `@Model` → data passed from controller.
    
- `@foreach` → iterate model collection.
    
- `asp-action` + `asp-route-id` → auto-generates links based on routing.
    

---

### 6. **Forms**

```csharp
@model Product
<form asp-action="Create" method="post">
    <div class="mb-3">
        <label asp-for="Name" class="form-label"></label>
        <input asp-for="Name" class="form-control" autocomplete="off"/>
    </div>
    <div class="mb-3">
        <label asp-for="Price" class="form-label"></label>
        <input asp-for="Price" type="number" class="form-control"/>
    </div>
    <button type="submit" class="btn btn-primary">Save</button>
    <a class="btn btn-secondary" asp-action="Index">Cancel</a>
</form>
```

- `asp-for` binds input to model property.
    
- Submit triggers **POST** endpoint.
    
- Use `RedirectToAction(nameof(Index))` after saving → safer than string literal.
    

---

### 7. **Layouts & Bootstrap**

```html
<html data-bs-theme="dark">
<head>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet"/>
</head>
<body>
    <div class="container mt-4">
        <h1>@ViewData["Title"]</h1>
        @RenderBody() <!-- inserts action view -->
    </div>
</body>
</html>
```

- `_Layout.cshtml` wraps views.
    
- `@RenderBody()` → placeholder for controller view.
    
- Bootstrap classes style tables, forms, buttons, and dark/light theme.
    

---

### 8. **Routing**

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Products}/{action=Index}/{id?}");
```

- `{controller}` → folder/controller
    
- `{action}` → action method
    
- `{id?}` → optional parameter
    
- MVC automatically matches action → view file.
    

---

### 9. **Takeaways / Rules**

- **GET → display view**, **POST → modify data**.
    
- **Controller name → view folder**, **Action → view file**.
    
- `_Layout` + `_ViewStart` → consistent layout.
    
- `_ViewImports` → shared imports for all views.
    
- **Tag helpers (`asp-*`)** → dynamic, safe links.
    
- `RedirectToAction(nameof(Index))` → avoid hardcoding route strings.
    
- HTML forms only support GET/POST; use JS for PUT/DELETE.
    

---
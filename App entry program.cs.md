this lesson was great as it was explained in details, I usually just guess where I should place middleware configuration or 
![[Pasted image 20260129182444.png]]

## Main parts:
```C#
var builder = WebApplication.CreateBuilder(args);
//This creates builder configuration sources are loaded here (appsettings.json, environment variables, etc.)  
// default DI sources are initialized (Logger, etc.)
//Services registeration phase

var app = builder.Build(); 

app.MapGet("/products", ()=> "This is products");

app.Run();
//Web server start listening to HTTP requests

```
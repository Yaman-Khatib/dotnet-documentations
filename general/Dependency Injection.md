## Overview:
- Service: a reusable **class** that does specific job 
- In real world: a service usually depends on other service
- logic is splitted across services
## Tightly coupling:
- it is when you have the dependencies initialized inside a service using **new** keyword
- this causes issues that any change in the dependency needs to change all services that uses it
- Makes testing complex and hardens moking
## Dependency Inversion:
- it solves the tightly coupling issue
- instead of depending on concrete classes you depend on interfaces
- then you inject the implementation (by constructor, method . . .)
- it can be useful to inject mock services for testing as the service doesn't depend on specific implementation you can pass any thing that implements that interface
## IoC containers:
- What do IoC stands for: Inversion of control Principle
- ASP .NET Comes with built in IoC
- application has a container called `IServiceCollection` is used to register services
- you just add services to the dotnet services container 
- it will automatically inject the dependencies for the constructor of services
- it the best option usually
- you may need 3rd party containers for edge cases Autofac, Dryloc for method injection

## Built in ASP.NET Services
- +250 services is already injected (which is usually used in systems but is not business logic)
- Examples: `ILogger`, `ICongfiguration`
## Grouping services
- It is a good and clean practice to group related services together instead of cluttering `program.cs` with hundreds of services registering
- I didn't understand the point where we added 
	`namespace: Microsoft.Extensions.DependencyInjection;` so it is accessible in `program.cs`
- We make it as extension method so it accepts parameter that has `this` and returns the same type `IServiceCollection` to allow extension chaining: 
  `builder.Services.AddWeatherServices().AddAccountingServices();`

- `namespace Microsoft.Extensions.DependencyInjection;` The dependency injection class should have this namespace in order to make this class available in all projects

## Registering services: Extension vs Descriptor:
- we normally do: `builder.Services.AddTransient<IMyService,Myservice>();`
- but we can actually use advanced way that helpes with reflection and metadata:
  ```C#
  builder.Services.Add(
  new ServiceDescriptor(
  typeof(IMyService),
  typeof(MyService),
  ServiceLifetime.Transient
  )
  );
  ```
## Multiple registerations:
- sometimes we have multiple implementations for same interface
- If you register multiple implementation for sma interface it will use the last service injected
```C#
builder.Services.AddScoped<IBucketService,AWSBucketService>();
builder.Services.AddScoped<IBucketService,FlameBucketService>();

//the one will be used is tha latest registered
```

- You can use multiple services:
```C#
constructor(IEnumerable<IBucketService> services)
{
}

// now you can use all injected service for that interface:
foreach(var service in services)
{
service.DoSomething();
}
```

## Keyed services:
- you have many implementations for one service so you can add key for each implementation inorder to use a specific implementation

```C#

// now you can use all injected service for that interface:


builder.Services
.AddKeyedTransient<IBucketService,SupabaseBucketService>("Supabase");

builder.Services
.AddKeyedTransient<IBucketService,AwsBucketService>("AWS");

constructor([FromKeyedServices("AWS")]IBucketService services)
{
	
}

// now you can use all injected service for that interface:
foreach(var service in services)
{
service.DoSomething();
}
```

## Factory pattern:

```C#
//Use factory pattern
builder.Services.AddTransient<IPaymentService>(
    sp =>
    {
        var config = sp.GetRequiredService<IConfiguration>();
        return config["PaymentMethod"] == "Stripe"?
        new StripePayment():
        new PaypalPayment();
    }
);
```

- Based on the configuration we decide which implementation we want for this service
- I cant understand why it is used if we can use keyed services I must find a special usecase for factory pattern

## Resolving at startup - Seeders and more:
```C#
var app = builder.Build();
using(var scope = app.Services.CreateScope())
{
    var sp = scope.ServiceProvider;
    var service = sp.GetRequiredService<DbInitializer>();
    service.SeedDb();
}
```

- This is useful for anything that needs to be done after project startup 
- run one-time startup service: currency sync
- Seed database
- Ping external services

## Service Lifetimes:
![[Pasted image 20260221145037.png]]

- Transient: when you want a fresh object from the service every time (even during request)
- Scoped: you get fresh instance of the service for every request (but during the same request you will get the same instance)
- Singleton: created at project startup and (the same instance of the service is used for all requests)

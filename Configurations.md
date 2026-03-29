## Host VS Application Configurations:
- Environment configuration: used to start the app (app name, root content)
- Application configuration: (Connection Strings, API keys, external services URLs), **It has higher priority over environment** 

## Configuration providers:

![[Pasted image 20260129215500.png]]


## Environment variables:
THE SECURITY OF YOUR SECRETS
Environment variables are used because they solve a very specific problem: **you never hardcode secrets** into source code or config that gets committed to Git. That’s the whole point. They’re not supposed to “travel with the code.” They’re supposed to stay OUT of the repo.

Here’s the real breakdown, no fluff.

### Why environment variables exist

They let you inject secrets or system-specific config at runtime, without putting them in git.  
Secrets in codebases always leak. Someone screenshots your repo, forks it, your dev machine gets hacked, you forget to remove a key, etc.

Environment variables keep your code clean:

- Code stays the same everywhere.
    
- Only the environment changes.
    

### “If someone pulls my code, how can they test?”

Simple. They set their own environment variables.

Example:  
You tell them:

- APISECRET
    
- DB_CONNECTION
    
- SENDGRID_KEY
    

They set them locally. They don’t need your production keys. They only need **their own test/dev keys**.

This prevents:

- Production secrets leaking to developers
    
- Accidental commits of sensitive data
    
- Hardcoded credentials that stay in git forever
    

Every serious team works this way.

### Why not json?

Storing secrets in json that sits in the repo is a disaster:

- That json gets committed
    
- It gets cloned to every developer’s machine
    
- It gets uploaded to GitHub/GitLab/Bitbucket
    
- The secrets live forever in commit history, even if you delete the file later
    

Cloud providers consider that a breach.

Environment variables avoid that risk.

### Where they’re used

Here’s the real-world use case breakdown.

### Development

You use:

- `.env` files ignored by git
    
- Environment variables set locally
    

Developers have their own “dev” API keys that aren’t sensitive.  
Those aren’t production-critical and can be regenerated any time.

Why? Because you never want devs to accidentally use production secrets.

### So when do you use what?

**Environment variables:**

- Secrets
    
- Connection strings
    
- API keys
    
- Tokens
    
- Anything that would ruin your day if someone leaked it
    

**JSON config (appsettings.json):**

- Non-sensitive config
    
- Feature flags
    
- Email templates
    
- Logging settings
    
- App-level options
    
- Anything that is fine being public

### Types of env:
| Method          | Stored Where                  | Travels With Code                   | Environment        |
| --------------- | ----------------------------- | ----------------------------------- | ------------------ |
| `setx` / OS env | OS user profile               | No                                  | System-wide (user) |
| `.env`          | File in project               | If committed, yes → usually ignored | Project-specific   |
| User-Secrets    | Hidden folder in user profile | No                                  | Dev only           |


## Command Line Configurations:
- Has high priority
- example: `dotnet run --urls="https://localhost:7070` , `dotnet run --APIKEY=12f31`

## User Secrets:
- `dotnet user-secrets init` initialize the user secrets file
- `dotnet user-secrets set "APIKEY" "SOMEVALUE`
- it only works in development mode (in production you can't access user secrets)\


## File Configuration (json)
- .NET adds by default 3 files: *launch.json* configure environment, (*appsettings.json* , *appsettings.development.json* )
- The .NET auto adds those files, you can add new config files but you need to configure it using: `builder.Configuration.AddJsonFile("myCustom.json");`

## Memory Configuration:
```C#
var configData = new Dictionary<string, string?>
{
  {"MaxConnections", "10"},
  {"key2", "value2"},
  {"key3", "value3"}
};

builder.Configuration.AddInMemoryCollection(configData);
```

Usefull for testing it overrides files

 
## Order & Override Rules:
![[Pasted image 20260131222205.png]]

## Access Configurations:

To map to a complex object: `configuration.GetSection("BucketSettings").Get<BucketSettings>();`


## Options pattern
### Singlton Options

- first we configure the options in the DI container:
  ```C#
  builder.services.Configure<BucketSettings>(builder.Configuration.GetSection("BucketSettings"));
  ```
- In the constructor of service add `IOptions<BucketSettings> bucketOptions`
- now use it as: `bucketOptions.Value` e.g: `bucketOptions.Value.bucketUrl`
- The current implementation uses singleton snapshot lifetime
### Scoped (Snapshot options)
- snapshot: every new request will be affected by config file change
- to access it per scope inject it to constructor of serve using:

	```
	serviceName(IOptiosSnapshot<BucketSettings> bucketOptions) { 
	// implementation
	}
	```

### Monitor Options
- this tracks change and provides fresh copy of the options
- `IOptionsMonitor<BucketSettings> bucketsOption` every change on config file will be detected even during a request

### Comparing differeneces between configurations:
![[Pasted image 20260214111535.png]]
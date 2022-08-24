Identity's default settings for passwords requires at least six characters, a capital letter, a digit, and a special character.

However, depending on the type of application, we may want different password requirements. While we're still developing an application, we may actually want looser requirements to quickly create and login dummy accounts for experimentation. After all, it's a hassle to type out a long password every time when we are in development mode.

We can override Identity's default settings by updating the `ConfigureServices()` method of `Startup.cs`.

<div class="filename">Startup.cs</div>

```csharp
...

public void ConfigureServices(IServiceCollection services)
{
   ...

    services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ToDoListContext>()
        .AddDefaultTokenProviders();


    // This is new:   
    services.Configure<IdentityOptions>(options =>
    {
        options.Password.RequireDigit = false;
        options.Password.RequiredLength = 0;
        options.Password.RequireLowercase = false;
        options.Password.RequireNonAlphanumeric = false;
        options.Password.RequireUppercase = false;
        options.Password.RequiredUniqueChars = 0;
    });
}

...
```

The configuration above allows us to input a password of a single character to create a new user. Even though the `RequiredLength` property is 0, we can't actually put in an empty password because our application will throw an `ArgumentNullException: Value cannot be null` error otherwise.

It should be obvious that the settings above should never be used in a production environment. However, it makes our lives easier during development.

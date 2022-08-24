In this lesson, we'll add Identity to our to do list. We will follow the process for adding authentication to an existing .NET application.

## Identity Setup and Configuration
---

### Extending Identity Classes

Let's start by creating a class to represent user accounts. Fortunately, Identity comes with a class to represent users called `IdentityUser`. As we can see in [the `IdentityUser` documentation entry](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.identityuser.-ctor?view=aspnetcore-5.0), this class contains a number of properties such as `Email`s, `UserName`s, and many other properties including the number of recorded failed login attempts.

We'll implement this functionality by creating our own custom class that extends from `IdentityUser`.

Add a file named `ApplicationUser.cs` to `Models`. It should contain the following code:

<div class="filename">Models/ApplicationUser.cs</div>

```csharp
using Microsoft.AspNetCore.Identity;

namespace ToDoList.Models
{
    public class ApplicationUser : IdentityUser
    {

    }
}
```

We extend Identity's `IdentityUser` class into our own `ApplicationUser` class using the `:` operator in the class declaration. This ensures our `ApplicationUser` class inherits all necessary functionality from Identity.

### Configuring Identity to Work with Entity

Identity is built to use Entity Framework and store user information in a database. It comes with the class `IdentityDbContext`, which extends Entity Framework's `DbContext` class to work with user authentication.

First, we'll need to include the package that connects the two:

```bash
$ dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore -v 5.0.0
```

We'll update `ToDoListContext.cs` so that it extends from `IdentityDbContext`:

<div class="filename">Models/ToDoListContext.cs</div>

```csharp
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

namespace ToDoList.Models
{
    public class ToDoListContext : IdentityDbContext<ApplicationUser>
    {
      // Don't change code in here.
    }
}
```

* We'll have to add a new using directive: `using Microsoft.AspNetCore.Identity.EntityFrameworkCore;`

* Once again, we extend a class with the `:` operator. We're instructing our `ToDoListContext` class to inherit all functionality from Identity's built-in `IdentityDbContext` class. This replaces the DbContext class that ToDoListContext was previously extending from.

* Notice that we're declaring `ApplicationUser` as the _type_ of `IdentityDbContext` we're inheriting in the class declaration. This tells Identity which class in the application will contain the user account information it will be responsible for authenticating.

### Configuring `Startup`

Finally, let's update our `Startup.cs` file and configure the application to use Identity with Entity Framework and MVC.

<div class="filename">Startup.cs</div>

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using ToDoList.Models;
//new code
using Microsoft.AspNetCore.Identity;

namespace ToDoList
{
  public class Startup
  {
    public Startup(IWebHostEnvironment env)
    {
      var builder = new ConfigurationBuilder()
          .SetBasePath(env.ContentRootPath)
          .AddJsonFile("appsettings.json");
      Configuration = builder.Build();
    }

    public IConfigurationRoot Configuration { get; set; }

    public void ConfigureServices(IServiceCollection services)
    {
      services.AddMvc();

      services.AddEntityFrameworkMySql()
        .AddDbContext<ToDoListContext>(options => options
        .UseMySql(Configuration["ConnectionStrings:DefaultConnection"], ServerVersion.AutoDetect(Configuration["ConnectionStrings:DefaultConnection"])));
        
      //new code
      services.AddIdentity<ApplicationUser, IdentityRole>()
                .AddEntityFrameworkStores<ToDoListContext>()
                .AddDefaultTokenProviders();
    }

    public void Configure(IApplicationBuilder app)
    {
      app.UseDeveloperExceptionPage();

      //new code
      app.UseAuthentication(); 

      app.UseRouting();

      //new code
      app.UseAuthorization();

      app.UseEndpoints(routes =>
      {
        routes.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");
      });

      app.UseStaticFiles();
      
      app.Run(async (context) =>
      {
        await context.Response.WriteAsync("Hello World!");
      });
    }
  }
}
```

* We add the `using` statement `using Microsoft.AspNetCore.Identity;` so our `Startup` class has access to Identity.

* We also tell Identity what we want to use as a model for our user with the line `services.AddIdentity<ApplicationUser, IdentityRole>()`.

* The order of the code in the `Configure()` method matters. If `app.UseAuthentication()`, `app.UseRouting()`, or any other methods are called in the wrong order, you may run into unhandled exceptions or issues logging in. See the [Microsoft docs on middleware](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?view=aspnetcore-5.0&source=docs#middleware-order) for more info.

### Run Migrations

Now that the project is configured to use Identity, build the project and run the migration commands in the command prompt within the project folder:

```bash
$ dotnet ef migrations add addIdentity
$ dotnet ef database update
```

### Repository Reference

Follow the link below to view how a sample version of the project should look at this point. Note that this is a link to a specific commit in the repository.

**[<i class="glyphicon glyphicon-folder-open"></i> Example GitHub Repo for To Do List](https://github.com/epicodus-lessons/c-sharp-to-do-list-dotnet-5-week-5/tree/2_identity_setup)**


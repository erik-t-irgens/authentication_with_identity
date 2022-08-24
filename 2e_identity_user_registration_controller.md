Now that we've added Identity to our application, let's add functionality that allows users to register for accounts.

## Accounts Controller
---

We'll start by adding an `AccountController`. There will be a lot of new code here and we'll go over each addition carefully.

<div class="filename">Controllers/AccountController.cs</div>

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Identity;
using ToDoList.Models;
using System.Threading.Tasks;
using ToDoList.ViewModels;

namespace ToDoList.Controllers
{
    public class AccountController : Controller
    {
        private readonly ToDoListContext _db;
        private readonly UserManager<ApplicationUser> _userManager;
        private readonly SignInManager<ApplicationUser> _signInManager;

        public AccountController (UserManager<ApplicationUser> userManager, SignInManager<ApplicationUser> signInManager, ToDoListContext db)
        {
            _userManager = userManager;
            _signInManager = signInManager;
            _db = db;
        }

        public ActionResult Index()
        {
            return View();
        }

        public IActionResult Register()
        {
            return View();
        }

        [HttpPost]
        public async Task<ActionResult> Register (RegisterViewModel model)
        {
            var user = new ApplicationUser { UserName = model.Email };
            IdentityResult result = await _userManager.CreateAsync(user, model.Password);
            if (result.Succeeded)
            {
                return RedirectToAction("Index");
            }
            else
            {
                return View();
            }
        }
    }
}
```

* In addition to adding a `using` directive for `Microsoft.AspNetCore.Identity`, we also add one for `System.Threading.Tasks`. This will allow us to use asynchronous Tasks so we can use `async` and `await` to register new users.

* We have private preferences for `_userManager` and  `_signInManager`. We'll use **dependency injection** in the `AccountController` constructor to configure these services for us.

* Further, we've also added a `using` directive for `ToDoList.ViewModels`. We'll go over `ViewModels` and how to use them in our applications shortly.

### Dependency Injection

**Dependency injection** is the act of providing a helpful tool (known as a **service**) to part of an application that needs it _before_ it actually needs it. This ensures that the application doesn't need to worry about locating, loading, finding, or creating that service on its own.

In our case, we're injecting the Identity's `UserManager` and `SignInManager` services into the `AccountController` constructor so that our controller will have access to these services as needed.

This follows what is known as the "explicit dependencies principle," which states that methods and classes should explicitly require any dependencies. This makes the code much easier to read and understand and also ensures that our code will function correctly.

* As its name suggests, [the `UserManager` service](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.usermanager-1?view=aspnetcore-5.0) helps manage saving and updating user account information.

* Similarly, [the `SignInManager`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1?view=aspnetcore-5.0) provides functionality for users to log into their accounts.

If you'd like to learn more about .NET-specific dependency injection, we recommend beginning with the [.NET documentation on Dependency Injection](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection).

### Routing

Now let's take a look at the routes we've added to `AccountController`. Our `Index()` and `Register()` GET routes are straightforward. However, our `Register()` POST route has a lot of new code:

<div class="filename">Controllers/AccountController.cs</div>

```csharp
[HttpPost]
public async Task<ActionResult> Register (RegisterViewModel model)
{
    var user = new ApplicationUser { UserName = model.Email };
    IdentityResult result = await _userManager.CreateAsync(user, model.Password);
    if (result.Succeeded)
    {
        return RedirectToAction("Index");
    }
    else
    {
        return View();
    }
}
```

This method is now an `async Task` because creating user accounts will be an asynchronous action. Our `Register()` action doesn't return an `ActionResult`. Instead, it returns a `Task` containing an `ActionResult`. Remember, the built-in `Task` class represents asynchronous actions that haven't been completed yet.

The `Register()` action also takes a `model` of type `RegisterViewModel` as an argument. We will create a `RegisterViewModel` in the next lesson and explain its purpose. Do not worry about it just yet.

Next, we create a new `ApplicationUser` with the `Email` from the form submission as its `UserName`.

We're now ready for our async method:

```csharp
IdentityResult result = await _userManager.CreateAsync(user, model.Password);
```

Remember that we injected Identity's `UserManager` as a service. This class has a method called `CreateAsync()`. As explained in [the documentation](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.usermanager-1.createasync?view=aspnetcore-5.0), this method will create a user with the provided password.

Our async method will return a new `IdentityResult` object which we call `result`. [The `IdentityResult` class](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.identityresult?view=aspnetcore-5.0) simply represents the result of an Identity-driven action whether it's successful or not.

We use `await` because `CreateAsync()` is an asynchronous action, which means our application needs to wait until `CreateAsync()` successfully returns an `IdentityResult` before we actually define `result`.

Note that `CreateAsync()` takes two arguments:

* An `ApplicationUser` with user information;
* A password that will be encrypted when the user is added to the database.

What if a user tries to create an account with an email that's already been taken or leaves the password field blank? Identity's default settings for passwords requires at least six characters, a capital letter, a digit, and a special character. We'll learn how to customize this configuration in a future lesson.

Because our `IdentityResult` object already contains information about whether or not Identity was successful in registering the new user account, we add an `if` statement. If `CreateAsync()` is successful, the controller redirects to `Index`. Otherwise it returns the `Register` view.

In this lesson, we've added an `AccountController` that includes Identity via dependency injection. We use `async` and `await` to use Identity's built-in `CreateAsync()` method.

In the next lesson, we'll create the necessary views for users to register an account along with a `ViewModel` that will allow our form to play nicely with Identity.

### Repository Reference

Follow the link below to view how a sample version of the project should look at this point. Note that this is a link to a specific commit in the repository.

**[<i class="glyphicon glyphicon-folder-open"></i> Example GitHub Repo for To Do List](https://github.com/epicodus-lessons/c-sharp-to-do-list-dotnet-5-week-5/tree/3_identity_controller)**


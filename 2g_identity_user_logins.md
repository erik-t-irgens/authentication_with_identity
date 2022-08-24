Now that users can successfully register for new accounts, let's add the logic necessary for them to sign in and out of their accounts.

## Logging In
---

First, let's walk through the process of logging a user in.

### ViewModel

Our login page will contain a form asking for the user's email and password. We'll create the necessary `ViewModel` in the `ViewModels` folder to manage this:

<div class="filename">ViewModels/LoginViewModel.cs</div>

```csharp
namespace ToDoList.ViewModels
{
    public class LoginViewModel
    {
        public string Email { get; set; }
        public string Password { get; set; }
    }
}
```

Again, notice we follow naming conventions by including the `ViewModel` prefix at the end of our View Model's filename.

### View

Next, we'll create the `View` containing our login form:

<div class="filename">Views/Account/Login.cshtml</div>

```html
@{
  Layout = "_Layout";
}

@using ToDoList.ViewModels

@model LoginViewModel

<h2>Log in with your account</h2>
<hr />
@using (Html.BeginForm())
{
    @Html.LabelFor(m => m.Email)
    @Html.TextBoxFor(m => m.Email)

    @Html.LabelFor(m => m.Password)
    @Html.PasswordFor(m => m.Password)

    <input type="submit" value="Log in" class="btn btn-default" />
}
```

### Controller Actions

Now we can add the necessary actions to the controller. We'll include a `GET` to retrieve and display the Login `View` in addition to a `POST` to go through the process of actually logging a user in once they submit the login form.

Let's take a look at the full code and then we'll explain it in detail.

<div class="filename">AccountController.cs</div>

```csharp
...
public ActionResult Login()
{
    return View();
}

[HttpPost]
public async Task<ActionResult> Login(LoginViewModel model)
{
    Microsoft.AspNetCore.Identity.SignInResult result = await _signInManager.PasswordSignInAsync(model.Email, model.Password, isPersistent: true, lockoutOnFailure: false);
    if (result.Succeeded)
    {
        return RedirectToAction("Index");
    }
    else
    {
        return View();
    }
}
...
```

We'll focus on the `Login()` POST method which once again uses an asynchronous method. Note that there are several similarities with our `Register()` POST method:

* Both methods are `async` and return a `Task<ActionResult>`.
* Both take a `ViewModel` as an argument.
* Both use an Identity method ending with `Async`. All async Identity methods have `Async` prepended to them.
* Both methods have a `result` that must `await` the completion of an Identity method.

Now let's take a closer look at the following line:

```csharp
Microsoft.AspNetCore.Identity.SignInResult result = await _signInManager.PasswordSignInAsync(model.Email, model.Password, isPersistent: true, lockoutOnFailure: false);
```

Remember that we've injected a `SignInManager` service, which is being referenced in the `signInManager` variable. The `SignInManager` class includes the `PasswordSignInAsync()` method, which has a self-explanatory name: it's an async method that allows users to sign in with a password.

`PasswordSignInAsync()` takes four parameters: `userName`, `password`, `isPersistent` and `lockoutOnFailure`. For now we're only handling username and password, so we set explicit boolean values for `isPersistent` and `lockoutOnFailure`.

However, just like in our `Register()` action, we want to ensure our application doesn't freeze or break if Identity can't successfully authenticate an account. That's why we add an `if` statement based on whether the `result` has succeeded or not. The `Microsoft.AspNetCore.Identity.SignInResult` object has a `Succeeded` boolean property to help with this.

### Displaying Login Confirmation

Since the action will redirect to _Index_ if the user successfully logs in, let's add some code to `Index.cshtml` to verify that it works and to inform users they're now logged in:

<div class="filename">Views/Account/Index.cshtml</div>

```html
@{
  Layout = "_Layout";
}

@using System.Security.Claims

<h2>Authentication with Identity</h2>
<hr />
@if (User.Identity.IsAuthenticated)
{
    <p>Hello @User.Identity.Name!</p>
}
else
{
    <p>@Html.ActionLink("Register", "Register")</p>
    <p>@Html.ActionLink("Log in", "Login")</p>
}
```

We added an `if/else` statement to display a greeting to the user if they are logged in. If they aren't, we display links to register or log in.

Note that we need to add `@using System.Security.Claims` in order to access the `User.Identity` properties `IsAuthenticated` and `Name`.

## Logging Out
---

Next, let's give the user the ability to log out.

### Controller Action

Add a `LogOff()` action to the controller.

<div class="filename">AccountController.cs</div>

```csharp
...
[HttpPost]
public async Task<ActionResult> LogOff()
{
    await _signInManager.SignOutAsync();
    return RedirectToAction("Index");
}
...
```

This method is straightforward. `SignInManager` has the asynchronous method `SignOutAsync()` that signs the user out. Everything else in this method should look familiar at this point.

### View

Finally, we'll add a logout button to Index:

<div class="filename">Views/Account/Index.cshtml</div>

```html
...
@if (User.Identity.IsAuthenticated)
{
    <p>Hello @User.Identity.Name!</p>
    @using (Html.BeginForm("LogOff", "Account"))
    {
        <input type="submit" class="btn btn-default" value="Log out" />
    }
}
...
```

We now have complete functionality for users to sign up, sign in and sign out.

### Repository Reference

Follow the link below to view how a sample version of the project should look at this point. Note that this is a link to a specific commit in the repository.

**[<i class="glyphicon glyphicon-folder-open"></i> Example GitHub Repo for To Do List](https://github.com/epicodus-lessons/c-sharp-to-do-list-dotnet-5-week-5/tree/5_identity_user_logins)**
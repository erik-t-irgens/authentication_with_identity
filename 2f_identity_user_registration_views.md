In the last lesson, we added a controller for creating new user accounts. In this lesson, we'll create the corresponding views as well as our first `ViewModel`. By the end of the lesson, we'll be able to create new users in the database via a registration form.

## ViewModels
---

When we deal with data that only shows up in the view, we can use a **ViewModel** instead of a Model. This allows us to specify which fields we want to collect from our view. Since we don't need to collect information for all the properties built into `ApplicationUser`, we'll create a `ViewModel` that only contains the properties we need.

Create a `ViewModels` folder in the project directory and add the class `RegisterViewModel.cs` to this folder. Add the following code:

<div class="filename">ViewModels/RegisterViewModel.cs</div>

```csharp
using System.ComponentModel.DataAnnotations;

namespace ToDoList.ViewModels
{
    public class RegisterViewModel
    {
        [Required]
        [EmailAddress]
        [Display(Name = "Email")]
        public string Email { get; set; }

        [Required]
        [DataType(DataType.Password)]
        [Display(Name = "Password")]
        public string Password { get; set; }

        [DataType(DataType.Password)]
        [Display(Name = "Confirm password")]
        [Compare("Password", ErrorMessage = "The password and confirmation password do not match.")]
        public string ConfirmPassword { get; set; }
    }
}
```

While this `ViewModel` may look different from other classes at first, it's really just a grouping of properties and data annotations. Now we have a data model we can use for user registration.

Also, note that the file name contains `ViewModel` as a suffix. It's standard naming convention to end each `ViewModel`'s file name with `ViewModel`.

## Registration View
---

Now let's take care of the `Register` view, which will include a form asking the user to enter an email address, password, and a confirmation password. Add a new directory in `Views` called `Account` and add a `Register.cshtml` file to the new directory with the following code:

<div class="filename">Views/Account/Register.cshtml</div>

```html
@{
  Layout = "_Layout";
}

@using ToDoList.ViewModels

@model RegisterViewModel

<h2>Register a new user</h2>
<hr />
@using (Html.BeginForm("Register", "Account", FormMethod.Post))
{
    @Html.LabelFor(user => user.Email)
    @Html.TextBoxFor(user=> user.Email)

    @Html.LabelFor(user=> user.Password)
    @Html.PasswordFor(user=> user.Password)

    @Html.LabelFor(user=> user.ConfirmPassword)
    @Html.PasswordFor(user=> user.ConfirmPassword)

    <input type="submit" value="Register" />
}
```

This code is straightforward. In addition to the standard code we've added to forms so far, we use the HTML helper method `PasswordFor` for our `Password` and `ConfirmPassword` fields.

Finally, let's add a view for our `Index()` route:

<div class="filename">Views/Account/Index.cshtml</div>

```html
@{
  Layout = "_Layout";
}

<h2>Authentication with Identity</h2>
<hr />
<p>@Html.ActionLink("Register", "Register")</p>
```

Let's also add a link to the account index in our homepage:

<div class="filename">Views/Home/Index.cshtml</div>

```html
...
<p>@Html.ActionLink("Create an account", "Index", "Account")</p>
```

We now have everything we need to create user accounts. Run the app and click on the link in the homepage to register an account. Note that Identity's default setting for a password is at least six characters, a capital letter, a digit, and a special character.

If everything is successful, we'll be directed back to _Index_. Navigate to the database in MySQLWorkbench and right-click on _AspNetUsers_. Click on _Select Rows — Limit 1000_. Our database should now include a new user.

In the next lesson, we'll integrate logic to allow users to sign in with their newly-created accounts.

### Repository Reference

Follow the link below to view how a sample version of the project should look at this point. Note that this is a link to a specific commit in the repository.

**[<i class="glyphicon glyphicon-folder-open"></i> Example GitHub Repo for To Do List](https://github.com/epicodus-lessons/c-sharp-to-do-list-dotnet-5-week-5/tree/4_identity_views)**


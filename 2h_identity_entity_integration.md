We now have a working user login and registration system, but it doesn't actually have any kind of impact on our application. Our users can do all CRUD regardless of whether they are signed in or not.

## Authorization with Identity
---

In this lesson, we'll integrate Identity more fully into our application by adding in authorization. Remember, **authorization** is the process of managing what a user is allowed to do. We'll update our to do list application so that only logged in users will be able to see their own lists. This is similar to many real-world applications such as email or blog sites where a signed-in user has access to their own content.

Note that we will not modify categories in this lesson and that by the end of this lesson, authorization will only work for creating and viewing a user's tasks.

### Updating the Model

First, let's add a property to our `Item.cs` model, which should then look like this:

<div class="filename">Models/Item.cs</div>

```csharp
using System.Collections.Generic;

namespace ToDoList.Models
{
    public class Item
    {
        public Item()
        {
            this.JoinEntities = new HashSet<CategoryItem>();
        }

        public int ItemId { get; set; }
        public string Description { get; set; }
        public virtual ApplicationUser User { get; set; } //new line

        public virtual ICollection<CategoryItem> JoinEntities { get;}
    }
}
```

The `User` property is declared `virtual` to allow Entity to lazy load its contents, improving our application's efficiency.

### Updating the Controller

Next, let's update our `ItemsController`. We'll start by adding the following `using` statements:

```csharp
...
//new code
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using System.Threading.Tasks;
using System.Security.Claims;
...
```

* `Microsoft.AspNetCore.Authorization` will allow us to actually authorize users.
* We'll need `Microsoft.AspNetCore.Identity` so our controller can interact with users from the database.
* `System.Threading.Tasks` will be necessary to call async methods.
* `System.Security.Claims` is important for using **claim based authorization**. A **claim** is a form of user identification. It states who a user is, not what the user can actually do. While the user identification itself doesn't authorize a user to do anything, it is necessary to first identify a user (through a claim) in order to determine whether they should be authorized.

Next, we'll add an attribute, a property, and update our constructor to account for the new functionality:

<div class="filename">Controllers/ItemsController.cs</div>

```csharp
...
namespace ToDoList.Controllers
{
  [Authorize] //new line
  public class ItemsController : Controller
  {
    private readonly ToDoListContext _db;
    private readonly UserManager<ApplicationUser> _userManager; //new line

    //updated constructor
    public ItemsController(UserManager<ApplicationUser> userManager, ToDoListContext db)
    {
      _userManager = userManager;
      _db = db;
    }
...
```

Let's break this code down into smaller pieces.

First, note that we include an `[Authorize]` attribute on `ItemsController`:

```csharp
...
[Authorize]
public class ItemsController : Controller
...
```

This allows access to the `ItemsController` only if a user is logged in. We'll add this attribute to a controller whenever we want to limit its access to signed-in users. 

**This is just one application of authorization.** 

In this scenario, the entirety of the controller is shielded from unauthorized users. We can negate this by including an `[AllowAnonymous]` attribute above any specific methods that we want unauthorized users to have access to. For example, we could put `[AllowAnonymous]` above the `Index` route, if we want users to be able to see a list of items, but require authorization before they view details. Alternatively, we could avoid putting the `[Authorize]` attribute on the entire class, and instead only place it on specific methods we want guarded. For example, if we wanted unauthorized users to view many routes in a controller, but protect your `Create`, `Update`, and `Delete` routes, you could simply `[Authorize]` those specific methods.

For the purposes of this lesson, let's continue with the `[Authorize]` route on the entire controller, as shown.

Next, let's take a look at our `readonly` property and our constructor:

<div class="filename">Controllers/ItemsController.cs</div>

```csharp
...
private readonly UserManager<ApplicationUser> _userManager;

public ItemsController(UserManager<ApplicationUser> userManager, ToDoListContext database)
{
  _userManager = userManager;
  _db = database;
}
...
```

This code should look more familiar now because our `AccountController` has the exact same code. We need an instance of `UserManager` to work with signed-in users. We also include a constructor to instantiate private `readonly` instances of the database and the `UserManager`.

Next, let's update the `Index` and `Create` action methods to utilize the new `UserManager`.

### `Index()` Action

Let's update our  `Index()` method:

<div class="filename">Controllers/ItemsController.cs</div>

```csharp
...
public async Task<ActionResult> Index()
{
    var userId = this.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    var currentUser = await _userManager.FindByIdAsync(userId);
    var userItems = _db.Items.Where(entry => entry.User.Id == currentUser.Id).ToList();
    return View(userItems);
}
...
```

There's a lot to unpack here.

* We start by using the `async` modifier because this action will run asynchronously. Because the action is asynchronous, it also returns a `Task` containing an action result.

* Then we locate the unique identifier for the currently-logged-in user and assign it the variable name `userId`. Let's go over the new logic in the following line of code:

```csharp
var userId = this.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
```

* `this` refers to the `ItemController` itself.
* `FindFirst()` is a method that locates the first record that meets the provided criteria.
* This method takes `ClaimTypes.NameIdentifier` as an argument. This is why we need a `using` directive for `System.Security.Claims`. We specify `ClaimTypes.NameIdentifier` to locate the unique ID associated with the current account. `NameIdentifier` is a property that refers to an Entity's unique ID. See [the ClaimTypes documentation](https://docs.microsoft.com/en-us/dotnet/api/system.security.claims.claimtypes?view=net-5.0) for more information on this class.
* Finally, we include the `?` operator after the line `this.User.FindFirst(ClaimTypes.NameIdentifier)`. This is called an **existential operator**. It states that we should only call the property to the right of the `?` if the method to the left of the `?` doesn't return `null`. Essentially, the code states that if `this.User.FindFirst(ClaimTypes.NameIdentifier)` returns `null`, don't call the property to the right of the existential operator. However, if it doesn't return `null`, it retrieves `Value` property.
* In other words, if we successfully locate the `NameIdentifier` of the current user, we'll call `Value` to retrieve the actual unique identifier value.

Once we have the `userId` value, we're ready to call our async method:

```csharp
var currentUser = await _userManager.FindByIdAsync(userId);
```

* First we call the `UserManager` service that we've injected into this controller.
* We call the `FindByIdAsync()` method, which, as its name suggests, is a built-in Identity method used to find a user's account by their unique ID.
* We provide the `userId` we just located as an argument to `FindByIdAsync()`.
* Thanks to the handy `Async` suffix in this methods name, we know it runs asynchronously. We include the `await` keyword so the code will wait for Identity to locate the correct user before moving on.

Finally, we create a variable to store a collection containing only the `Item`s that are associated with the currently-logged-in user's unique `Id` property:

```csharp
...
var userItems = _db.Items.Where(entry => entry.User.Id == currentUser.Id).ToList();
return View(userItems);
...
```

We use the `Where()` method, which is a LINQ method we can use to query a collection in a way that echoes the logic of SQL. We can use `Where()` to make many different kinds of queries, as the method accepts an expression to filter our results.

In this case, we're simply asking Entity to find items in the database where the user id associated with the item is the same id as the id that belongs to the `currentUser`. This ensures users only see their own tasks in the view.

### `Create()` POST Action

Let's now edit our `Create()` action. Make sure you update the **post** method and not the **get** method.

<div class="filename">Controllers/ItemsController.cs</div>

```csharp
[HttpPost]
public async Task<ActionResult> Create(Item item, int CategoryId)
{
    var userId = this.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    var currentUser = await _userManager.FindByIdAsync(userId);
    item.User = currentUser;
    _db.Items.Add(item);
    _db.SaveChanges();
    if (CategoryId != 0)
    {
        _db.CategoryItem.Add(new CategoryItem() { CategoryId = CategoryId, ItemId = item.ItemId });
    }
    _db.SaveChanges();
    return RedirectToAction("Index");
}
```

The first two lines of this action are exactly the same as the first two lines of our `Index()` action. We start by finding the value of the current user. Then we associate the current user with the `Item`'s `User` property. This makes the association so that an `Item` belongs to a `User`. Finally, we add the item to the database and save it as we did before.


### Updating Views for Authorization

Finally, let's update our views. We'll start with the updated `Index.cshtml` for _Items_:

<div class="filename">Views/Items/Index.cshtml</div>

```html
@{
  Layout = "_Layout";
}

@using ToDoList.Models;
@model IEnumerable<ToDoList.Models.Item>

<h1>Items for @User.Identity.Name</h1>

@if (Model.Any())
{
  <ul>
    @foreach (Item item in Model)
    {
      <li>@Html.ActionLink($"{item.Description}", "Details", new { id = item.ItemId })</li>
    }
  </ul>
} 
else
{
  <h3>No items have been added yet!</h3>
}

<p>@Html.ActionLink("Add new item", "Create")</p>

<p>@Html.ActionLink("Home", "Index", "Home")</p>
```

Because we use `System.Security.Claims`, we'll be redirected to the _Account/Login_ view if we aren't logged in.

We also use the method `Any()` instead of `Count` with our `if` statement and switch the branching logic around. If the `Model` includes any `Item`s, this will return true, and we'll loop through our list. Otherwise, the statement will return false and we'll display the "no items added" message.

Here's the view for our form:

<div class="filename">Views/Items/Create.cshtml</div>

```html
@{
  Layout = "_Layout";
}

@model ToDoList.Models.Item

<h4>Add a new task</h4>

@using (Html.BeginForm())
{
    @Html.LabelFor(model => model.Description)
    @Html.TextBoxFor(model => model.Description)

    @Html.Label("Select category")
    @Html.DropDownList("CategoryId")

    <input type="submit" value="Add new item" class="btn btn-default" />
}
<p>@Html.ActionLink("Show all items", "Index")</p>
```

Note that it looks exactly the same as it did before. Our controller handles the association between an `Item` and a `User` and the form has nothing to do with it.

### Running the application

Before we run the application, there are a couple of other things we need to do. First, let's add a link at the bottom of the Account Index view to the homepage to ease traversal:

```html
...
<p>@Html.ActionLink("Home", "Index", "Home")</p>
```

Next, we'll need to add a new migration and update the database. When we added the `User` property to our `Item` model, we dictated a change to the expected database schema but did not update the database to reflect this change. In our project, let's run the following command:

```bash
$ dotnet ef migrations add Authorization
```

This command will add a new migration and its corresponding designer file to our Migrations directory while updating the model snapshot for our to do list context. Now, let's update the database:

```bash
$ dotnet ef database update
```

We can now run our application, log in as a user, and create and view our own items.

### Repository Reference

Follow the link below to view how a sample version of the project should look at this point. Note that this is a link to a specific commit in the repository.

**[<i class="glyphicon glyphicon-folder-open"></i> Example GitHub Repo for To Do List](https://github.com/epicodus-lessons/c-sharp-to-do-list-dotnet-5-week-5/tree/6_integrating_entity_and_identity)**

During Intermediate JavaScript, we learned how to write asynchronous code and manage asynchronicity while performing complex actions like API calls. In this lesson, we'll discuss how C# and .NET handle asynchronous code.

Why is this relevant now? In the next few lessons, we'll learn how to authenticate users with Identity. This will require our applications to manage asynchronous actions, so we need to learn how to recognize and write asynchronous methods.

## Synchronous Operations
---

So far the C# code we've written is **synchronous** and our code executes a single line at a time. The `Edit()` action is an example of a synchronous method:

```csharp
...
public ActionResult Edit(int id)
{
    Item thisItem = db.Items.FirstOrDefault(item => item.ItemId == id);
    return View(thisItem);
}
...
```

Let's walk through what occurs when this synchronous method is called:

1. When `Edit()` is called, .NET looks in the database to find a specific item.

2. After locating the item, it `return`s the `Edit()` view.

3. While .NET is looking in the database, the browser waits. It cannot return the view until `thisItem` is found because the line of code attempting to locate `thisItem` appears _before_ we return the view.

4. When `thisItem` is located, the application can return the view to be displayed in the browser.

Because the lines of code are executed in order, this is a **synchronous** operation.

## Asynchronous Operations
---

On the other hand, an **asynchronous** operation allows other code to run while a method is waiting to return.
This is very similar to what we learned in JavaScript. However, the code we write to manage this process looks much different.

There are three primary parts to an async C# method:

1. The `async` keyword

2. The `await` keyword

3. A special `Task` class that represents an action or actions the program may not have completed yet because they're async.

Let's walk through an example async method:

```csharp
static async void ProcessTextFileAsync()
{
  Task<int> task = ExampleAsyncMethodThatTakesAWhile();

  Console.WriteLine("Please wait patiently while I run the ExampleAsyncMethodThatTakesAWhile().");

  int x = await task;

  Console.WriteLine("Return value of ExampleAsyncMethodThatTakesAWhile(): " + x);
}
```

1. First, the `async` keyword is included in the method's signature. Async methods should always have an `async` modifier in their declaration. It tells our application the method should run asynchronously.

2. Next, notice the `await` keyword in the line `int x = await task`. As its name suggests, this keyword makes the application wait until the specified async action `task` is completed. Once `task` is completed, it will define `x`. This gives us a great deal of control over our code. We can allow multiple lines of code to run asynchronously and then add multiple "waiting points" with `await`.

3. Finally, our hypothetical async method `ExampleAsyncMethodThatTakesAWhile()` has a return type of `Task<int>`. This line stores the return in the variable `task`. In C#, **an asynchronous method can only return `void` or a `Task` object representing the asynchronous operation itself.** A `Task` represents ongoing work. While a `Task` is `void` by default, there are a few different options available, including the following:
  * `Task<int>` returns an `int`;
  * `Task<string>` returns a `string`;
  * `Task<IActionResult>` returns an `ActionResult`.
  
In the code above we want the result to be the integer `x`. In order to turn a `Task<int>` into an `int`, we use the `await` keyword. This forces the program to wait until `task` is appropriately defined as an `int` before moving on to subsequent lines of code.

Let's consider one more pseudocode example:

```csharp
static async void DoMyHolidayErrandsForMeAsync()
{
  Task<int> determineHowManyGiftsIShouldBuy = processSantasList("C:\\naughty_or_nice.txt");
  BakeSugarCookies();
  HangLights();
  int giftNumber = await determineHowManyGiftsIShouldBuy;
  for (int index = 0; index < giftNumber.Length; index++)
  {
    BuyHolidayGifts();
    MarkOffList(index);
  }
}
```

Here, we have an `async` method called `DoMyHolidayErrandsForMeAsync()`. We could describe the method as doing the following: "I need you to do holiday errands. Start determining how many gifts we need to buy and feel free to multi-task and bake the cookies and hang lights while you're doing that, too. But **wait** to define `giftNumber` until after the `processSantasList()` method fully finishes because we need its results before continuing. After we have that number, we can buy the necessary number of gifts and start marking them off our list."

### Rules and Conventions

Before we wrap up, let's review the rules and conventions for async methods in C#:

* Async methods always have an `async` modifier in their declaration, as seen above.

* It's also a best practice to include the term `Async` at the end of the method name as seen in the `ProcessTextFileAsync()` and `DoMyHolidayErrandsForMeAsync()` methods.

* Again, an asynchronous method can only return `void` or a `Task` object representing the asynchronous operation itself.

* However, there are different types of `Task`s to choose from, including `Task<int>`, `Task<string>` returns a `string`, and `Task<IActionResult>`.

* The `await` keyword can **only** be used in a method that includes the `async` modifier in its signature.

* Because `await` waits for a `Task` to finish and return its specified data type, we can use it to "unwrap" a `Task`. For instance, if we `await` something declared as a `Task<int>`, we'll receive an `int` value, because that's what that `Task` returns when its async code is finished running. Similarly, if we use `await` on a `Task<string>`, we'd receive a string value.

## Additional Resources
---

* Check out [this article from dotnetperls](http://www.dotnetperls.com/async) for some examples of async/await methods in use.

* For more information about the `Task` class, we recommend the [msdn documentation](https://msdn.microsoft.com/en-us/library/system.threading.tasks.task.aspx).

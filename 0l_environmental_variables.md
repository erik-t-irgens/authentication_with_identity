Over the last several lessons, we've learned to make an API call. However, there's one more thing we need to do before we move on. Our API key is currently embedded in the code itself. This means that anyone that looks at the repository can see our credentials. Even in a closed-source project, it would be easy for contractors or disgruntled employees to walk off with access to services they shouldn't.

There are a lot of ways to keep sensitive information separate from the rest of an app. It's a common practice to store this information in a separate file and then put that file in a `.gitignore` file before committing or deploying any code. We practiced this process when working with API keys in our JavaScript course previously. We can do something similar in our .NET apps using **environmental variables**.

## Environment Variables
---

Let's walk through how we can hide sensitive credentials with environmental variables. We'll continue with the project we've been working on over the past several lessons.

First, we need to create a file called `EnvironmentVariables.cs`. This file will contain a class with our sensitive passwords:

<div class="filename">Models/EnvironmentVariables.cs</div>

```csharp
namespace MvcApiCall.Models
{
    public static class EnvironmentVariables
    {
        public static string ApiKey = "[YOUR-API-KEY-HERE]";
    }
}
```

Next, we'll remove our API key from `HomeController.cs` and replace it with the value we defined in `EnvironmentalVariables`:

<div class="filename">Models/HomeController.cs</div>

```csharp
using Microsoft.AspNetCore.Mvc;
using MvcApiCall.Models;

namespace MvcApiCall.Controllers
{
  public class HomeController : Controller
    {
        public IActionResult Index()
        {
            var allArticles = Article.GetArticles(EnvironmentVariables.ApiKey);
            return View(allArticles);
        }
    }
}
```

## Ignoring Sensitive Data
---

Finally, we need to ensure that Git doesn't track our `EnvironmentVariables`. We can do this by adding a `.gitignore` file to our project that lists our `EnvironmentVariables.cs` file:

<div class="filename">.gitignore</div>

```gitignore
EnvironmentVariables.cs
```

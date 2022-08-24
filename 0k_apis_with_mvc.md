Now that we know how to turn an API response into objects, let's look at how we can integrate this into an MVC application. We're going to create a simple MVC app that will allow users to see basic information about the top headlines on the New York Times.

## Project Setup
---

Create a new directory called `MvcApiCall`. You may use the `dotnet new mvc` command or you build the application from scratch.

Next, let's add RestSharp and NewtonSoft.Json:

```bash
$ dotnet add package RestSharp --version 106.6.10
$ dotnet add package Newtonsoft.Json --version 12.0.2
```

## `Article` Class
---

We'll start by creating an `Article` class similar to the one in our console application. In order to actually make an API call, we'll translate part of our console program into a `GetArticles()` method inside the `Article` class:

<div class="filename">Models/Article.cs</div>

```csharp
using System.Collections.Generic;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

namespace MvcApiCall.Models
{
  public class Article
  {
    public string Section { get; set; }
    public string Title { get; set; }
    public string Abstract { get; set; }
    public string Url { get; set; }
    public string Byline { get; set; }

    public static List<Article> GetArticles(string apiKey)
    {
      var apiCallTask = ApiHelper.ApiCall(apiKey);
      var result = apiCallTask.Result;

      JObject jsonResponse = JsonConvert.DeserializeObject<JObject>(result);
      List<Article> articleList = JsonConvert.DeserializeObject<List<Article>>(jsonResponse["results"].ToString());

      return articleList;
    }
  }
}
```

Next, we'll create an `ApiHelper` class similar to the one we created in the last lessons:

<div class="filename">Models/ApiHelper.cs</div>

```csharp
using System.Threading.Tasks;
using RestSharp;

namespace MvcApiCall.Models
{
  class ApiHelper
  {
    public static async Task<string> ApiCall(string apiKey)
    {
      RestClient client = new RestClient("https://api.nytimes.com/svc/topstories/v2");
      RestRequest request = new RestRequest($"home.json?api-key={apiKey}", Method.GET);
      var response = await client.ExecuteTaskAsync(request);
      return response.Content;
    }
  }
}
```



This code should all be familiar from the last two lessons.

## Controller and Views
---

We'll keep the controller actions very simple. We'll just include an index route.

<div class="filename">Controllers/HomeController.cs</div>

```csharp
using Microsoft.AspNetCore.Mvc;
using MvcApiCall.Models;

namespace MvcApiCall.Controllers
{
  public class HomeController : Controller
    {
        public IActionResult Index()
        {
            var allArticles = Article.GetArticles("[YOUR-API-KEY-HERE]");
            return View(allArticles);
        }
    }
}

```

Note that because `GetArticles()` is a static method, it's called on the `Article` class itself.

And here's our view:

<div class="filename">Views/Home/Index.cshtml</div>

```html
<h1>All Articles:</h1>

<ol>
@foreach (Article article in Model)
{
    <ul>
        <li>Section: @article.Section</li>
        <li>Title: @article.Title</li>
        <li>Abstract: @article.Abstract</li>
        <li>Url: @article.Url</li>
        <li>Byline: @article.Byline</li>
    </ul>
    <br>
}
</ol>
```

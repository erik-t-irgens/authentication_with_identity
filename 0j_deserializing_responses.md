If we were making our API console application for users, they wouldn't want to copy and paste the response of an API call into a JSON formatter. It's our job to clean up the data and make it more human-readable. We can do this by **deserializing** the data. We can think of serialization as the process of turning data into a format that can be streamed while deserialization is the process of returning that data back to its original state. In this case, we're using deserialization to better make sense of the data and make it easier for users to read. Later in the lesson, we'll complete the deserialization process and learn how to convert JSON data into C# objects.

First, we need to include the `Newtonsoft.Json` NuGet package, which is a JSON framework for .NET that can be used for both serialization and deserialization. We'll install it using the CLI:

```bash
$ dotnet add package Newtonsoft.Json --version 12.0.2
```

Add these two `using` directives to `Program.cs`:

<div class="filename">Program.cs</div>

```csharp
...
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
...
```

Now we can turn our API response into a JSON object to deal with the messages directly. Replace the last line of the `Main()` method with the following code:

<div class="filename">Program.cs</div>

```csharp
...
static void Main(string[] args)
{
    var apiCallTask = ApiHelper.ApiCall("[YOUR-API-KEY-HERE]");
    var result = apiCallTask.Result;
    // This line should be deleted: Console.WriteLine(result);
    
    JObject jsonResponse = JsonConvert.DeserializeObject<JObject>(result);
    Console.WriteLine(jsonResponse["results"]);
}
...
```

Here, we turn the giant string stored as `result` into JSON data.

* `JsonConvert.DeserializeObject()<JObject>(result)` converts the JSON-formatted string `result` into a `JObject`.
* The type `JObject` comes from the `Newtonsoft.Json.Linq` library and is a .NET object we can treat as JSON.

Now we have access to the data stored in the `"results"` key. All we have to do is call `jsonResponse["results"]`. If we run the program again, the response will be nicely formatted and will include everything stored in the `"results"` key.

Notice that each key in the JSON response is in lower snake-case, not the PascalCase commonly used in C#. When retrieving data from a JSON response, always make sure to use the name of the JSON key with the exact casing it appears in the response object.

## Transforming JSON into C# Objects
---

Now that we have all this data about the messages we've sent, we might want to transform it into C# objects that can be used in more complex programs. The `DeserializeObject()` method will do this for us.

Let's start by creating a `Article` class that includes all the properties of the news we might want to use in an application. Note that we won't use all of the keys that the NYTimes provides — instead, we'll use just enough to demonstrate the process of transforming the JSON data into C# objects.

<div class="filename">Article.cs</div>

```csharp
...
namespace ApiTest
{
    public class Article
    {
        public string Section { get; set; }
        public string Title { get; set; }
        public string Abstract { get; set; }
        public string Url { get; set; }
        public string Byline { get; set; }
    }
...
```

Now we can turn the JSON data about our messages into `Article` objects. Note that we need to add `System.Collections.Generic` since we will be using `List`s.

<div class="filename">Program.cs</div>

```csharp
...
using System.Collections.Generic;
...
        static void Main()
        {
            var apiCallTask = ApiHelper.ApiCall("[YOUR-API-KEY-HERE]");
            var result = apiCallTask.Result;

            JObject jsonResponse = JsonConvert.DeserializeObject<JObject>(result);
            List<Article> articleList = JsonConvert.DeserializeObject<List<Article>>(jsonResponse["results"].ToString());

            foreach (Article article in articleList)
            {
                Console.WriteLine($"Section: {article.Section}");
                Console.WriteLine($"Title: {article.Title}");
                Console.WriteLine($"Abstract: {article.Abstract}");
                Console.WriteLine($"Url: {article.Url}");
                Console.WriteLine($"Byline: {article.Byline}");
            }
        }
...
```

If we run our program, we'll see that the response is nicely formatted and includes only the properties listed above.

Now let's take a closer look at the code. We use the `DeserializeObject()` method to create a list of `Article`s. The method will automatically grab any JSON keys in our response that match the names of the properties in our class. In order for this to work, the property name has to match the JSON key. This means that the `Section` property for our message class needs to be named `Section`. We could not rename it to something like `Category` because the information is named `"section"` in the JSON data.

Because the JSON keys must match the object property names, we'll sometimes need to go deeper than depicted in the example above before turning API response data into a C# object. For instance, if the response information we'd like to deserialize is contained in an object that's part of an array which is nested under a specific JSON key, we'd need to programmatically isolate that specific data whose keys match our object properties before converting it into an object. We can't just pass in the whole big response. Fortunately, we can always use Postman to take a closer look at how JSON key-value pairs are nested.

In this lesson, we've learned how to make an API call and then parse and deserialize the results. There's a lot of new information here, including new ways to use tasks and async code. It may take some time to absorb this material, but for now the important thing is putting the pieces together to successfully make API calls. In the next lesson, we'll learn how to incorporate our new tools into an MVC application.

### Repository Reference

Follow the link below to view how a sample version of the project should look at this point. Note that this is a link to a specific commit in the repository.

**[<i class="glyphicon glyphicon-folder-open"></i> Example GitHub Repo for Sample New York Times API Call](https://github.com/epicodus-lessons/c-sharp-console-api-call/tree/2_deserializing_responses)**
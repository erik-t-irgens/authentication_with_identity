In this lesson, we'll learn how to make an API call with a console application. We will start with a console application instead of MVC because the process of making an API call and making sense of the response is fairly involved. We'll use the New York Times' Top Stories API along with a tool called [RestSharp](http://restsharp.org/) to make HTTP requests. Read the ["Getting Started"](https://github.com/restsharp/RestSharp/wiki/Getting-Started) page of the RestSharp documentation to get the basic idea of how it works.

In order to use the Top Stories API, you'll need to create a free [New York Times developer account](https://developer.nytimes.com/). Follow the [Get Started](https://developer.nytimes.com/get-started) steps to create an application and get your own API key. We recommend doing this early as API keys can sometimes take a little while to become activated.

Let's start by creating a project directory named ConsoleApiCall. Next, we can add a new console application with the command:

```bash
$ dotnet new console
```

Next, Let's add the RestSharp package:

```bash
$ dotnet add package RestSharp --version 106.6.10
```

The `dotnet new` command has already added a basic `Program.cs` file for us. Let's update it to include the following code:

<div class="filename">Program.cs</div>

```csharp
using System;
using System.Threading.Tasks;
using RestSharp;

namespace ApiTest
{
  class Program
  {
    static void Main()
    {
      var apiCallTask = ApiHelper.ApiCall("[YOUR-API-KEY-HERE]");
      var result = apiCallTask.Result;
      Console.WriteLine(result);
    }
  }

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

Let's first take a look at the `ApiCall` static method that we've created:

* We create a class called `ApiHelper` that contains a static method `ApiCall` which takes an `apiKey` parameter.

* We want our API calls to run asynchronously so that the application is responsive and free to run other tasks while the HTTP request/response loop executes. In order to achieve this, we add the `async` keyword to our method declaration.

* Whenever a method is declared as `async`, we need to return a `Task` type. We specify the return type of our Task object (a `string`) in the angle brackets, but a generic `Task` can also be returned.

* Note that we use the base URL _https://api.nytimes.com/svc/topstories/v2_ from the Top Stories API. We instantiate a RestSharp `RestClient` object and store the connection in a variable called `client`.

* Next, we create a `RestRequest` object. This is our actual request. We include the path to the endpoint we are looking for (`home.json`) along with our API key.  We also specify that we will be using a GET Http method.

* Note that we utilize C#'s string interpolation to place the `apiKey` variable into the `RestRequest` by placing a `$` before a string and then placing any interpolated values in curly braces. This is really similar to how we did template literals in JavaScript.

* Then we use the `await` keyword to specify that we need to receive a result before we attempt to define `response`. We call the `RestClient`'s `ExecuteTaskAsync` method and pass in our request object.

* Finally, we return the `Content` property of the `response` variable, which is a string representation of the response content.

* In the `Main` function, we create a variable to store the returned `Task` from our `async` function and then call the `ApiHelper` class' `ApiCall` method. You will need to replace `[YOUR-API-KEY-HERE]` in the method call with your own API key.

* Then, we create a variable to store the `Result` of the `Task`, which in our case is a string representation of the API call's response content.

* Lastly, we write the result to the console.

Next, run the program with `dotnet run`. We'll get a long, very dense response that has all the data for the New York Times' top stories.

To make better sense of this data, we can paste the response into a JSON formatter like this [one](http://jsonviewer.stack.hu/). Copy the data into the formatter and then click the "Format" button. Of course, Postman can help us read this data, too.

However, this isn't an ideal way to deal with API responses. In the next lesson, we'll learn more about parsing and deserializing JSON data in our C# application.

### Repository Reference

Follow the link below to view how a sample version of the project should look at this point. Note that this is a link to a specific commit in the repository.

**[<i class="glyphicon glyphicon-folder-open"></i> Example GitHub Repo for Sample New York Times API Call](https://github.com/epicodus-lessons/c-sharp-console-api-call/tree/1_initial)**
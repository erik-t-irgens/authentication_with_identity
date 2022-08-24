In this lesson we'll introduce authentication in general as well as Identity, the authentication tool built into ASP.NET Core.

## Authentication
---

**Authentication** is the process of identifying users by matching sign-in credentials with user account data in the application. Whenever you log into an account through a website, that site or application is _authenticating_ you.

## Authorization
---

Keep in mind authentication differs from authorization. While authentication is the process of determining who you are, **authorization** is the process of managing what you are allowed to do. For example, an admin would have different permissions than a moderator or a basic user.

## Introduction to Identity
---

To support user accounts and authentication in our .NET applications, we'll use a popular tool called **Identity**. As explained in the Microsoft Article [Introduction to Identity on ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/identity?tabs=visual-studio&view=aspnetcore-5.0):

> ASP.NET Core Identity is a membership system which allows you to add login functionality to your application. Users can create an account and login with a user name and password or they can use an external login provider such as Facebook, Google, Microsoft Account, Twitter or others.

It's considered the recommended approach for managing user accounts in ASP.NET MVC Core applications. In fact, the `Microsoft.AspNetCore.Identity` is included with the framework.

In the next lesson, we'll begin walking through how to integrate Identity into our own ASP.NET MVC applications.

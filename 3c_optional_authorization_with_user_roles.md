Now that our application has users, we might want to consider adding different roles. For example, a site can have some users that can view pages, registered users who can leave comments, moderators who can ban certain users for leaving too many spam comments, administrators, and so on.

This is part of **user authorization**, which is the process of telling who a user is. Identity has a system in place for user authorization.

We've done some basic authorization with the `[Authorize]` attribute on a controller. However, this authorization is fairly limited. If a user is logged in, they will be able to access that controller. There are many use cases where we'll want more fine-grained control over authorization in our applications.

If you are interested in exploring roles further, check out the following [tutorial](https://docs.microsoft.com/en-us/aspnet/core/security/authorization/secure-data?view=aspnetcore-5.0).

You are not expected to incorporate roles for this section's independent project. However, if you are looking for a challenge, try incorporating them into your multi-day project.

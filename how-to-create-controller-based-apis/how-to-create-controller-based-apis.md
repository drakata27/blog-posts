# How to create controller based APIs in .NET Core

## Learn how to create a basic CRUD API using C# and .NET 9.0

![alt cover](https://aleksdraka-blog-posts.s3.eu-west-2.amazonaws.com/how-to-create-controller-based-apis/cover.jpeg "Cover")

### Introduction

In this blog post I will create a simple controller based API for a flash card application using .NET Core 9.0 and Rider as my IDE of choice. If you are using Visual Studio or VSCode you can follow the official Microsoft documentation from which this blog was inspired by clicking
[here](https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-web-api?view=aspnetcore-9.0&tabs=visual-studio "Microsoft Docs")

### Routes

| API                         | Description    | Request    | Body                 |
| --------------------------- | -------------- | ---------- | -------------------- |
| GET /api/flashcards         | Get all cards  | No         | Array of flash cards |
| GET /api/flashcards/{id}    | Get a card     | No         | Flash card           |
| POST /api/flashcards        | Add a new card | Flash card | Flash card           |
| PUT /api/flashcards/{id}    | Update a card  | Flash card | No                   |
| DELETE /api/flashcards/{id} | Delete a card  | No         | No                   |

### Project Setup

Open your IDE and create a new project. Then follow the steps below:

1. Name it **FlashCardApi**
2. Select **net9.0**
3. Select **Web API**
   ![alt new project](https://aleksdraka-blog-posts.s3.eu-west-2.amazonaws.com/how-to-create-controller-based-apis/pic1.png "New project")

4. After that go to advanced settings and uncheck **UseMinimalAPIs** (or select Controller API depending on your IDE)
   ![alt new project 2](https://aleksdraka-blog-posts.s3.eu-west-2.amazonaws.com/how-to-create-controller-based-apis/pic2.png "New project 2")

5. Your project structure should look like that:
   ![alt new project 3](https://aleksdraka-blog-posts.s3.eu-west-2.amazonaws.com/how-to-create-controller-based-apis/pic3.png "New project 3")

6. Add the following package by pasting the command below in the terminal. In this tutorial we will keep it simple and store data in memory rather than setting up a database which will be covered in future post

```
dotnet add package Microsoft.EntityFrameworkCore.InMemory --version 9.0.0
```

7. Run the following command which will add Scalar as our API client instead of Swagger

```
dotnet add package Scalar.AspNetCore
```

8. Finally, add the following code in the `Program.cs` file,
   within the configuration section of the app
   pipeline.

```
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.MapScalarApiReference(options =>
    {
        options
            .WithTitle("FlashCardAPI")
            .WithDefaultHttpClient(ScalarTarget.CSharp, ScalarClient.HttpClient);
    });
}
```

With that our project is set and we can begin writing the API!

### Model class

We need to add a model class which is
a representation of the data that is managed by our app.
Create a new folder named **Models** which is used by convention and add
a new class named **FlashCardItem**

```
namespace FlashCardAPI.Models;

public class FlashCardItem
{
    public long Id { get; set; }
    public string? Question { get; set; }
    public bool Answer { get; set; }
}
```

### Database context

The database context is like a middleman between our application
and the database. It's a special class we create by inheriting from
DbContext in Entity Framework. It knows about our data models (tables)
and handles database operations like reading, writing, updating,
and deleting data.

Add a new class in the Models folder name **CardContext**

```
using Microsoft.EntityFrameworkCore;

namespace FlashCardAPI.Models;

public class FlashCardContext : DbContext
{
    public FlashCardContext(DbContextOptions<FlashCardContext> options) : base(options)
    {
    }
    public DbSet<FlashCardItem> FlashCardItems { get; set; } = null!;
}
```

Next we have to register our DbContext service with the Dependency Injection (DI)
container so that the service can be used by the controller. Your `Program.cs`
file should look like this:

```
using FlashCardAPI.Models;
using Microsoft.EntityFrameworkCore;
using Scalar.AspNetCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddDbContext<FlashCardContext>(opt =>
    opt.UseInMemoryDatabase("FlashCardList"));
builder.Services.AddOpenApi();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.MapScalarApiReference(options =>
    {
        options
            .WithTitle("FlashCardAPI")
            .WithDefaultHttpClient(ScalarTarget.CSharp, ScalarClient.HttpClient);
    });
}

app.UseHttpsRedirection();

app.UseAuthorization();

app.MapControllers();

app.Run();
```

### Creating controller

Now we need to scaffold a controller, meaning generate a new one with our DbContext and FlashCard model. To do so:

1. Right click **Controllers** folder
2. Select **Scaffolded Item**
3. Select the following
   ![alt pic 4](https://aleksdraka-blog-posts.s3.eu-west-2.amazonaws.com/how-to-create-controller-based-apis/pic4.png "pic 4")
4. Choose FlashCard Item as model and FlashCardContext as DbContext

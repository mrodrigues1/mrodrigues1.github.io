---
layout: post
title: 'How To: Use EntityFramework Core In-Memory Database For Unit Testing'
date: 2018-03-19 10:00:37.000000000 -03:00
type: post
categories: unittest
tags: .net aaa database ef inmemorydatabase sut
permalink: "/2018/03/19/how-to-use-entityframework-core-in-memory-database-for-unit-testing/"
image: 
  path: /assets/img/blog/How-To-Use-EntityFramework-Core-In-Memory-Database-For-Unit-Testing.png
description: >
  In this post you will learn how to use entityframework in memory database.
---
You can download this article project on [GitHub](https://github.com/mrodrigues1/SimpleToDo).

In this post, I'll show how to use the in-memory feature of Entity Framework core to create unit tests involving the database. Let's jump right in!

Create unit tests using a real _DbContext_ is a pain because we have to mock the entire context to achieve this.&nbsp;Entity Framework core makes unit tests easier to write by providing us with an in-memory database provider. It creates an in-memory representation of our _DbContext_, for this reason, we don't have to worry about mocking the database.

## **Setup your context**

First, the database configuration must be done in the `Startup.cs` class, as it usually is in ASP.NET core projects. To illustrate:

~~~csharp
public class Startup
{
    //...........
    //Other Stuff
    //...........

    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddDbContext<ToDoDbContext>(
            options =>
                options
                .UseSqlServer(Configuration.GetConnectionString("SimpleToDo")));

        //...........
        //Other Configurations
        //...........
    }

    //...........
    //Other Stuff
    //...........
}
~~~

Next, the context should accept multiple database providers. We need a constructor which receives a&nbsp;`DbContextOptions` object:

~~~csharp
public ToDoDbContext(DbContextOptions options) : base(options) { }
~~~

Finally, we can create the context with different configurations.

## **Create First Unit Test**

We will add a new NuGet package to our unit test project. Open the package manager console and use the following command:

`PM> Install-Package Microsoft.EntityFrameworkCore.InMemory`

Now we can start to use the in-memory database feature.

Create a new unit test class for which functionality you want to test. In my case, it will be for the `ToDoListRepository.cs`.

~~~csharp
public class ToDoListRepositoryTests
{
    [Fact]
    public void CreateToDoList_WithValidObject_NewListIdIsNotEqualsToZero()
    {
        //Arrange
        var newToDoList = new List
        {
            Name = "Unit Test"
        };

        var sut = CreateSUT();
    
        //Act
        sut.CreateToDoList(newToDoList);

        //Assert
        Assert.True(newToDoList.ListId != 0);
    }

    private ToDoListRepository CreateSUT()
    {
        var dbOptions = new DbContextOptionsBuilder<ToDoDbContext>()
                            .UseInMemoryDatabase(databaseName: "ToDoDb")
                            .Options;

        var context = new ToDoDbContext(dbOptions);

        return new ToDoListRepository(context);
    }        
}
~~~

In this example, the unit test is validating if the database saved the to-do list.&nbsp;The magic happens in the `CreateSUT()` method. It isolates the system under test (sut) creation, helping avoid code duplication, as a result.

Inside the method, we&nbsp;use&nbsp;_DbContextOptionsBuilder_&nbsp;to create&nbsp;`dbOptions`&nbsp;object.&nbsp;_DbContextOptionsBuilder_&nbsp; gives us the option to use an in-memory database, with the method&nbsp;`UseInMemoryDatabase()`. Now we send&nbsp;`dbOptions`&nbsp;as a parameter to create the context.

After the arrange phase in the unit test, we save the list in the database with&nbsp;`sut.CreateToDoList(newToDoList);`.

Finally,&nbsp;we validate if the _ListId_ is different than zero, in the assertion phase. If the assertion is true, the database saved the register.

## **Wrap up**

This concludes the post, you can use this sample code as an example to create your own unit test using the in-memory database feature from Entity Framework core.

## **References and Further Reading**

- [Using Entity Framework Core in-memory database for unit testing -&nbsp;Gunnar Peipman](http://gunnarpeipman.com/2017/04/aspnet-core-ef-inmemory/)
- [Testing with InMemory - Microsoft Docs](https://docs.microsoft.com/en-us/ef/core/miscellaneous/testing/in-memory)
- [SUT Factory - Matheus Rodrigues](https://www.matheus.ro/2017/09/25/sut-factory/)

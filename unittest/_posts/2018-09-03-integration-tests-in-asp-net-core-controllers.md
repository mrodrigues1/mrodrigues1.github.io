---
layout: post
title: 'Integration Tests In ASP.NET Core Controllers'
date: 2018-09-03 08:30:48.000000000 -03:00
type: post
categories: unittest
tags: aaa aspnetcoremvc autofixture fluentassertions xunit
image: 
  path: /assets/img/blog/Integration-Tests-In-ASP.NET-Core-Controllers.png
permalink: "/2018/09/03/integration-tests-in-asp-net-core-controllers/"
description: >
  Continuing the series of posts to explore different testing aspects in asp .net core. We’re going experiment with Test Server to create integration tests.
---
Continuing the series of posts to explore different testing aspects in asp .net core. We’re going experiment with Test Server to create integration tests.

Test Server is a new features that asp .net core is bringing. It creates a browser abstraction, allowing us to simulate application’s behavior without opening a browser.

There are many ways to deal with the database in integration tests. The idea is to not affect the data that’s already in the database. We could use an entirely different database, creating and deleting it every time we run our tests.

Other approach is to use the same database as the application, but clean up data after each test. For example, if we add a register into the database and in the end of the test we remove it.

We can follow a transactional approach. Which means that we create a transaction scope within each unit test to not commit changes when we call `.SaveChanges()`. Then, when the test is over, this transaction is disposed and no changes are committed to the database.

Entity Framework Core In-Memory database could also be used. Following this approach we won’t have to worry about storing data. But, how can we call this an integration test if we aren’t using a real database? Yeah, that's right, this approach will fall into the unit test category.

For this post, we’re going to use the transactional approach.

## **Project**

I’m going to use the same project from my last post. It’s a simple todo list app that I use to experiment with the new features from ASP.NET core.

In this post, I’m not going to show every implementation detail, feel free to download/clone the repository.

[You can download or clone the git repository here.](https://github.com/mrodrigues1/SimpleToDo)

## **SUT**

We’re going to create integration tests for the `ToDoListController`&nbsp;class:

~~~csharp
public class ToDoListController : Controller
{
    private readonly IToDoListService _toDoListService;

    public ToDoListController(IToDoListService toDoListService)
    {
        _toDoListService = toDoListService;
    }

    // GET: ToDoList
    public async Task<IActionResult> Index()
    {
        return View(await _toDoListService.ToDoLists());
    }

    // GET: ToDoList/Details/5
    public async Task<IActionResult> Details(int? id)
    {
        if (id == null) return NotFound();

        var list = await _toDoListService.FindById(id.Value);

        if (list == null) return NotFound();

        return View(list);
    }

    // GET: ToDoList/Create
    public IActionResult Create()
    {
        return View();
    }

    // POST: ToDoList/Create        
    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Create([Bind("Id,Name")] ToDoList toDoToDoList)
    {
        if (ModelState.IsValid)
        {
            await _toDoListService.Create(toDoToDoList);

            this.AddAlertSuccess($"{toDoToDoList.Name} created.");

            return RedirectToAction(nameof(Index));
        }

        return View(toDoToDoList);
    }

    // GET: ToDoList/Edit/5
    public async Task<IActionResult> Edit(int? id)
    {
        if (id == null) return NotFound();

        var list = await _toDoListService.FindById(id.Value);

        if (list == null) return NotFound();

        return View("Edit", list);
    }

    // POST: ToDoList/Edit/5        
    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Edit(int id, [Bind("Id,Name")] ToDoList toDoList)
    {
        if (id != toDoList.Id)
            return NotFound();

        if (!ModelState.IsValid) return View(toDoList);

        try
        {
            await _toDoListService.Update(toDoList);
        }
        catch (DbUpdateConcurrencyException)
        {
            var todoExists = await _toDoListService.Exists(id);

            if (!todoExists)
                return NotFound();

            throw;
        }

        this.AddAlertSuccess($"{toDoList.Name} updated.");
        return RedirectToAction(nameof(Index));
    }

    // GET: ToDoList/Delete/5
    public async Task<IActionResult> Delete(int? id)
    {
        if (id == null) return NotFound();

        var list = await _toDoListService.FindById(id.Value);

        if (list == null) return NotFound();

        return View(list);
    }

    // POST: ToDoList/Delete/5
    [HttpPost]
    [ActionName("Delete")]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> DeleteConfirmed(int id)
    {
        var listName = await _toDoListService.Remove(id);

        this.AddAlertSuccess($"{listName} removed.");

        return RedirectToAction(nameof(Index));
    }
}
~~~

Most of `ToDoListController`&nbsp;class is boilerplate code, generated when creating a class via scaffolding. I made some tweaks, for instance, took out to `IToDoListService`&nbsp;all code all database code related and there is some code related to an alert system. [You can check about this alert system here](https://www.matheus.ro/2017/12/18/how-to-create-a-simple-alert-system-using-tag-helpers-in-asp-net-core-mvc/).

## **Setup Fixture**

The database and transactions configuration requires a more steps.

Let’s start with a class that ensures database exists and can run the code migrations. We call this class Fixture:

~~~csharp
public class Fixture
{
    static Fixture()
    {
        Configuration = GetConfiguration();
        CreateDatabase();
    }

    private static IConfiguration GetConfiguration()
        => new ConfigurationBuilder().AddJsonFile("appsettings.json").Build();

    private static void CreateDatabase()
    {
        var options = new DbContextOptionsBuilder<ToDoDbContext>()
            .UseSqlServer(Configuration["DbConnection"])
            .Options;

        new ToDoDbContext(options).Database.Migrate();
    }

    protected static IConfiguration Configuration { get; }
}
~~~

This class has a static constructor, running each time Fixture is instantiated. In the constructor we get the configuration which has the database connection string. `GetConfiguration()`&nbsp;is responsible to get all configuration from `appsettings.json`&nbsp;file. This file is pretty similar to the file we have on the web project, but we only have the connection string there:

~~~csharp
{
 "DbConnection":
   "Server=(localdb)\\MSSQLLocalDB;Database=SimpleToDo;Trusted_Connection=True;MultipleActiveResultSets=true"
}
~~~

The connection string is the same as in the web application, but feel free to change to whatever connection string you want.

In the `CreateDatabase()`&nbsp;method, we create a `DbContextOptionsBuilder`&nbsp;object with `Configuration["DbConnection"]`&nbsp;to get the connection string from the file. Then, we create a new `ToDoDbContext`&nbsp;instance, sending the options into the constructor. Finally, we call `.Migrate()`&nbsp;to run all code migrations. This method also creates the database if needed.

Done with the Fixture class, we can move on to the next step.

## **Setup WebFixture**

Now, we need a class which will contain the transaction scope configuration and instantiate Test Server. Also, it will inherit Fixture class, to guarantee that the database exists. Without further ado, let’s take a look at the `WebFixture`&nbsp;implementation:

~~~csharp
public class WebFixture<TStartup> : Fixture, IDisposable where TStartup : class
{
    private readonly IServiceProvider _services;

    public HttpClient Client;

    public ToDoDbContext DbContext { get; }

    public IDbContextTransaction Transaction;

    public WebFixture()
    {
        IWebHostBuilder builder = WebHost.CreateDefaultBuilder()
            .UseStartup<TStartup>();
        var server = new TestServer(builder);
        Client = server.CreateClient();
        _services = server.Host.Services;

        DbContext = GetService<ToDoDbContext>();
        Transaction = DbContext.Database.BeginTransaction();
    }

    protected T GetService<T>() => (T) _services.GetService(typeof(T));

    public void Dispose()
    {
        if (Transaction == null) return;

        Transaction.Rollback();
        Transaction.Dispose();
    }
}
~~~

`WebFixture<TStartup>` is a generic class which accepts an `Startup.cs`&nbsp;object. If you take look on the constructor, `TestServer`&nbsp;object receives an `IWebHostBuilder`&nbsp;object in its constructors parameter. The Startup contains all container and HTTP request pipeline configurations.

We use `WebHost.CreateDefaultBuilder()`method to create `IWebHostBuilder`. Then, `TestServer` constructor receives our builder as a parameter.

With the `TestServer` instance, we create the client and assign it to a public variable. `Client` is responsible to make calls to controller actions. Next, we get the registered services from `server.Host`&nbsp;and assign it to a private variable `_services`.

Using `GetService()`&nbsp;method to get `DbContext`&nbsp;object and assign it to a public variable, so it will be accessible in classes that inherits it. To finish the constructor, we open a transaction with `DbContext.Database.BeginTransaction()`&nbsp;and store it in a public variable. It opens a transaction in the database then, the database will not commit the changes right away, it only when we call `Transaction.Commit()`.

This class implements `IDisposable`&nbsp;interface. The idea is to rollback all database changes and dispose the transaction after each test runs.

About the startup stub class that we need, let’s take a look on it in the following section.

## **Startup Stub**

This class is similar to Startup class from the web project, but there’s one catch that will make the difference when making integration tests with transactions.

~~~csharp
public class StartupStub
{
    static StartupStub()
    {
        Configuration = GetConfiguration();
    }

    protected static IConfiguration Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<CookiePolicyOptions>(options =>
        {
            // This lambda determines whether user consent for non-essential cookies is needed for a given request.
            options.CheckConsentNeeded = context => true;
            options.MinimumSameSitePolicy = SameSiteMode.None;
        });

        services.AddDbContext<ToDoDbContext>(
            options => options.UseSqlServer(Configuration["DbConnection"]),
            ServiceLifetime.Singleton,
            ServiceLifetime.Singleton);

        services.AddTransient<IToDoListService, ToDoListService>();
        services.AddTransient<ITaskService, TaskService>();

        services.AddTransient<IToDoListRepository, ToDoListRepository>();
        services.AddTransient<ITaskRepository, TaskRepository>();

        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
    }

    public void Configure(IApplicationBuilder app, ToDoDbContext dbContext)
    {
        dbContext.Database.Migrate();            

        app.UseHttpsRedirection();
        app.UseStaticFiles();
        app.UseCookiePolicy();

        app.UseMvc(routes =>
        {
            routes.MapRoute(
                name: "default",
                template: "{controller=ToDoList}/{action=Index}/{id?}");
        });
    }

    private static IConfiguration GetConfiguration()
        => new ConfigurationBuilder().AddJsonFile("appsettings.json").Build();
}
~~~

The differences is when calling `WebHost.CreateDefaultBuilder().UseStartup<TStartup>();`&nbsp;on the `WebFixture<TStartup>`&nbsp;class constructor, we need a class containing both `ConfigureServices(...)`&nbsp;and `Configure(...)`&nbsp;methods. I know that isn’t good to have duplication, but let me explain why. The difference is on how we register the `DbContext`:

~~~csharp
services.AddDbContext<ToDoDbContext>(
            options => options.UseSqlServer(Configuration["DbConnection"]),
            ServiceLifetime.Singleton,
            ServiceLifetime.Singleton);
~~~

`AddDbContext<T>` has an overload with three parameters, one for the option and two others responsible to control the `DbContext`&nbsp;and `DbContextOptions`&nbsp;life cycles. I’m setting both object to be Singletons, you may think that [singletons are evil](http://wiki.c2.com/?SingletonsAreEvil) but there is a reason.

The default life cycle for both objects is Scoped, meaning that for each request the application creates a new instance of them. Scoped life cycle would not work, because `WebFixture<TStartup>`&nbsp;class would get an instance of the `DbContext` and the test would get another instance. So, the changes we would make on the `DbContext` from the `WebFixture<TStartup>`&nbsp;class are not going reflect when calling the application on the test. That is why we need `DbContext` to be a singleton, it’s gonna be only one `DbContext` instance throughout the whole application. Then, when we make a change, the client’s calls will be up to date since it is the same `DbContext` instance.

## **Unit Test Class**

I’m using three test libraries on this project. The first one is [xUnit](https://xunit.github.io/), it’s the unit test framework. [FluentAssertions](https://fluentassertions.com/), which helps us to create better assertions for our test. And [AutoFixture](https://github.com/AutoFixture/AutoFixture)framework, which allow us to create objects with dummy data and focus only on what’s relevant to test. AutoFixture is a powerful framework, I encourage you to check it want to learn more about it.

We have to create a class to implement the tests, since we’re going to test `ToDoListController`&nbsp;class, I’m going to call it `ToDoListControllerTests`:

~~~csharp
public class ToDoListControllerTests : IClassFixture<WebFixture<StartupStub>>
{
    private readonly WebFixture<StartupStub> _fixture;

    public ToDoListControllerTests(WebFixture<StartupStub> fixture)
    {
        _fixture = fixture;
    }
    
    //Tests...
}
~~~

The class inherits from `IClassFixture<WebFixture<StartupStub>>`&nbsp;using the `StartupStub`&nbsp;class created before. `IClassFixture<T>`&nbsp;acts as `[ClassInitialize]`&nbsp;from MsTest framework or `[OneTimeSetup]`&nbsp;from NUnit. Meaning that whatever object we place, its constructor will only run one time before all tests from the class run. `ToDoListControllerTests`&nbsp;constructor receives `WebFixture<TStartup>`&nbsp;that `IClassFixture`&nbsp;injects and we call it `_fixture`. This `_fixture` will gives access to all public properties from `WebFixture`&nbsp;has, `Client`, `DbContext`&nbsp;and `Transaction`. These are the properties we need to create our integration tests.

Let’s move to the first test.

## **First Integration Test**

For the first test, let’s start with a simple one. It’s a test to check if the client is able to call the application correctly returning a http success status code:

~~~csharp
[Fact]
public async Task Index_GetAsyncCall_ResponseReturnsSuccessStatusCode()
{
    //Act
    var response = await _fixture.Client.GetAsync("/");

    //Assert
    response.EnsureSuccessStatusCode();
}
~~~

We don’t need any setup here, just have to call the application root. To check if it was successful, we call `EnsureSuccessStatusCode()`&nbsp;on the response.

As can be seen, we send a http get request to the application root, `“/”`. This is the equivalent of send a request to `“/ToDoList/Index”`. Because the application default route is configured in this way on the `StartupStub`&nbsp;class.

This is a fairly simple test, let’s move to a more complex test.

## **Index Action Response Contains Recently Added To Do List**

In this test we’re going to check if a recently added to do list is present on the view.

~~~csharp
[Fact]
public async Task Index_GetAsyncCall_ReturnNewToDoListToView()
{
    //Arrange
    var toDoList = ToDoListFactory.Create().Single();
    await _fixture.DbContext.ToDoList.AddAsync(toDoList);
    await _fixture.DbContext.SaveChangesAsync();

    //Act
    var response = await _fixture.Client.GetAsync("/");

    //Assert 
    response
        .Content
        .ReadAsStringAsync()
        .Result
        .Should()
        .Contain(toDoList.Name);
}
~~~

In the arrange, we create a to do list and save it into the database. But, how we can see if the to do list is on the screen?

For that, we call the Index action (“/”) from the `ToDoListController`, action responsible to show all to do lists from database. We aren’t able to cast the response object to `List<ToDoList>`&nbsp;object because the response only contains HTML that the application will show on the screen.

One way to check if the response is right, is to get the response content with `.ReadAsStringAsync()`&nbsp;to access its HTML. Finally, we check the HTML, looking for the to do list name that were created in the beginning of the test.

One more thing to talk about this test, in the arrange section, you can see that I’m using a factory to create a new `ToDoList`&nbsp;object. It helps object creation and let the test cleaner:

~~~csharp
public static class ToDoListFactory
{
    private static readonly AutoFixture.Fixture Fixture = new AutoFixture.Fixture();

    public static IEnumerable<ToDoList> Create(int count = 1)
        => Fixture
            .Build<ToDoList>()
            .With(x => x.Id, 0)
            .With(x => x.Tasks, new List<Task>())
            .CreateMany<ToDoList>(count);
}
~~~

In the `ToDoListFactory`&nbsp;class, we’re going to use AutoFixture framework to help us create dummy objects. `Create(...)`&nbsp;method has a parameter to indicate how many `ToDoList`&nbsp;objects it will create. I’m not going into details regarding AutoFixture, it’s not my objective here, but you can check more about it [here](https://github.com/AutoFixture/AutoFixture).

## **Adding a new To Do List with a Post Request**

To add a new to do list in our database, we have to call the http post version of the ToDoList/Create action. Since, we need to send complex data in the request, not only return data in the response:

~~~csharp
[Fact]
public async Task Create_PostAsyncCallWithValidToDoList_RedirectToIndexAction()
{
    //Arrange            
    var formData = new Dictionary<string, string>
    {
        { nameof(ToDoList.Name), "To Do List 1" }
    };

    //Act
    var response = await _fixture.Client.PostAsync(
        "/ToDoList/Create",
        new FormUrlEncodedContent(formData));

    //Assert                        
    response.Headers.Location.ToString().Should().Be("/");
}
~~~

To start, we create `formData`, which consists of data we want to send to server. It’s a dictionary that will contain the model property name as a key and a value. The dictionary key has to be the same as the property in the object that the action is receiving.

Then, to post data into the server we have to call `.PostAsync()`&nbsp;method in our client. This method has two parameters, one for the request uri and another one for the http content. For the second parameter, we’re creating a `FormUrlEncodedContent`&nbsp;object with the `formData` object.

Now, moving to the test’s assertion. I’m just checking if the application redirects to its root, meaning the to do list creation was successful.

We run the test and we’re getting an error:

[caption id="attachment\_1480" align="aligncenter" width="775"][![]({{ site.baseurl }}/assets/2018/09/Screenshot_1.png)](https://www.matheus.ro/wp-content/uploads/2018/09/Screenshot_1.png) Bad Request[/caption]

The error says that our request was a bad request. But why? It’s because we are missing one important aspect when working with asp.net core mvc and posting data to the server, the Anti Forgery Token.

### **Anti Forgery Token**

Anti Forgery Token is responsible for prevent cross-site request forgery (CSRF) attack. It’s a method that generates a code and put it on the view to avoid send malicious or fake data to the server.

The method inserts an hidden HTML on the view:

~~~html
<input name="__RequestVerificationToken" value="CfDJ8OQwpi5a48lKkQO8m3zba5EQqYER3bFD0yB37yQAExLvqhUVJ1c68FMknSS04h8u62BEYtsMwQQGudqp6DOs1--1K_IoPq0g0dRmeSiuPkHhZ5I1HACf-djqPNytvUc8TmsD7sGFNwXtvGTroe-GIf4" type="hidden">
~~~

In ASP.NET Core MVC the tag `<form>`&nbsp;creates the Anti Forgery Token automatically when on a view.

So, the problem we’re having with this test is that the missing token because of the `[ValidateAntiForgeryToken]`&nbsp;attribute on the `[HttpPost]`&nbsp;create action. This attribute validates if the data on the request has the correct token. The token has to be in the request, but how can we do this?

To get the token, we have to make a request to a page that has a `<form>`, get the code that `<form>`&nbsp;generates and add it to our request:

~~~csharp
public static class AntiForgeryHelper
{
    private static string _antiForgeryToken;
    private static SetCookieHeaderValue _antiForgeryCookie;

    public static Regex AntiForgeryFormFieldRegex = new Regex(
        @"\<input name=""__RequestVerificationToken"" type=""hidden"" value=""([^""]+)"" \/\>");

    public static async Task<string> EnsureAntiForgeryTokenAsync(HttpClient client)
    {
        if (_antiForgeryToken != null)
            return _antiForgeryToken;

        var response = await client.GetAsync("/ToDoList/Create");
        response.EnsureSuccessStatusCode();

        _antiForgeryCookie = TryGetAntiForgeryCookie(response);

        Assert.NotNull(_antiForgeryCookie);

        AddCookieToDefaultRequestHeader(client, _antiForgeryCookie);

        _antiForgeryToken = await GetAntiForgeryToken(response);

        Assert.NotNull(_antiForgeryToken);

        return _antiForgeryToken;
    }

    private static SetCookieHeaderValue TryGetAntiForgeryCookie(HttpResponseMessage response)
    {
        if (response.Headers.TryGetValues("Set-Cookie", out IEnumerable<string> values))
        {
            return SetCookieHeaderValue.ParseList(values.ToList())
                .SingleOrDefault(
                    c => c.Name.StartsWith(
                        ".AspNetCore.AntiForgery.",
                        StringComparison.InvariantCultureIgnoreCase));
        }

        return null;
    }

    private static void AddCookieToDefaultRequestHeader(
        HttpClient client,
        SetCookieHeaderValue antiForgeryCookie)
    {
        client.DefaultRequestHeaders.Add(
            "Cookie",
            new CookieHeaderValue(antiForgeryCookie.Name, antiForgeryCookie.Value)
                .ToString());
    }

    private static async Task<string> GetAntiForgeryToken(HttpResponseMessage response)
    {
        var responseHtml = await response.Content.ReadAsStringAsync();
        var match = AntiForgeryFormFieldRegex.Match(responseHtml);

        return match.Success ? match.Groups[1].Captures[0].Value : null;
    }
}
~~~

The main focus of the `AntiForgeryHelper.cs`&nbsp;class is the `EnsureAntiForgeryTokenAsync()`&nbsp;method.

It’s a static async method with that returns the anti forgery token value. Receives our client as a parameter to make a request to the `"/ToDoList/Create"`, which is a view that has the token. After that, we try to get the anti forgery cookie from the response using `TryGetAntiForgeryCookie()`. If the cookie is nowhere to be found, we’re trying to get the cookie from a view that doesn’t have the \<form\> tag.

With the cookie in hands, we add it the client default request headers. &nbsp;

Finally, we can get the anti forgery token from the html in the response. To do this, we have to read the response content and scrap the token using a regex.

In the method’s beginning, there’s a check to see if the token already exists. I’m doing this because we don’t need to get a new token for each request, if the token is already exists, there’s no need to get new one.

Now, we need to add the token to our test and see if it’s working.

### **Add Anti Forgery Token To The Request**

We need to to add the token into the `formData`:

~~~csharp
[Fact]
public async Task Create_PostAsyncCallWithValidToDoList_RedirectToIndexAction()
{
    //Arrange            
    var formData = new Dictionary<string, string>
    {
        {
            "__RequestVerificationToken",
            await AntiForgeryHelper.EnsureAntiForgeryTokenAsync(_fixture.Client)
        },
        { nameof(ToDoList.Name), "To Do List 1" }
    };

    //Act
    var response = await _fixture.Client.PostAsync(
        "/ToDoList/Create",
        new FormUrlEncodedContent(formData));

    //Assert            
    response.Headers.Location.ToString().Should().Be("/");
}
~~~

By adding the token in the request and getting the token using `AntiForgeryHelper.EnsureAntiForgeryTokenAsync()`&nbsp;method, the test now works.

## **Edit Action Is Loading The Correct To Do List**

In this test we want to load a to do list in the edit view and check if the view is loading it correctly.

~~~csharp
[Fact]
public async Task Edit_GetAsyncCallWithNewToDoListId_ReturnToDoListToView()
{
    //Arrange
    var toDoList = ToDoListFactory.Create().Single();
    await _fixture.DbContext.ToDoList.AddAsync(toDoList);
    await _fixture.DbContext.SaveChangesAsync();

    //Act
    var response = await _fixture.Client.GetAsync($"/ToDoList/Edit/{toDoList.Id}");

    //Assert            
    response
        .Content
        .ReadAsStringAsync()
        .Result
        .Should()
        .Contain(toDoList.Id.ToString());
}
~~~

We start by adding a new to do list into the database using `DbContext`. Then, we call `GetAsync()`, sending the url as parameter with the `toDoList.Id`&nbsp;in the query string.

In the assertion, we read the response content and check if it contains the `toDoList.Id`, meaning that application loaded the to do list in the view.

## **Calling Edit Action With a Null ToDoList Id**

In this test, we want to check if the application returns a http not found status code when we make a call to `"/ToDoList/Edit/"` without the to do list id on the query string:

~~~csharp
[Fact]
public async Task Edit_GetAsyncCallWithNullId_ReturnNotFoundStatusCode()
{
    //Arrange

    //Act
    var response = await _fixture.Client.GetAsync("/ToDoList/Edit/");

    //Assert            
    response
        .StatusCode
        .Should()
        .Be(HttpStatusCode.NotFound);
}
~~~

## **Editing Previously Created ToDoList**

Let’s say we want to edit a to do list:

~~~csharp
[Fact]
public async Task Edit_PostAsyncCallWithValidIdAndToDoList_RedirectToIndexView()
{
    //Arrange
    var toDoList = ToDoListFactory.Create().Single();
    await _fixture.DbContext.ToDoList.AddAsync(toDoList);
    await _fixture.DbContext.SaveChangesAsync();

    var formData = new Dictionary<string, string>
    {
        {
            "__RequestVerificationToken",
            await AntiForgeryHelper.EnsureAntiforgeryTokenAsync(_fixture.Client)
        },
        { "id", toDoList.Id.ToString() },
        { "Id", toDoList.Id.ToString() },
        { "Name", "ToDoList Test 1" }
    };

    //Act
    var response = await _fixture.Client
        .PostAsync(
            "/ToDoList/Edit/",
            new FormUrlEncodedContent(formData));

    //Assert                        
    response.Headers.Location.ToString().Should().Be("/");
}
~~~

First, we create a to do list and save it to the database. To finish the arrange, we create the `formData`&nbsp;with the validation token and the data we want to edit.

I’m adding both “id” and “Id” to the form data. `"/ToDoList/Edit/"` action needs the “id” on the query string and “Id” is for to do list object itself when receiving a call.

Then, we make a `PostAsync()`call to the edit action.

Finally, we validate if the application redirects the user to the root(“ToDoList/Index”), meaning that the to do list was updated.

## **Posting An Invalid Model To Edit Action**

How about about testing a negative flow?

We could post a model with an invalid state to check if the view shows the right errors messages. To get a model into an invalid state, we need to post the model without data in properties that have `[Required]`&nbsp;data attribute.

To simulate this behavior, we can send data to the edit action(“ToDoList/Edit”) without `“Name”` key on the `formData`&nbsp;object:

~~~csharp
[Fact]
public async Task Edit_PostAsyncCallWithModelStateInvalid_ShowErrorMessageOnEditView()
{
    //Arrange
    var toDoList = ToDoListFactory.Create().Single();
    await _fixture.DbContext.ToDoList.AddAsync(toDoList);
    await _fixture.DbContext.SaveChangesAsync();

    var formData = new Dictionary<string, string>
    {
        {
            "__RequestVerificationToken",
            await AntiForgeryHelper.EnsureAntiforgeryTokenAsync(_fixture.Client)
        },
        { "id", toDoList.Id.ToString() },
        { "Id", toDoList.Id.ToString() }
    };

    //Act
    var response = await _fixture.Client
        .PostAsync(
            "/ToDoList/Edit/",
            new FormUrlEncodedContent(formData));

    //Assert                        
    response
        .Content
        .ReadAsStringAsync()
        .Result
        .Should()
        .Contain("The Name field is required.");
}
~~~

The action should return the response with the validation message in the its HTML. The validation message, `"The Name field is required."`, is the default message when data attribute `[Required]`&nbsp;is present on a property’s class.

## **Removing A ToDoList**

For the last test, let’s just remove a to do list from the database using `DeleteConfirmed`&nbsp;action.

~~~csharp
[Fact]
public async Task DeleteConfirmed_PostAsyncCallWithValidIdAndToDoList_RedirectToIndexView()
{
    //Arrange
    var toDoList = ToDoListFactory.Create().Single();
    await _fixture.DbContext.ToDoList.AddAsync(toDoList);
    await _fixture.DbContext.SaveChangesAsync();

    var formData = new Dictionary<string, string>
    {
        {
            "__RequestVerificationToken",
            await AntiForgeryHelper.EnsureAntiforgeryTokenAsync(_fixture.Client)
        },
        { "id", toDoList.Id.ToString() }
    };

    //Act
    var response = await _fixture.Client
        .PostAsync(
            $"/ToDoList/Delete/",
            new FormUrlEncodedContent(formData));

    //Assert            
    response.Headers.Location.ToString().Should().Be("/");
}
~~~

The test’s structure is similar to the other ones. We’re sending only the to do list id in the request, because this is the only information necessary to remove the to do list from database.

In the assertion we check if the application redirect the user to index action, meaning the operation was successful.

## **References and Further Reading**

- [Leveling Up Your .Net Testing Patterns - Part II Transactional Integration Testing - nance.io](https://nance.io/leveling-up-your-dotnet-testing-transactional-integration-testing-in-asp-net-core/)
- [Integration Testing for ASP.NET Core Applications - dotnetcurry](https://www.dotnetcurry.com/aspnet-core/1420/integration-testing-aspnet-core)
- [Integration testing your Asp .Net Core App – Dealing with Anti Request Forgery (CSRF), Form Data and Cookies - Stefan Hendriks' blog](http://www.stefanhendriks.com/2016/05/11/integration-testing-your-asp-net-core-app-dealing-with-anti-request-forgery-csrf-formdata-and-cookies/)
- [Easier functional and integration testing of ASP.NET Core applications - Scott Hanselman](https://www.hanselman.com/blog/EasierFunctionalAndIntegrationTestingOfASPNETCoreApplications.aspx)
- [Integration tests in ASP.NET Core - Microsoft Docs](https://docs.microsoft.com/en-us/aspnet/core/test/integration-tests)

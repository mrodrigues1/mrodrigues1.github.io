---
layout: post
title: Unit Tests In ASP.NET Core Controllers
date: 2018-05-28 09:00:15.000000000 -03:00
type: post
categories: unittest
tags: aaa aspnetcoremvc fluentassertions sut
image: 
  path: /assets/img/blog/Unit-Tests-In-ASP.NET-Core-Controllers-1.png
permalink: "/2018/05/28/unit-tests-in-asp-net-core-controllers/"
description: >
  Continuing the series of posts to explore different testing aspects in asp .net core. We’re going to create unit test for controllers.
---
In this new series of posts, I want to show different aspects of testing an ASP.NET core application.

I will start this post showing how to create unit tests for controllers. Then I am going to talk integration test in the controller and explore ASP.NET core new feature, Test Host. After that, let’s see how we can test our application business logic.

Moving to the database realm. I’m going to show how we can create unit tests for code that deals with `DbContext`, using Entity Framework Core In-memory database feature. And them, how to do integration tests with a real database, with strategies to maintain a database for integration tests.

Let’s jump right into it!

## **Project**

The samples I’m going to show throughout this series are from a small project I create to experimenting with the new ASP.NET core features.

It is a to do list, where you can create many lists and create tasks in those lists.

### **Technologies / Frameworks In The Project**

- Visual Studio 2017 Community
- ASP.NET Core 2.0
- Entity Framework Core 2.0
- LocalDB
- xUnit
- FluentAssertions
- FakeItEasy

In this post, I’m not going to show every implementation detail, feel free to download/clone the repository.

[You can download or clone the git repository here.](https://github.com/mrodrigues1/SimpleToDo)

## **ToDoList Controller Class**

Take a look at the following controller class:

~~~csharp
public class ToDoListController : Controller
{
    private readonly IToDoListService _toDoListService;

    public ToDoListController(IToDoListService toDoListService)
    {
        _toDoListService = toDoListService;
    }

    // GET: List
    public async Task<IActionResult> Index()
    {
        return View(await _toDoListService.ToDoLists());
    }

    // GET: List/Details/5
    public async Task<IActionResult> Details(int? id)
    {
        if (id == null) return NotFound();

        var list = await _toDoListService.FindById(id.Value);

        if (list == null) return NotFound();

        return View(list);
    }

    // GET: List/Create
    public IActionResult Create()
    {
        return View();
    }

    // POST: List/Create
    // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
    // more details see http://go.microsoft.com/fwlink/?LinkId=317598.
    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Create([Bind("Id,Name")] ToDoList toDoList)
    {
        if (ModelState.IsValid)
        {
            await _toDoListService.Create(toDoList);

            this.AddAlertSuccess($"{toDoList.Name} created successfully.");

            return RedirectToAction(nameof(Index));
        }

        return View(toDoList);
    }

    // GET: List/Edit/5
    public async Task<IActionResult> Edit(int? id)
    {
        if (id == null) return NotFound();

        var list = await _toDoListService.FindById(id.Value);

        if (list == null) return NotFound();

        return View("Edit", list);
    }

    // POST: List/Edit/5
    // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
    // more details see http://go.microsoft.com/fwlink/?LinkId=317598.
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
            bool todoExists = await _toDoListService.Exists(id);

            if (!todoExists)
                return NotFound();
            else
                throw;
        }

        this.AddAlertSuccess($"{toDoList.Name} updated successfully.");
        return RedirectToAction(nameof(Index));
    }

    // GET: List/Delete/5
    public async Task<IActionResult> Delete(int? id)
    {
        if (id == null) return NotFound();

        var list = await _toDoListService.FindById(id.Value);

        if (list == null) return NotFound();

        return View(list);
    }

    // POST: List/Delete/5
    [HttpPost, ActionName("Delete")]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> DeleteConfirmed(int id)
    {
        string listName = await _toDoListService.Remove(id);

        this.AddAlertSuccess($"{listName} removed successfully.");

        return RedirectToAction(nameof(Index));
    }
}
~~~

Here, we are going to create unit tests for `ToDoListController`. It has the Index action to show the to do list, and create, edit and delete actions as well.

Even though this is most boilerplate code from the ASP.NET core, it serves the purpose we need.

One difference is that I created a service class `IToDoListService`, to isolate &nbsp;business logic from the controller.

Have thin controllers is important because it has less responsibilities and become easier to maintain and create unit tests. When create a controller with ASP.NET core MVC scaffolding all actions are async operations, I choose to leave as it is.

I use the default dependency injection provider from ASP.NET core to register &nbsp;our dependencies and inject them into the controller.

See bellow, `IToDoListService`&nbsp;and its implementation `ToDoListService`:

~~~csharp
public interface IToDoListService
{
    Task<List<ToDoList>> ToDoLists();
    Task<ToDoList> FindById(int id);
    Task Create(ToDoList toDoList);
    Task Update(ToDoList toDoList);
    Task<string> Remove(int id);
    Task<bool> Exists(int id);
}


public class ToDoListService : IToDoListService
{
    private readonly IToDoListRepository _toDoListRepository;

    public ToDoListService(IToDoListRepository toDoListRepository)
    {
        _toDoListRepository = toDoListRepository;
    }

    public Task<List<ToDoList>> ToDoLists() =>
        _toDoListRepository.ToDoLists().ToListAsync();

    public Task<ToDoList> FindById(int id) =>
        _toDoListRepository.FindById(id);

    public Task Create(ToDoList toDoList) =>
        _toDoListRepository.Create(toDoList);

    public Task Update(ToDoList toDoList) =>
        _toDoListRepository.Update(toDoList);

    public async Task<string> Remove(int id)
    {
        var list = await this.FindById(id);

        await _toDoListRepository.Remove(list);

        return list.Name;
    }

    public Task<bool> Exists(int id) =>
        _toDoListRepository.Exists(id);
}
~~~

`ToDoListService` class implements&nbsp;`IToDoListService` interface. It has business logic and handles the communication between controller and repository layer.

All methods in this class are async operation, following the same patterns as in the controller.

I am injecting&nbsp;`IToDoListRepository`&nbsp;via constructor. It isolates database calls from the service layer, as a result, leaves the class testable.

Here are the&nbsp;`ToDoList`&nbsp;and&nbsp;`Task`&nbsp;entities:

~~~csharp
public class ToDoList
{
    public int Id { get; set; }

    [Required, MaxLength(250)]
    public string Name { get; set; }

    public virtual List<Task> Tasks { get; set; }
}

public class Task
{
    public int Id { get; set; }

    public int ToDoListId { get; set; }

    [Required, MaxLength(500)]
    public string Name { get; set; }

    [MaxLength(4000)]
    public string Description { get; set; }

    public bool Done { get; set; }

    public virtual ToDoList ToDoList { get; set; }
}
~~~

## **Unit Tests For ToDoListController**

In this section we are going to create unit tests for `ToDoListController`.

I choose xUnit framework, but you can choose whatever unit test framework you like. For the assertions I am using[FluentAssertions](https://fluentassertions.com/) framework, which helps in the unit test readability. You can check[my post about FluentAssertions](https://www.matheus.ro/2018/04/16/improving-unit-test-readability-in-net-with-fluent-assertions/) to see in detail.

Finally, I’m using the[&nbsp;FakeItEasy](https://github.com/FakeItEasy/FakeItEasy) mocking framework, to create mocks, stubs and fakes. I choose FakeItEasy instead of other mocking frameworks because it has a simpler syntax.

### **Index Action View Return Type Is ViewResult**

~~~csharp
public async Task<IActionResult> Index()
{
    return View(await _toDoListService.ToDoLists());
}
~~~

We are going to start testing the Index action, which returns to do lists to the view, see the unit test in the following code:

~~~csharp
[Fact]
public void Index_ReturnViewResult()
{
    //Arrange
    IToDoListService toDoListServiceFake = A.Fake<IToDoListService>();
    A.CallTo(() => toDoListServiceFake.ToDoLists())
        .Returns(new List<ToDoList>());

    ToDoListController sut = CreateSut(toDoListServiceFake);

    //Act
    IActionResult result = sut.Index().Result;

    //Assert
    result
        .Should()
        .BeOfType<ViewResult>();
}

private ToDoListController CreateSut(IToDoListService toDoListServiceFake)
{
    return new ToDoListController(toDoListServiceFake);
}
~~~

In this test we use the AAA pattern. First, we Arrange all data necessary to exercise the sut (system under test). Execute the action (Act) and, finally Assert if the result is what we expect.

In the arrange, we instantiate a fake of our&nbsp;`IToDoListService`&nbsp;and set up&nbsp;`ToDoLists()`&nbsp;method to return an empty list of to dos. Since we are creating unit tests for the controllers, we will create fakes for all ToDoListController dependencies. Then, we create&nbsp;`ToDoListController`&nbsp;instance(sut), sending the&nbsp;`toDoListServiceFake`&nbsp;as parameter.&nbsp;`CreateSut()`&nbsp;creates the sut instance. As we will need to create it in all tests, it isolates in one method and every test can use it, [you can check more about sut factory here](https://www.matheus.ro/2017/09/25/sut-factory/).

Moving on, in the Act phase is calling the&nbsp;`Index()`&nbsp;action from the controller and storing the result in a variable, so we can check it. A side note here, the .Result after calling the action is to return the actual result of the action, since it is an async operation.

Finally, we check if the result is of the&nbsp;`ViewResult`&nbsp;type. FluentAssertions helps a lot here, leaving the assertion simple and human readable.

The next tests will follow the same structure.

### **Index Action Model Return Type Is ToDoList’s List**

~~~csharp
public async Task<IActionResult> Index()
{
    return View(await _toDoListService.ToDoLists());
}
~~~

The index action returns a model to the view and we can verify if the action returns the right model type.

~~~csharp
[Fact]
public void Index_ReturnTypeIsListOfToDoList()
{
    //Arrange
    IToDoListService toDoListServiceFake = A.Fake<IToDoListService>();
    A.CallTo(() => toDoListServiceFake.ToDoLists())
        .Returns(new List<ToDoList>());

    ToDoListController sut = CreateSut(toDoListServiceFake);

    //Act
    IActionResult result = sut.Index().Result;

    //Assert
    result
        .As<ViewResult>()
        .Model
        .Should()
        .BeOfType<List<ToDoList>>();
}
~~~

This test is almost the same as the last one, the difference is in the assertion. Instead of asserting the result directly, we use&nbsp;`.As<ViewResult>()`&nbsp;to cast the result to&nbsp;`ViewResult`&nbsp;and then we can access&nbsp;`ViewResult`&nbsp;properties, like the Model. Now, we check if the model type is `List<ToDoList>`.

### **Index Action Return Default ViewName**

~~~csharp
public async Task<IActionResult> Index()
{
    return View(await _toDoListService.ToDoLists());
}

[Fact]
public void Index_ReturnDefaultViewName()
{
    //Arrange
    IToDoListService toDoListServiceFake = A.Fake<IToDoListService>();
    A.CallTo(() => toDoListServiceFake.ToDoLists())
        .Returns(new List<ToDoList>());

    ToDoListController sut = CreateSut(toDoListServiceFake);

    //Act
    IActionResult result = sut.Index().Result;

    //Assert
    result
        .As<ViewResult>()
        .ViewName
        .Should()
        .BeNull();
}
~~~

This one is a little trick. When we return `View(Object)`, the framework knows that it should look for a view that has the same name as the action. In this case is the&nbsp;`Index`&nbsp;view. The&nbsp;`View()`&nbsp;method has an overload with one more parameter, where we can specify the view name.

In our&nbsp;`Index`&nbsp;action, we do not specify the view name, so it will be null. That is the default value when not specified.

### **Index Action Return One ToDoList**

~~~csharp
public async Task<IActionResult> Index()
{
    return View(await _toDoListService.ToDoLists());
}

[Fact]
public void Index_ReturnOneToDoList()
{
    //Arrange
    IToDoListService toDoListServiceFake = A.Fake<IToDoListService>();
    List<ToDoList> toDoLists = new List<ToDoList>
    {
        CreateToDoListDefault()
    };

    A.CallTo(() => toDoListServiceFake.ToDoLists())
        .Returns(toDoLists);

    ToDoListController sut = CreateSut(toDoListServiceFake);

    //Act
    IActionResult result = sut.Index().Result;

    //Assert
    result
        .As<ViewResult>()
        .Model
        .As<List<ToDoList>>()
        .Should()
        .HaveCount(1);
}

private ToDoList CreateToDoListDefault()
{
    return new ToDoList
    {
        Id = 1,
        Name = "ToDo"
    };
}
~~~

In this test we are checking if the action returns a to do list to the screen.

First, in the arrange, we create a&nbsp;`List<ToDoList>`&nbsp;instance containing one `ToDoList`. I created the helper method&nbsp;`CreateToDoListDefault()`&nbsp;to facilitate the to do list creation.

After that, we set up&nbsp;`ToDoLists()`&nbsp;method to return&nbsp;`toDoLists`&nbsp;object.

And then, in the assertion, we cast the result to&nbsp;`ViewResult` as usual, to access its model. Finally, we cast the model to&nbsp;`List<ToDoList>`, and then we assert against it and check if it contains one to do list in the response.

### **Edit Get Action Return Not Found Status Code**

~~~csharp
public async Task<IActionResult> Edit(int? id)
{
    if (id == null) return NotFound();

    var toDoList = await _toDoListService.FindById(id.Value);

    if (toDoList == null) return NotFound();

    return View("Edit", toDoList);
}

[Fact]
public void Edit_ReceiveNullId_ReturnNotFoundStatusCode()
{
    //Arrange
    int notFoundStatusCode = 404;
    IToDoListService toDoListServiceFake = A.Fake<IToDoListService>();
    ToDoListController sut = CreateSut(toDoListServiceFake);

    //Act
    IActionResult result = sut.Details(null).Result;

    //Assert
    result
        .As<NotFoundResult>()
        .StatusCode
        .Should()
        .Be(notFoundStatusCode);
}
~~~

Fail paths are important to validate and that is what we are doing in this unit test.

We are exercising the http get version of the `Edit`&nbsp;action, where we load the edit view based on an id to load a particular to do list. The test checks when the action receives a null value, if it returns a not found result with a status code 404 (not found).

To being with, we create a variable to store the expected result, which is `notFoundStatusCode`. Create a fake&nbsp;`A.Fake<IToDoListService>()`&nbsp;and the sut as usual, but this time we do not need to set up any fake method, since the path we are exercising do not use the fake.

So, in the assert phase, we cast the result to&nbsp;`NotFoundResult`&nbsp;and check if the result returns not found status code.

### **Edit Get Action Return Edit View**

~~~csharp
public async Task<IActionResult> Edit(int? id)
{
    if (id == null) return NotFound();

    var toDoList = await _toDoListService.FindById(id.Value);

    if (toDoList == null) return NotFound();

    return View("Edit", toDoList);
}

[Fact]
public void Edit_ToDoListExists_ReturnEditViewName()
{
    //Arrange
    int toDoListId = 1;
    ToDoList toDoList = CreateToDoListDefault();
    IToDoListService toDoListServiceFake = A.Fake<IToDoListService>();
    A.CallTo(() => toDoListServiceFake.FindById(A<int>.Ignored))
        .Returns(toDoList);

    ToDoListController sut = CreateSut(toDoListServiceFake);

    //Act
    IActionResult result = sut.Edit(toDoListId).Result;

    //Assert
    result
        .As<ViewResult>()
        .ViewName
        .Should()
        .Be("Edit");
}
~~~

Now, we are checking if the edit get action is returning the right view.

As I mentioned before,&nbsp;`View()`&nbsp;has an overload that we can send the view name and the object. In the edit action we are specifically saying to return the edit view.

In the assertion, we look at the&nbsp;`ViewName`&nbsp;property to verify if the action returns the&nbsp;`Edit`&nbsp;view.

### **DeleteConfirm Action Return Type is RedirectToActionResult**

~~~csharp
[HttpPost, ActionName("Delete")]
[ValidateAntiForgeryToken]
public async Task<IActionResult> DeleteConfirmed(int id)
{
    string listName = await _toDoListService.Remove(id);

    this.AddAlertSuccess($"{listName} removed successfully.");

    return RedirectToAction(nameof(Index));
}

[Fact]
public void DeleteConfirm_ValidToDoListId_RedirectToIndexView()
{
    //Arrange
    int toDoListId = 1;
    string toDoName = "ToDo List Unit Test";
    IToDoListService toDoListServiceFake = A.Fake<IToDoListService>();
    A.CallTo(() => toDoListServiceFake.Remove(A<int>.Ignored))
        .Returns(toDoName);

    ToDoListController sut = CreateSut(toDoListServiceFake);
    sut.TempData = new TempDataDictionary(A.Fake<HttpContext>(), A.Fake<ITempDataProvider>());

    //Act
    IActionResult result = sut.DeleteConfirmed(toDoListId).Result;

    //Assert
    result
        .Should()
        .BeOfType<RedirectToActionResult>();
}
~~~

`DeleteConfirmed`&nbsp;action is responsible to delete a to do list from the database based on the to do list id and redirect the user to the index action.

The reason&nbsp;`Remove()`&nbsp;return to do list name is because in the line below `this.AddAlertSuccess($"{listName} removed successfully.");`, we send a success alert message to the screen and we need the list name for that.

This test needs a little more set up, because we have to set up the sut’s&nbsp;`TempData`&nbsp;property because the alert uses it. So, we configure&nbsp;`Remove()`&nbsp;method to return a list name and create the sut. Then, we create a new&nbsp;`TempDataDictionary`&nbsp;to set up the&nbsp;`TempData`&nbsp;with some fake objects, since we don’t need &nbsp;them for this unit test.

After executing the action, we check if the result type is&nbsp;`RedirectToActionResult`&nbsp;with `.BeOfType<T>()`.

### **Edit Post Action Throws DbUpdateConcurrencyException**

~~~csharp
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
        bool todoExists = await _toDoListService.Exists(id);

        if (!todoExists)
            return NotFound();
        else
            throw;
    }

    this.AddAlertSuccess($"{toDoList.Name} updated successfully.");
    return RedirectToAction(nameof(Index));
}

[Fact]
public void EditPost_ConcurrencyInUpdateAndToDoListExists_ThrowsDbUpdateConcurrencyException()
{
    //Arrange
    const int toDoListId = 1;
    ToDoList toDoList = CreateToDoListDefault();
    IToDoListService toDoListServiceFake = A.Fake<IToDoListService>();

    DbUpdateConcurrencyException exception = 
        new DbUpdateConcurrencyException(
            "Update concurrency exception",
            new List<IUpdateEntry> { A.Fake<IUpdateEntry>() });

    A.CallTo(() => toDoListServiceFake.Update(A<ToDoList>.Ignored))
        .ThrowsAsync(exception);

    A.CallTo(() => toDoListServiceFake.Exists(A<int>.Ignored))
        .Returns(true);

    ToDoListController sut = CreateSut(toDoListServiceFake);

    //Act
    Func<Task<IActionResult>> action = () => sut.Edit(toDoListId, toDoList);

    //Assert
    action
        .Should()
        .Throw<DbUpdateConcurrencyException>();
}
~~~

We can verify if a method is throwing an exception and FluentAssertions make it easy.

There is a lot going on in the edit post action, but it has the goal of update the to do list in the database and redirect the user to the index view. The catch is handling the&nbsp;`DbUpdateConcurrencyException`&nbsp;which&nbsp;``&nbsp;throws when a concurrency issue happens.

Inside the catch we check if that to do list exists. If not exists it returns a&nbsp;`NotFound()`&nbsp;object and if the to do exists it just throw the exception. We are going to exercise the path where it throws the exception.

To start, the arrange is convoluted because we need a lot of data set up. We have the basic set up, like creating a default to do list and our fake `IToDoListService`. Now, to set up&nbsp;`Update()`&nbsp;method we have instantiate the exception first. It is trick to instantiate `DbUpdateConcurrencyException`. It receives the exception message and a list of&nbsp;`IUpdateEntry`&nbsp;as parameters. I found easier to just fake the a&nbsp;`IUpdateEntry` entry on the list. When setting up&nbsp;`.Update()`&nbsp;instead of calling &nbsp;`.Returns()`, we call&nbsp;`.ThrowsAsync()`&nbsp;with the exception.

Moving to the act phase, instead of executing the function, we are creating a function which store the edit action. The reason why I choose to store it in a function is because we do not want to call the function and throw the exception in this test’s section.

In the assertion, we call the function to execute it and we verify if it threw the exception. This is where FluentAssertions becomes pretty handy, with&nbsp;`.Throw<DbUpdateConcurrencyException>()`&nbsp;we can check if the function threw the exception, without needing to put some sort of _try/catch_ block in the unit test.

### **Edit Post Action Return Model State Error**

~~~csharp
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
        bool todoExists = await _toDoListService.Exists(id);

        if (!todoExists)
            return NotFound();
        else
            throw;
    }

    this.AddAlertSuccess($"{toDoList.Name} updated successfully.");
    return RedirectToAction(nameof(Index));
}

[Fact]
public void EditPost_InvalidModalState_ReturnViewWithOneModelStateError()
{
    //Arrange
    int toDoListId = 1;
    ToDoList toDoList = new ToDoList
    {
        Id = 1
    };

    IToDoListService toDoListServiceFake = A.Fake<IToDoListService>();

    ToDoListController sut = CreateSut(toDoListServiceFake);

    string modelStateErrorKey = "Name";
    sut.ModelState.AddModelError(modelStateErrorKey, "Name is required.");

    //Act
    IActionResult result = sut.Edit(toDoListId, toDoList).Result;

    //Assert
    result
        .As<ViewResult>()
        .ViewData
        .ModelState[modelStateErrorKey]
        .Errors
        .Should()
        .HaveCount(1);
}
~~~

Continuing testing the edit action, for the last scenario, we are going to validate the path where we have an invalid model state.

To have an invalid model state, we have to add a model error into the controller’s&nbsp;`ModelState`&nbsp;property. The&nbsp;`ModelState` is a dictionary, so we need a key, that is the property's name and an error message.

In the test, I created a to do list without name and adding an error to the&nbsp;`ModelState `indicating that the name is required.

Just a side note here, when running the system, ASP.NET core MVC fills the&nbsp;`ModelState` automatically based on the user inputs on the screen. For example, if the user is editing a to do list on the screen but does not fill the name field, when he submits the form,&nbsp;`ModelState` will have the same error added on the test.

Then we execute the action and check if the object has one error. To access `ModelState` we cast the result to&nbsp;`ViewResult`&nbsp;and access the&nbsp;`ViewData`&nbsp;property. We use the key created earlier to get the right&nbsp;`ModelState` entry. Finally, we check if `ModelState` contains one error.

## **Conclusion** 

In this post I showed to you the most common ways to unit test an ASP.NET core MVC controller. Using the FluentAssertions framework to improve unit test readability.

See you in the next one!

[Photo by Vadim Sherbakov](https://unsplash.com/@madebyvadim?utm_medium=referral&utm_campaign=photographer-credit&utm_content=creditBadge "Download free do whatever you want high-resolution photos from Vadim Sherbakov")


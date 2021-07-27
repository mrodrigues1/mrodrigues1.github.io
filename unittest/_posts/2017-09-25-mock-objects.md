---
layout: post
title: Mock Objects
date: 2017-09-25 01:19:23.000000000 -03:00
categories: unittest
tags: mock fakeiteasy
permalink: "/2017/09/25/mock-objects/"
---
In my [last post](https://www.matheus.ro/2017/09/25/what-are-stubs/) I talked about stubs and how to you can use them to isolate external dependencies in your unit tests. In this post I want to talk about another type of test double, Mock Objects.

A Mock object is basically a fake object that simulates behavior of real object in controlled ways. The main difference between mock and stubs is that when using a mock you can verify if the object under test called the fake object as expected, more about that later. When you make test like that, you are creating Interaction Tests, as the name says, you are testing the interactions between components in the system.

I don’t recommend the use of interaction tests, it will be better if you choose this type of test as your last option, because when your system starts to grow, the interactions can change a lot

Value-base and state-based tests are a preferable type of tests, because the first checks the value returned from a function and the second is about checking for noticeable behavior of the system under test, after changing its state.

I will show you two examples of mocks in unit tests, the first will be a handwritten mock example, just to show how we can write a mock but I don't recommend to use it, and following that, i’m gonna show an example using an isolation framework with&nbsp;[FakeItEasy](https://fakeiteasy.github.io/).

## **Code sample**

~~~csharp
public class TodoList
{
    private ILogger _logger;    

    public TodoList(ILogger logger)
    {
        _logger = logger;
    }

    public List<Task> TaskList { get; set; }

    public void AddTask(Task task)
    {
        if(string.IsNullOrEmpty(task.Name))
        {
            _logger.LogError("Task name is mandatory.");
            throw new ArgumentNullException();
        } else
        {
            TaskList.Add(task);
        }
    }
}
~~~

~~~csharp
public interface ILogger
{   
    void LogError(string message);
}
~~~

~~~csharp
public class Task
{
    public string Name { get; set; }
}
~~~

In this code sample, we have the _TodoList_ class containing a method called _AddTask_, in this method we log errors if the task has an empty name using _ILogger_ interface and throw an exception, and finally when the task is valid, we add then to the task list.

## **Handwritten mock&nbsp;example**

~~~csharp
public class TodoListTests
{
    [Fact]
    public void AddTask_InvalidTask_LogError()
    {
        //Arrange        
        MockLogger mockLogger = new MockLogger();
        TodoList todo = new TodoList(mockLogger);
        Task task = new Task();

        //Act
        var result = Assert.Throws<ArgumentNullException>(
            () => todo.AddTask(task));

        //Assert
        Assert.Equal("Task name is mandatory.", mockLogger.LastError);
    }
}
~~~

~~~csharp
public class MockLogger : ILogger
{
    public string LastError { get; set; }

    public void LogError(string message)
    {
        LastError = message;
    }
}
~~~

As we can see in the test, instead of sending a concrete implementation of the _ILogger_ interface, we send a fake implementation. By doing that, we’re able to verify what message was sent to the logger when a task have an empty name, using the _LastError_ property.

This is a simple example of how to write a mock, but as said early, i don't recommend to use handwritten mocks, it's better to use an isolation framework to do the work for you.

## **Using isolation (mocking) framework**

In the previous section, we look at write mocks manually. In the this section, we’ll look at a more elegant solution in the form of isolation frameworks. They‘re basically a reusable library that can create and configure fake objects at runtime. These objects are referred as dynamic mocks and dynamic stubs.

For this example we’re using the FakeItEasy framework, it’s a well documented and easy to use isolation framework, i’ll recommend some other frameworks later. Now back to the test.

~~~csharp
public class TodoListIsolationFrameworkTests
{
    [Fact]
    public void AddTasks_InvalidTask_CallLogError()
    {
        //Arrange        
        ILogger mockLogger = A.Fake<ILogger>();
        A.CallTo(() => mockLogger.LogError(A.<string>.Ignored));

        TodoList todo = new TodoList(mockLogger);
        Task task = new Task();

        //Act
        var result = Assert.Throws<ArgumentNullException>(
            () => todo.AddTask(task));

        //Assert
        A.CallTo(() => mockLogger.LogError(A.<string>.Ignored)).MustHaveHappened();
    }
}
~~~

We refactored the previous example to use FakeItEasy. Now we need to call:&nbsp;`A.Fake<ILogger>();`

With this we can create a configurable dynamic object of our interface for your tests.

To configure the object: `A.CallTo(() => mockLogger.LogError(A..Ignored));`

Here we’re configuring the call to the _LogError_ method. The `A.CallTo()` means that when we make a call a certain method of our mock object, we’re expecting something to happen, as in this case _LogError_ is a void method, we don’t need configure a return for the method. Also we can configure the parameters that the method is receiving, in this case we don’t care about what is being send, so we can just do this: `A.Fake<string>.Ignored`.

Doing that means that whatever string we are sending, the method will works fine. In the assert fase of the test we verify if _LogError_ method was called during the test execution.

~~~csharp
A.CallTo(() =>mockLogger.LogError(A<string>.Ignored)).MustHaveHappend();
~~~

It’s almost the same call as when we’re configuring the method, the only difference here is the `MustHaveHappened()`, this verify if the _LogError_ method was called during the test execution.

## **Using isolation framework as a&nbsp;stub**

Imagine if we have to implement a new requirement for our todo list, like get all the tasks from a database this time, the class will look something like this:

~~~csharp
public class TodoList
{
    private ILogger _logger;    
    private IDatabase _db;

    public TodoList(ILogger logger, IDatabase db)
    {
        _logger = logger;
        _db = db;
    }    

    public void AddTask(Task task)
    {
        if(string.IsNullOrEmpty(task.Name))
        {
            _logger.LogError("Task name is mandatory.");
            throw new ArgumentNullException();
        } else
        {
            _db.AddTask(task);
        }
    }

    public List<Task> GetAllTasks()
    {
        return _db.GetAllTasks();
    }
}
~~~

Now we have a new external dependency on the class, _IDatabase_, and the new method _GetAllTasks()_, that calls the database to get all tasks.

~~~csharp
[Fact]
public void GetAllTasks_ReturnTasks()
{
    //Arrange        
    ILogger mockLogger = A.Fake<ILogger>();
    IDatabase mockDatabase = A.Fake<IDatabase>();

    List<Task> tasks = new List<Tasks>
    {
        new Taks { Name = "Tasks 1"; },
        new Taks { Name = "Tasks 2"; },
        new Taks { Name = "Tasks 3"; }
    }

    A.CallTo(() => mockDatabase.GetAllTasks()).Returns(tasks);

    TodoList todo = new TodoList(mockLogger, mockDatabase);

    //Act
    var result = todo.GetAllTasks();

    //Assert
    Assert.Equal(3, result.Count);
}
~~~

To test the new method _GetAllTasks_, we have to create mocks for _ILogger_ and _IDatabase_ dependencies, _ILogger_ can be just a dummy object, with no configuration, for the _IDatabase_ interface we need to configure the _GetAllTasks_ method. To configure method with return values it’s pretty similar as we saw in the previous examples, the main difference here is that we call `Returns()` after CallTo, that enable us to return data when the mock method is called inside the test execution.

In the assert fase, we just make an assertion that verify if the data returned has the same amount of tasks as we configure in the mock.

This is just an example of how can you use an isolation framework as a stub to return data, it’s basically a Value-based test, because you’re not making the assertion against the mock, just asserting if the returning data was correct.

## **Isolation frameworks for .net**

FakeItEasy is the not only isolation framework in the .net, I also recommend [Moq](https://github.com/moq/moq4) and [NSubstitute](https://nsubstitute.github.io/), these are very good isolation frameworks to checkout.

## **Further reads and references**

- [The Art of Unit Testing, 2nd Edition](https://www.manning.com/books/the-art-of-unit-testing-second-edition)
- [FakeItEasy documentation](https://fakeiteasy.readthedocs.io/en/stable/)
- [Mocks Aren’t Stubs, by Martin Fowler](https://martinfowler.com/articles/mocksArentStubs.html)
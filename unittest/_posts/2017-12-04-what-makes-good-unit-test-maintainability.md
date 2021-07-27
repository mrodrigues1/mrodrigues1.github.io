---
layout: post
title: What Makes Good Unit Test? Maintainability
date: 2017-12-04 08:00:19.000000000 -03:00
type: post
categories: unittest
tags: maintainability dry mock namingconvention sut
permalink: "/2017/12/04/what-makes-good-unit-test-maintainability/"
image: 
  path: /assets/img/blog/What-Makes-Good-Unit-Test_-Maintainability.png
description: >
  How maintainability can make good unit tests.
---
In this post I'm going to talk about how we can write more maintainable unit tests.

Maintainability is of the main issues that we face when writing unit tests. The number of unit tests can grow a lot, and become harder and harder to maintain and understand, every little change on the code seem to break one test or another, even if you don't introduce a bug in the code.

This post's focus is to present some techniques to create more maintainable unit tests.

## **Removing Duplication**

Code duplication in tests are bad as duplication in production code. The DRY(Don't Repeat Yourself) should be applied in the test code, in the same way as in production code.

One of the techniques to avoid code duplication in the tests is to create a SUT factory. A SUT factory is an auxiliary method or a class that is in charge of creating the system under test instance. I wrote a post talking about the SUT factory more in-depth, you can check it [here](https://www.matheus.ro/2017/09/25/sut-factory/).

To explain briefly in this post, a SUT factory is used to reduce duplication and improve the maintainability of our unit tests by concentrating in one place the sut's creation.

Given the test:

~~~csharp
[TestMethod]
public void GenericTestUsingConstructor()
{
    //Arrange
    MySUT sut = new MySUT();
 
    //Act
    //...Exercise  the sut

    //Assert
    //...Verifying the results
}
~~~

Now, what if MySUT class got some new dependencies?

We're going to have to update all tests that involves MySUT class, this is not very maintainable. We could create a SUT factory in this case:

~~~csharp
public MySUT CreateMySUT()
{
    var fakeDependency = new FakeDependency();

    return new MySUT(fakeDependency);
}
~~~

And them use it in out test:

~~~csharp
[TestMethod]
public void GenericTestUsingSUTFactory()
{
    //Arrange
    MySUT sut = CreateMySUT();

    //Act
    //...Exercise  the sut

    //Assert
    //...Verifying the results
}
~~~

With this technique we can reduce a lot of code duplication in the test's arrange phase. We can have more than one SUT factory for each SUT, for example, we could pass parameters to create more customizable suts.

Another way to reduce the code duplication in the arrange phase is to use the built-in setup method, to initialize all the tests dependencies.

It'll be something like this:

~~~csharp
public class GenericTests
{
    private MySUT _sut;

    [TestInitialize]
    public void SetupGenericTests()
    {
        var fakeDependency = new FakeDependency();

        _sut = new MySUT(fakeDependency);
    }

    [TestMethod]
    public void GenericTestUsingSetup()
    {        
        //Act
        //...Exercise  the sut

        //Assert
        //...Verifying the results
    }
}
~~~

Don't do this!

Setting up the arrange phase using the TestInitialize method is good to reduce code duplication, but as soon as our class start to grow, this approach become hard to maintain. Each unit test has its own particularity, if we use only one method for the arrange phase, we'll end up setting up stuff that has nothing to do with a particular unit test. Be careful in the way that you use the setup method, isn't always a good idea, I'll show in the next section why.

## **Using Setup Methods In a Maintainable Way**

We've already saw a sample of how to use the setup method in last section, now I want to show the Do's and Don'ts of this method's type.

The setup method is easy to implement and use. It's so easy, that sometimes can be used for things that wasn't meant for, and the tests become less readable and maintainable.

Setup methods have limitations:

- They aren't always the best case for removing duplication. Sometimes you need to remove duplication in other areas like, asserting logic, creating the sut.
- They can't have parameters or return values.
- Can't be used as factory methods. They run before the test execution, so they need to be more generic. Test sometimes have some specific things, or call a separated method with a specific parameter (for example, retrieve an object and set its properties).
- Can only contain code that applies to all tests in the current test class.

Now that we listed some of the setup method limitations, let's see how some developers try to get around them in their journey to use setup methods no matter what, instead of using helper methods.

## **Initializing Objects That Are Only Used By Some Tests**

I have seen test suites that had one test class for each class, so far, so good. But the setup method on these classes were huge, trying to set up almost every dependency for every single test, when not all tests needed all dependencies.

The setup method is used to initialize everything related to all tests. Using the setup to initialize all the dependencies, we ended up with a lengthy and hard to understand setup method. A solution to this problem is to refactor calls to specific things into helper methods and call them in the setup method or in the specific test.

It'll be something like this:

~~~csharp
private MySUT _mySut;
private Mock _mock;
private Mock _mockTwo;
private List _testDataOne;
private List _testDataTwo;

[TestInitialize]
public void BadInitializeMethod()
{
    _mock = new Mock();
    _mockTwo = new Mock();

    _testDataOne = new List
    {
        new TestDataOne(),
        new TestDataOne(),
        new TestDataOne()
    };

    _testDataTwo = new List
    {
        new TestDataTwo(),
        new TestDataTwo(),
        new TestDataTwo()
    };

    _mock.Setup(x => x.GetData()).Returns(_testDataOne);

    _mockTwo.Setup(x => x.GetData()).Returns(_testDataTwo);

    _mySut = new MySUT(_mock.Object, _mockTwo.Object);
}
~~~

Bad Setup Method

Other thing that I want to point it out, is to not arrange fakes inside the setup method. This will just make the test even harder to maintain and read.

It's better if each test setup its own mocks and stubs by calling helper method within the test, doing so will the test more read, because the reader will exactly what's happening, without the need to be jumping back and for between the test and the setup method.

At the time that I'm writing this article, I almost completely stop using setup methods. They not help to write unit tests that are maintainable and easy to read, normally its generate the complete opposite. The test code should nice and clean, like the production code. If your production code is crap, don't use it as an excuse to write unmaintainable unit tests. Use factory and helper, and things will be better for everyone.

## **Isolating Tests**

The basic concept of test isolation is that each test should run in its own small world, isolated from every other test that may do similar or different things. Lack of, or none isolation is one of the biggest reasons to end up with bad unit tests.

When the tests aren't isolated very well, it'll become a pain for you. Problems will appear and the developer will have no idea where they came from, the tests can become so connected that after some time the developer get tired of it and give up on trying to fix the problem.

The lack of isolation is caused by several tests "smells" and I'm going to enumerate some of them.

### **Constrained Test Order**

Tests that are expected to run in a particular order or expecting other test's results.

This often occurs when the tests uses global or static data. When comes the time for the test run, he's expecting some code to be on an exact state or memory, for example, in a global variable.

The problem is that most of the test platform don't guarantee that the tests will run in a certain order, so if the test pass today, but can fail tomorrow.

Many problems can occur when tests don't enforce isolation. Running subsets of tests can cause different results than running all tests. Removing or change a tests may affect the result of others. Maintain tests become harder, because you need to see how the tests are related to each other, and how each affects the state.

One possible solution is to find the data being&nbsp;shared, extract to a function and replace with a fake.

### **Shared-state Corruption**

Tests that shares objects in-memory without any rollback before each execution.

Imagine a simple set of tests for a CRUD (Create/Read/Update/Delete). The tests are going to share the same object in-memory to manipulate the data.

~~~csharp
public class ShareContextTests
{
    private MyContext _context;

    [TestInitialize]
    public void Initialize()
    {
        _context = new MyContext();
    }


    [TestMethod]
    public void ContextAdd()
    {
        //Arrange
        var testData = new TestDataOne();

        //Act 
        _context.Add(testData);
        var result = _context.FindById(1);

        //Assert            
        Assert.IsNotNull(result);

    }

    [TestMethod]
    public void ContextEdit()
    {
        //Arrange
        var testData = _context.FindById(1);
        testData.Name = "Change Name";

        //Act 
        _context.Update(testData);
        var result = _context.FindById(1);

        //Assert            
        Assert.AreEqual("Change Name", result.Name);
    }
}
~~~

In this example, the code of the second test is expecting that the object was already loaded to do the update. We can't guarantee that the test ContextAdd will run before the test ContextEdit.

This problem is pretty similar as the last one. We can get different results when we run all tests or a small test subset. Make a change to a test can be a problem, because we need to be careful and see in which way a test in related to others.

Unit test shouldn't assume that a shared object is in a clean state. The test should re-create at beginning or rollback at end of test, the object in memory, to guarantee clean state for the object.

### **External Shared-state Corruption**

Tests with shared external resources and without rollback.

This is very like the shared-state corruption smell, the difference is that this smell is related to integration style testing. Tests that rely on databases, web services or external files.

The solution is to extract methods that interacts with some external resource, maybe to its own class, and mock it.

## **Avoiding Multiple Asserts On Different Concerns**

I already talked about this in my last [post](https://www.matheus.ro/2017/10/23/unit-test-reliability/), in which I discuss this topic in more depth. The main idea is to test only one thing at time.

Sometimes we put more than one assert in a unit test, thinking about saving some time and writing less code. What if this test fails? How can we know which assertion fail?

When one assertion fails the following assertions will not run, this means that we don't care about the other assertions that didn't run? of course, we care.

There are two ways to solve this problem:

- Create separate unit test for each assertion.
- Use parameterized unit tests.

## **Over Specification**

Over specification in unit tests means that a stub is being used as a mock object (stubs are object that we'll not assert against, we use it to test something else). I've already written post about [stubs](https://www.matheus.ro/2017/09/25/what-are-stubs/)&nbsp;and [mocks](https://www.matheus.ro/2017/09/25/mock-objects/), if you want to go deeper in these topics.

Look at this example:

~~~csharp
[Fact]
public void Call_Return_A_Book()
{
    //Arrange
    var mockBookRepository = new Mock<IBookRepository>();

    var book = new Book
    {
        Id = 1,
        Name = "The Lord of the Rings"
    };

    mockBookRepository.Setup(x => x.GetBook(It.IsAny<int>())).Returns(book);

    var sut = new BookService(mockBookRepository.Object);

    //Act
    var result = sut.GetBook(1);

    //Assert
    mockBookRepository.Verify(x => x.GetBook(It.IsAny<int>()), Times.Once);

    Assert.Equal(book.Name, result.Name);
}
~~~

In this example we are setting up a method of a stub to return data, normal stuff.

`mockBookRepository.Setup(x => x.GetBook(It.IsAny<int>())).Returns(book);`

The problem is on the arrange phase.

`mockBookRepository.Verify(x => x.GetBook(It.IsAny<int>()), Times.Once);`

`Assert.Equal(book.Name, result.Name);`

What we want to test is the assert line. Calling verify on the GetBook is making the test fragile. This line is turning this into an interaction test, and does it matter whether&nbsp;the GetBook is getting called in the test execution? No. The only line that matters is the assert line.

The author Roy Osherove have tree general rules to verify over specification in tests:

- If we have "Verify" and "Assert" in the same test, it usually a signal of a test over specification.
- When we have the "Setup" and "Return" on a fake object, make sure to name it correctly, like "stubYY" or "mockYY", so we know in which object we want to call verify in the assertion phase (quick tip, usually don't call "Verify").
- Try to test only the result instead of verify the interaction. The only case that you need to verify if a method has been called is in the case of a&nbsp;_void_ method, since they don't have return, the only way to communicate is to verify if it has been called.

I already saw a lot of developers saying that adding an extra "verify" it's a good thing, since we're testing more things. Of course, that we're testing more things, but they are internal things, the interactions and the app behavior, change a lot during the development.

## **Conclusion**

By doing this small adjustments, you can guarantee&nbsp;that your unit tests will become more maintainable, and they'll not be so fragile. It can be long process, but try to improve one thing at a time we'll help you a lot in the future.

This concludes my post about maintainability in unit testing.

## **References and Further Reading**

- [The Art of Unit Testing, Second Edition - Roy Osherove - Chapter 8](https://www.manning.com/books/the-art-of-unit-testing-second-edition)
- [Unit Testing: Setup Methods or Not?](https://jeremybytes.blogspot.com.br/2015/06/unit-testing-setup-methods-or-not.html)
- [Unit Testing: SUT Setup Methods and Code Maintainability](http://www.michaelgmccarthy.com/2016/08/03/unit-testing-sut-setup-methods-and-code-maintainability/)
- [Is Your Unit Test Isolated? - Agile in a Flash](http://agileinaflash.blogspot.com.br/2012/04/is-your-unit-test-isolated.html)
- [Interaction Testing With Mocks Leads To A Less Maintainable Unit Test](http://hamidmosalla.com/2017/11/28/interaction-testing-with-mocks-leads-to-a-less-maintainable-unit-test/)
- [Over Specification in&nbsp;Tests - Roy Osherove](http://osherove.com/blog/2008/7/12/over-specification-in-tests.html)
- [SUT Factory](https://www.matheus.ro/2017/09/25/sut-factory/)

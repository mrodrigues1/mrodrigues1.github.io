---
layout: post
title: SUT Factory
date: 2017-09-25 03:57:55.000000000 -03:00
categories: unittest
tags: sut
permalink: "/2017/09/25/sut-factory/"
description: >
  What is the best way to create the system under test?
---
SUT factory may be a fancy name to what is basically a utility method, used to create an instance of the system under test. The main point of a SUT Factory is to reduce code duplication and improve maintainability of your unit tests suite.

I’ll show you a quick example of how can you implement this type of utility method.

Imagine a test like this one:

~~~csharp
[TestMethod]
public void TestUsingConstructor()
{
    //Arrange
    MySUT sut = new MySUT();

    //Act
    //...Exercise  the sut

    //Assert
    //...Verifying the results
}
~~~

You can see that in this test we’re just creating our system under test, exercising and checking the outcome, usual stuff. Imagine if a new requirement was requested and we end up with a constructor like this:

~~~csharp
public MySUT(IMyDependency dependeny)
//...
//Rest of the implementation
//...
~~~

Now all of MySUT’s tests that use the default constructors will break. Because they don’t have this dependency, we would’ve to update all tests to account for this new requirement, just to compile the code.

Instead of doing this, we can create a factory method for the class, then we’ll have a single place to update the creation logic:

~~~csharp
private MySUT CreateMySUT()
{
    IMyDependency myDependency = new MyDependencyFake();

    return new MySUT(myDependency);
}
~~~

~~~csharp
[TestMethod]
public void TestUsingFactoryMethod()
{
    //Arrange
    MySUT sut = CreateMySUT();

    //Act
    //...Exercise  the sut

    //Assert
    //...Verifying the results
}
~~~

Every time that MySUT’s constructor needs to be changed, we update the CreateMySUT method. This helps to remove duplication and improve maintainability of your test.

## **References for Further Reading**

- [SUT Factory by Mark Seemann](http://blog.ploeh.dk/2009/02/13/SUTFactory/)
- [xUnit Patterns — Test Utility Method](http://xunitpatterns.com/Test%20Utility%20Method.html)
- [The Art of Unit Testing, 2nd Edition](https://www.manning.com/books/the-art-of-unit-testing-second-edition)

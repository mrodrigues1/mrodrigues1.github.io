---
layout: post
title: Unit Test And Time Dependency
date: 2017-09-25 04:27:50.000000000 -03:00
type: post
categories: unittest
tags: dependencyinjection
permalink: "/2017/09/25/unit-test-time-dependency/"
image: 
  path: /assets/img/blog/unit_test_time_dependency.jpg
description: >
  Have a time sensitive application? It can affect unit tests outcomes.
---
When you start to write unit tests, inevitably you’ll encounter a hard time when a functionality depends on time. Depend on _DateTime.Now_ doesn’t work well for unit testing, your tests will pass on certain time of day and fail in others.

To resolve this problem we need to isolate the time dependency of system, to be able to make our unit test reliable.

## **Example**

So, we have a shop that gives some discount based on the hour of the day, and we have a method that return how much the discount will be:

~~~csharp
public class Shop
{
    //Constructor
    //public Shop(.....

    //Other Implementations
    //.....

    public int GetCurrentDiscount()
    {
        switch (DateTime.Now.Hour)
        {
            case 9:
                return 30;
            case 12:
                return 20;
            case 18:
                return 40;
            default:
                return 0;
        }
    }
}
~~~

The time dependency is on _DateTime.Now.Hour_, the discount is given at 9h, 12h, and 18h.

The implementation of a unit test that wants to verify the discount at 9h, looks like this:

~~~csharp
[Fact]
public void GetDiscount_9hDiscount_ReturnDiscountForTheHour()
{
    //Arrange
    var sut = new Shop();
    var expectedDiscount = 30;

    //Act
    var result = sut.GetCurrentDiscount();

    //Assert
    Assert.Equal(expectedDiscount, result);
}
~~~

Look what happen when I run this test:

![]({{ site.baseurl }}/assets/2017/09/1-UrwcQIjGkDSxHXXV8GqHtw.png){:.lead}

Visual Studio Test Runner – Test Red
{:.figcaption}

Unfortunately, the test fails, because it was expecting the 9h discount but it was executed in another time of the day. In this way the test will only pass when the system’s clock hit 9h.

To able to control the result of the test we can create an interface for the _DateTime_ class and inject it in our _Shop_ class via constructor.

The first thing to do is to define the interface, I named the interface _ITimeProvider_:

~~~csharp
public interface ITimeProvider
{
    DateTime Now();
}
~~~

So, I simply put in the interface one method that will return _DateTime.Now()_. You can create an interface with more methods for your needs but for this example I’ll expose just the _Now()_ method.

The concrete implementation of the interface that will be used in our production code will look like this:

~~~csharp
public class CurrentTimeProvider : ITimeProvider
{
    public DateTime Now()
    {
        return DateTime.Now;
    }
}
~~~

_CurrentTimeProvider_ inherits the interface, and implements the _Now()_ method.

Back to the _Shop_ class, we need to inject the _ITimeProvider_ interface via constructor.

~~~csharp
public class Shop
{
    private readonly ITimeProvider _dateTime;

    public Shop(ITimeProvider timeProvider)
    {
        _dateTime = timeProvider;
    }

    //Other Implementations
    //.....

    public int GetCurrentDiscount()
    {
        switch (_dateTime.Now().Hour)
        {
            case 9:
                return 30;
            case 12:
                return 20;
            case 18:
                return 40;
            default:
                return 0;
        }
    }
}
~~~

Now in the _GetCurrentDiscount_ method we can use the property just like a instance of the _DateTime_.

We need to update our unit test now, since we have a new dependency on _Shop_ class. Basically, we are going to use the Moq framework to create a stub for the _ITimeProvider_, and provide this stub to the Shop class.

~~~csharp
[Fact]
public void GetDiscount_9hDiscount_ReturnDiscountForTheHour()
{
    //Arrange
    var timeProviderStub = new Mock<ITimeProvider>();
    timeProviderStub.Setup(t => t.Now()).Returns(new DateTime(2017, 07, 21).AddHours(9));

    var sut = new Shop(timeProviderStub.Object);
    var expectedDiscount = 30;

    //Act
    var result = sut.GetCurrentDiscount();

    //Assert
    Assert.Equal(expectedDiscount, result);
}
~~~

Using the Setup method of the mock framework, we can setup the interface’s methods and the methods’s return, as you can see. So as the requirement for our test, Now() will return a date that the hour is 9, when he is called in the test’s execution.

Now when we run the test:

![]({{ site.baseurl }}/assets/2017/09/1-836mkSOK2KG7so76snx8MQ.png){:.lead}

Visual Studio Test Runner - Test Green
{:.figcaption}

The test now passes, because we were capable of isolating the time dependency from our test.

## Conclusion

This is a case that you can improve the design of your system to enable unit tests. I think that’s very important because a good code for me, has a lot of different characteristics, and one of then is definitely the ability create unit tests against.

Talking about the time dependency problem in specific, there a couple of different implementations around the internet to resolve this problem, some of them are listed bellow. I prefer the implementation that I showed here, because i think that is a simple implementation and suits well to the problem.

## **References and Further Reading**

- [Unit Test: DateTime.Now — Stack Overflow](https://stackoverflow.com/questions/2425721/unit-testing-datetime-now)
- [Testing Against The Current Time — ploeh](https://blogs.msdn.microsoft.com/ploeh/2007/05/12/testing-against-the-current-time/)
- [Dealing With Time In Unit Tests — Ayende @ Rahien](https://ayende.com/blog/3408/dealing-with-time-in-tests)

---
layout: post
title: What Makes Good Unit Test? Readability
date: 2018-01-15 10:00:04.000000000 -03:00
type: post
categories: unittest
tags: aaa namingconvention readability xunit
permalink: "/2018/01/15/makes-good-unit-test-readability/"
image: 
  path: /assets/img/blog/What-Makes-Good-Unit-Test_-Reliability-1.png
description: >
  How readability can make good unit tests.
---
The code that we write is more read than written. We write the code once, and it's read many times more. Because of this, we need to write code that the reader could understand. We should care about the unit test's code quality as much as we care for our production code.&nbsp;In this series' last installment I'm going to talk about how we can write more readable unit tests.

## **Naming Convention - Unit Test**

The way in which we name our unit test is important, I would say it's the most important thing when writing unit tests because, it's the first thing that we see in the unit test. To have good unit tests, we have to understand what's being tested, by reading the test name.

So, we have to convey meaning and maintain the same naming pattern throughout the project. The author Roy Osherove, in his book:&nbsp;[The Art of Unit Testing 2nd Edition](https://www.manning.com/books/the-art-of-unit-testing-second-edition), show a naming convention that in my opinion is one of the best, because it shows what's being tested, how's being tested and what the expected result is.&nbsp;

The convention is&nbsp;. The&nbsp;_UnitOfWork_ usually is the name of the method being tested. The&nbsp;_StateUnderTest_ will be the state that's being tested, such as, the parameters that you're using. The&nbsp;_ExpectedBehavior_ is the expected test's result, it can be a value, an exception or the system's state after the test.

To illustrate, take a look at the&nbsp;`SumCalculator` class:

~~~csharp
public class SumCalculator
{
    public int TotalSum { get; set; }
    
    public int Sum(int a, int b)
    {
        if (a < 0 || b < 0)
            return -100;
            
        TotalSum = a + b;
        return TotalSum;
    }
}
~~~

Based in the&nbsp;`Sum()`&nbsp;method, we can create unit test with this names:&nbsp;

- Sum\_BiggerThanZeroNumbers\_ReturnCalculatedValue
- Sum\_NegativeNumberAs1stParameter\_ReturnMinus100
- Sum\_NegativeNumberAs2ndParameter\_ReturnMinus100

[You can check my post about unit test naming conventions](https://www.matheus.ro/2017/09/24/unit-test-naming-convention/), where I talked more in-depth about this topic.

## **Naming Convention - Variables**

How you write your variables names is pretty important, and in your unit tests it should be in the same importance's level as your production code. Giving good variable names, can assure that whoever is reading the code will understand it, and see what the test is trying to prove quickly.

The next example show an example of a poorly written unit test. An almost unreadable unit test, because you can't say what this test is about.

~~~csharp
[Fact]
public void Bad_UniTest_Name()
{
    var sumCalculator =  new SumCalculator();

    var sum = sumCalculator.Sum(-5, 100);

    Assert.Equal(-100, sum);
}
~~~

As we can see, this unit test is hard to understand. We've the method&nbsp;`Sum()`, which is making the sum of two values, and in the assert the expected result is a magic number, "-100", what does this mean?

If we look at the test's name, it's pretty bad written as well. To really know what's happening in the test, we need&nbsp;to look at the implementation, that's not what we want.

How can write a version that we could actually understand what's going on in this test, by changing the variable names?

~~~csharp
[Fact]
public void Bad_UniTest_Name()
{
    var MINUS_VALUE_IS_ENTERED_AS_PARAMETER = -100;
    var sut = new SumCalculator();

    var result = sut.Sum(-5, 100);
            
    Assert.Equal(MINUS_VALUE_IS_ENTERED_AS_PARAMETER, result);
}
~~~

In this version, I create a new variable,&nbsp;`MINUS_VALUE_IS_ENTERED_AS_PARAMETER`, so now we're able to understand what's the magic number means. I changed the&nbsp;`SumCalculator`&nbsp;variable name to&nbsp;`sut`, system under test, because this class is our test subject. Calling your test subject, _sut_, is recommended to maintain a pattern through the project. The last change was the variable&nbsp;`sum`&nbsp;to&nbsp;`result`, this is a minor change but this helps to see where the result of the action that you are testing is.

We can see how this version is better, by simply changing some variable names and creating a new one. By choosing good variable names, we were able to understand what's happening in the test, even though the test's name is poorly written.&nbsp;

## **AAA Pattern**

To improve unit test's readability, it's important that we separate the data creation, actions and assertions in the test. That's why we've a separation pattern called AAA, Arrange, Act and Assert.

The Arrange phase contains all the data setup for the unit test. In the Act phase, we've the action itself, the _sut's_ method call, which will generate the result. The last phase is the Assert phase, that will have all the assertions need for what we are trying to prove.

In the next instance, we can see a unit test organized with the AAA pattern.

~~~csharp
[Fact]
public void Sum_NegativeNumberAs1stParameter_ReturnMinus100()
{
    //Arrange
    var MINUS_VALUE_IS_ENTERED_AS_PARAMETER = -100;
    var sut = new SumCalculator();

    //Act
    var result = sut.Sum(-5, 100);

    //Assert
    Assert.Equal(MINUS_VALUE_IS_ENTERED_AS_PARAMETER, result);
}
~~~

## **Setup and TearDown Methods**

These kind of methods has its use, but I think that they're more harmful than helpful. The Setup and TearDown methods are used to aggregate common code. Setup method is used for code in the arrange phase, data setup, mocks and stub creation. The TearDown method has the goal to clean the variables and states used by more than one test in a class.

In paper, these methods are a good idea. They're supposed to improve the code quality by reducing code duplication in unit tests. But they hurt the code's readability, by hiding some information when we're reading the tests. The Setup method can become big if we try to set up all the data for a test class in only one method, in result, we end up not having all the information on the test itself. To grasp all test's information, we need to go back and forth between the test and the Setup/TearDown methods.

As time passed, I've used less and less these methods. Now, I'm mostly using helper and utility methods, instead of the Setup and TearDown methods. This solution suits the problem well, because we can write helper methods with good names and most of the times don't need to look at the method implementation. So, we still have the code duplication's reduction and also we have&nbsp;more readability in our unit tests.

## **Conclusion**

To wrap up, in this post I showed how can you improve the unit test's readability. First, we talk about naming conventions, in the unit test naming level and in the variable level. I demonstrate how&nbsp;improve the readability by writing better names and maintain a naming convention in project. After that, I showed how to organize the unit tests, using the AAA pattern, as result, our tests have a clear separation between the arrange, action and assertion phases. Finally, I talked about the Setup and TearDown methods, and how they seems like a good idea, but in practices they aren't that useful. It's better to write your own helper/utility methods, because they can serve the purpose and improve readability.

## **References and Further Reading**

- [The Art of Unit Testing, Second Edition – Roy Osherove – Chapter 8](https://www.manning.com/books/the-art-of-unit-testing-second-edition)

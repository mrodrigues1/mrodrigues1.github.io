---
layout: post
title: 'TDD Anti-patterns: The Free Ride / Piggyback'
date: 2018-04-30 09:00:49.000000000 -03:00
type: post
categories: unittest
tags: tdd
permalink: "/2018/04/30/tdd-antipatterns-the-free-ride-piggyback/"
image: 
  path: /assets/img/blog/TDD-Anti-patterns-The-Free-Ride-Piggyback.png
description: >
  Continuing the series of posts about unit tests anti patterns.
---
_A_[_Stack Overflow thread_](https://stackoverflow.com/questions/333682/unit-testing-anti-patterns-catalogue) _inspired this series._

New requirements are constantly emerging in software development. Production code and test code are always changing. When we implement a new requirement, besides the implementation, we need to update or add the unit tests.

If we are changing an existing feature, we have two options to deal with the unit tests. The first is to create new tests to cover new scenarios. And the second and the easy way is to add an extra assertion in an existing test to cover that case.

This anti-pattern happens when we choose to go with the second option.

## **Cause**

The cause for this pattern to happen is when rather than create a new unit test method to test another/distinct feature/functionality **,** a new assertion with his respective action is placed along another assertion in an existing unit test.

## **How To Avoid** 

Let’s take a look at the following example:

~~~csharp
public class DiscountCalculator
{
    public decimal CalculateDiscount(bool firstPurchase, int age)
    {
        if (firstPurchase)
            return 0.1M;
        else if (age > 60)
            return 0.2M;

        return 0;
    }
}
~~~

Here, we have a discount calculator which is responsible to calculate discounts for customers. `CalculateDiscount`&nbsp;method receives a flag indicating if is the first his first time purchasing and the customer age.

There is a unit test created for this method as well:

~~~csharp
[Fact]
public void CalculateDiscount_ExpectedDiscountForFirstTimePurchase()
{
    //Arrange            
    decimal expected = 0.1M;
    decimal expectedResultAge = 0.2M;

    var sut = new DiscountCalculator();

    //Act
    var result = sut.CalculateDiscount(true, 30);
    var resultAge = sut.CalculateDiscount(false, 65);

    //Assert            
    Assert.Equal(expected, result);
    Assert.Equal(expectedResultAge, resultAge);
}
~~~

I see two problems with this test. The name is not telling what the test is doing, since we are testing first purchase and age paths in the same test. And when the test fails, how can we know which assertion failed?

The test is clearly doing two things. It maybe seem like they are two similar test cases but they are exercising different concepts.

To solve this problem, we have to split this test to exercise both cases separately:

~~~csharp
[Fact]
public void CalculateDiscount_FirstPurchaseTrueAsParameter_Expected10PercentDiscount()
{
    //Arrange
    int age = 30;
    bool firstPurchase = true;
    decimal expectedResult = 0.1M;

    var sut = new DiscountCalculator();

    //Act
    var result = sut.CalculateDiscount(firstPurchase, age);

    //Assert
    Assert.Equal(expectedResult, result);
}

[Fact]
public void CalculateDiscount_Age65AsParameter_Expected20PercentDiscount()
{
    //Arrange
    int age = 64;
    bool firstPurchase = false;
    decimal expectedResult = 0.2M;

    var sut = new DiscountCalculator();

    //Act
    var result = sut.CalculateDiscount(firstPurchase, age);

    //Assert
    Assert.Equal(expectedResult, result);
}
~~~

With two tests, we will know the reason the test fail, because it has only one assertion, also the names are much more clear now. By looking at the test name, we can say what the test is doing.

The important thing to remember is to test only one concept per test.

## **Further Reading and References**

- [TDD Antipatterns - Agile in a Flash](http://agileinaflash.blogspot.com/2009/06/tdd-antipatterns.html)

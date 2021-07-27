---
layout: post
title: 'Unit Test Patterns: Pass/Fail Patterns'
date: 2018-04-09 09:00:01.000000000 -03:00
type: post
categories: unittest
tags: xunit
permalink: "/2018/04/09/unit-test-patterns-pass-fail-patterns/"
image: 
  path: /assets/img/blog/Unit-Test-Patterns-PassFail-Patterns.png
description: >
  Unit test patterns series of posts.
---
Pass/Fail Patterns are the most basic unit test patterns, they are our first way to tell if the production code is working as it should.

## **The Simple-Test Pattern**

The Simple-Test pattern has the objective to test only one thing at a time. We have a condition and an expected result.

When a test pass, it tells us that the code work for the given input on the test, just that. Same is valid when we are expecting a failure from the test, it tells only that the test caught the error.

The developer satisfies itself when see all green in the test runner. But, we cannot have 100% sure if it will work under other set of conditions.

Simple-Test pattern test the basic application logic.

## **The Code-Path Pattern**

Code-Path Pattern is the next step to follow from the Simple-Test Pattern.This pattern exercise the control flow, meaning that we test the application paths. We create multiples tests to exercise application paths instead of creating just one for each method.

Testing multiples paths gives more confidence that our code is working under a number of circumstances.

One thing to keep in mind is when you are developing with a test first methodology, like TDD. How can we know the application flow before even writing the code? We don’t. So, we have to continue creating more tests as the code develop.

Using this pattern we can just look at our tests and understand the application behavior. Like a documentation.

## **The Parameter-Range Pattern**

The Parameter-Range pattern are tests that receives parameters. Parameterized tests receive a sets of parameters which are responsible to test a path and then expect a result for each parameter set.

I recommend to create parameters sets for success and for failures in separate tests. Because having both in the same set can hide the test intent. Parameterized test names are not to clear, because we are exercising multiple paths in the same test. Is better keep the context as simples as possible and not mix both success/failure paths. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

Some .net unit test frameworks support parameterized test feature, for instance, xUnit and NUnit.

Take a look at the following code sample using xUnit framework:

~~~csharp
[Theory]
[InlineData(2)]
[InlineData(4)]
[InlineData(6)]
public void IsEven_PositiveEvenNumbersAsParameters_ReturnTrue(int value)
{
    Assert.True(IsEven(value));
}

bool IsEven(int value)
{
    return value % 2 == 0;
}
~~~

This ends the post, hope you enjoyed, I see you in the next one!

## **Further Reading and References**

- [Advanced Unit Test, Part V - Unit Test Patterns](https://www.codeproject.com/Articles/5772/Advanced-Unit-Test-Part-V-Unit-Test-Patterns)
- [Useful design patterns for unit testing/TDD? - Stack Overflow](https://stackoverflow.com/questions/3840125/useful-design-patterns-for-unit-testing-tdd)

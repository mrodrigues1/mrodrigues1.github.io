---
layout: post
title: 'TDD Anti-patterns: The Liar'
date: 2018-01-22 09:30:53.000000000 -03:00
type: post
categories: unittest
tags: tdd
permalink: "/2018/01/22/tdd-anti-patterns-liar/"
image: 
  path: /assets/img/blog/TDD-Anti-patterns_-The-Liar.png
description: >
  Continuing the series of posts about unit tests anti patterns.
---
_This series is inspired by a [Stack Overflow thread](https://stackoverflow.com/questions/333682/unit-testing-anti-patterns-catalogue)._

Imagine have a big test suite with +500 tests. Every unit test's result is green. The continuous integration is working fine, so, there's nothing related to unit tests breaking the build.&nbsp;

We start to make changes in production code, and need to create or update unit tests. When we're reading the unit tests, we found tests that based on the name should do one thing, but the implementation is testing an unrelated thing.

This is an anti-pattern called: The Liar. The Liar is a unit test that runs and doesn't fail, but with a closer look, it isn't testing what's claiming to test. The test could be named after a class/method name, but in reality is testing another class/method. The test can be called _ShouldReturnEmptyForNegativeInputs_, but it actually uses only positive numbers in the test. The Liar provides a false sense of security.

## **The Liar Code Example**

Next, I'm gonna show a code example of the Liar and how we can address this anti-pattern:

~~~csharp
[Fact]
public void ShouldReturnEmptyForNegativeInputs()
{
    //Arrange
    var expectedResult = 200;
    var sut = new Calculator();

    //Act
    var result = sut.Sum(100, 100);

    //Assert
    Assert.Equal(expectedResult, result);
}
~~~

In the example, we have a test named&nbsp;_ShouldReturnEmptyForNegativeInputs_, but instead of using negative values, we have only positive values and the assert is checking the sum's result.

The test is saying one thing in the name and trying to prove another thing in the assertion. One solution in this case is to update the test name to a name that match the implementation. Other possible solution is to change the test's implementation to match the test name.

## **Conclusion**

The Liar is one of the most harmful tdd anti-patterns, it gives a false security's sense, because it hides behind the test's implementation. Thereupon, it's hard find in the code.

My tip to avoid this problem is to when creating/updating unit tests always check if the test's implementation is matching the name.

## **Further Reading and References**

- [TDD Anti-patterns - Agile In Flash](http://agileinaflash.blogspot.com.br/2009/06/tdd-antipatterns.html)
- [5 TDD Antipatterns - MadeTech](https://www.madetech.com/blog/5-tdd-antipatterns)
- [Unit testing Anti-patterns catalogue - StackOverFlow](https://stackoverflow.com/questions/333682/unit-testing-anti-patterns-catalogue)

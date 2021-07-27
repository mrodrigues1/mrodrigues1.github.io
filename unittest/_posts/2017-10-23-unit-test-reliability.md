---
layout: post
title: What Makes Good Unit Test? Reliability
date: 2017-10-23 23:55:17.000000000 -03:00
type: post
categories: unittest
tags: tdd reliability
permalink: "/2017/10/23/unit-test-reliability/"
image: 
  path: /assets/img/blog/What-Makes-Good-Unit-Test_-Reliability.png
description: >
  How reliability can make good unit tests.
---
A unit test verifies if the system under test is working the way it should, but how can we trust the test?

Anyone which codes, can also write unit tests. It's hard in the beginning, but if we keep making it, we can get used to. Write reliable unit tests in other hand, is way more difficult.

We tend to think that unit tests are not production code, I disagree with it, our test code should as good as the production code, if not better.

To be able to write good unit tests, they should have three properties:

- Reliability
- Maintainability
- Readability

We can have many tests as we want,&nbsp;If we can't rely on them, maintain them, or read them, our tests worth nothing.

This is the first post of a series about writing good unit tests, each post will be about one of these three properties, this post is about reliability.

## **Reliability**

When we run our tests, we&nbsp;should be confident about its results. If we see all greens in the test runner, this means that we know why the tests are passing. Having sometimes a test that sometimes passes and other times fails, is a big red flag, indicating that we must take a closer look into this test, to see why this is happening.

A developer wants to run tests and trust their results, when we see a test fails, there are two options, the first is that we found a bug in our code, and the second is that our test is broken and need to be refactored or even removed from the project.

### **See The Test Failing**

This is one of the most important things about unit tests, we have to see them fail. "But wait, aren't you saying before, if a test fails, there's a bug in the system?", yeah, you are right. Let me explain what I mean.

The TDD follow the Red/Green/Refactor cycle. In this case is extremely important to see the test fail. Because if a test doesn't fail, you have a false positive. Seeing test fail and making it pass, means that&nbsp;in fact, you are test something.

The other case is when we're writing tests for existing code. In this case, it's better to follow a proper guide to write characterization tests. It says that we have to first write an assertion that will fail, let the assertion tell the right behavior of system under test and then write the proper assertion to make the test green.

### **Unit Test Don't Need Logic**

The production code has all the logic, our tests don't. When the tests have logic, the possibility of a bug being introduced in a test is high, because the test will have more than one path to follow. This cause the test to fail or pass sometimes, as I said earlier.&nbsp;

Our unit test should be simple as possible and follow the AAA structure, Arrange, Act, and Assert. Arrange for the creation of the system under test and data setup. Act is when exercise the&nbsp;system under test. Assert is the assertion phase, when we verify the result's correctness.

One exception to this is when we need to create a lot of test data, it's acceptable to use a for each. But even in this case, we have better options, like, the [AutoFixture framework](https://github.com/AutoFixture/AutoFixture), which can be used to create dummy test data, or use the builder pattern to create your data, or even write plain objects to avoid a for each.

### **Test Only One Thing At a Time**

Each unit test should test only one thing at a time, this will isolate that one thing and prove whether it works.

Given the method:

~~~csharp
public int Sum(int x, int y)
{
    if (x > 10)
        x = 0;

    if (y < 10)
        y = 10;

    return x + y;
}
~~~

~~~csharp
[Test]
public void Sum_PositiveNumbersAsParameters_ReturnResult()
{
    //Arrange
    var calculator = new Calculator();

    //Assert
    Assert.AreEqual(16, calculator.Sum(1, 15));
    Assert.AreEqual(20, calculator.Sum(10, 5));
    Assert.AreEqual(15, calculator.Sum(20, 15));
}
~~~

This is a bad thing?

I'd say yes, because when one of those four assertions fails, how can we know which assertion fails?

We would have to debug the code to see which one is failing.

Other approach to test with multiple assertions in one unit test is to use the data attribute features from frameworks, like the TestCase from NUnit:

~~~csharp
[TestCase(1, 15, 16)]
[TestCase(10, 5, 20)]
[TestCase(20, 15, 15)]
public void Sum_PositiveNumbersAsParameters_ReturnResult(int x, int y, int expected)
{
    //Arrange
    var calculator = new Calculator();

    //Assert
    Assert.AreEqual(expected, calculator.Sum(x, y));
}
~~~

The xUnit framework has a similar feature using the InlineData Attribute.

Parameterized test is a great feature from these two frameworks, but I don't think that they are worth, because the problem remains, we still have multiple assertions. When a developer sees a test with more than one assertion or parameter, this can encourage him to add more assertions to the test, resulting in a unit tests in which we ended up testing a lot of stuff, and the test's meaning can become hiding inside the test.

### **Put Each Test Where It Belongs**

We should be able to differentiate each type of test and move them to the right place in our solution. This is extremely important because the unit tests should run pretty fast and on the other hand, integration tests normally have some setup to do before run, so&nbsp;because of that, they're not fast tests to run.

Unit test that is slow to run is often forgotten, it's a good practice to move unit tests that are running slow to the integration test project. By doing this, the developers will be more likely to run the unit tests more often and check if anything is broken in the system.

## **Conclusion**

These are the characteristics that I believe that makes our unit tests more reliable, and when a developer runs the tests, he's confident with the results. It's a good mindset to have when creating your unit test and see where you can apply them.

In my next post, I'll talk about maintainability in the unit tests.

## **References And Further Reading**

- [The Art of Unit Testing 2nd Edition - Chapter 8 -&nbsp;Roy Osherove](https://www.manning.com/books/the-art-of-unit-testing-second-edition)
- [Why trust tests?&nbsp;<small>- Mark Seemann</small>](http://blog.ploeh.dk/2013/04/02/why-trust-tests/)
- [Writing Characterization Tests - Tim Ottinger and Jeff Langr](https://agileinaflash.blogspot.com.br/2009/02/writing-characterization-tests.html)
- [A Unit test should test only one&nbsp;thing - Roy Osherove](http://osherove.com/blog/2005/4/3/a-unit-test-should-test-only-one-thing.html)
- [NUnit - TestCase attribute](https://github.com/nunit/docs/wiki/TestCase-Attribute)
- [xUnit - InlineData attribute](https://xunit.github.io/docs/getting-started-desktop.html)

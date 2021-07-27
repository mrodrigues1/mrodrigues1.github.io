---
layout: post
title: 'TDD Anti-patterns: The Test With No Name'
date: 2018-04-02 09:00:19.000000000 -03:00
type: post
categories: unittest
tags: tdd maintainability namingconvention readability
permalink: "/2018/04/02/tdd-anti-patterns-the-test-with-no-name/"
image: 
  path: /assets/img/blog/TDD-Anti-patterns-The-Test-With-No-Name.png
description: >
  Continuing the series of posts about unit tests anti patterns.
---
_A [Stack Overflow thread](https://stackoverflow.com/questions/333682/unit-testing-anti-patterns-catalogue) inspired this series._

When we find a bug, the tendency is to create or update a unit test to reproduce and validate the error. Instead of improving an existing test and without necessity, the developer creates a unit test called _testForBug1234_. This is the same name as the bug in the bug tracker.

At that time it make sense, but what if the test fail in the future?

We may need to search for “Bug: 1234” on the tracker to figure the test’s intent. What if the tracker changes or we can’t get the old tracker information?

Without this information that unit test hard to understand.

Another instance of test with no name happens when we name tests, such as, _TestMethod_, _TestingSomeStuff_, _JohnTest_ and etc. This type of tests hurts readability and maintainability.

## **How to Avoid The Test With No Name**

The solution to not fall into this anti-pattern is to use a unit test naming convention throughout your project.

The one I recommend is from Roy Osherove’s book, The Art Of Unit Test. Which consists of UnitOfWork_StateUnderTest_ExpectedBehavior. It breaks the test’s name in 3 parts, what will be tested (UnitOfWork), the state under test (StateUnderTest) and what the outcome will be (ExpectedBehavior).

To exemplify, imagine a calculator which only do sum operations, receive two numbers and doesn’t accept negative numbers.

What kind of tests could we create?

- Sum_BiggerThanZeroNumbers_ReturnCalculatedValue
- Sum_NegativeNumberAs1stParameter_ThrowException
- Sum_NegativeNumberAs2ndParameter_ThrowException

I already talked about naming conventions in a separate post, the link on the last section.

## **Conclusion**

Keep in mind to have a naming convention when developing unit tests. This will increase your project’s readability and maintainability.

## **Further Reading and References**

- [Unit testing Anti-patterns catalogue - Stack Overflow](https://stackoverflow.com/questions/333682/unit-testing-anti-patterns-catalogue)
- [Unit Test Naming Conventions - Matheus Rodrigues](https://www.matheus.ro/2017/09/24/unit-test-naming-convention/)

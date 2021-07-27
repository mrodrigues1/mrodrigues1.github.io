---
layout: post
title: Unit Test Naming Convention
date: 2017-09-24 01:11:28.000000000 -03:00
categories: unittest
tags: namingconvention
permalink: "/2017/09/24/unit-test-naming-convention/"
image: 
  path: /assets/img/blog/naming_convention.jpg
description: >
  Naming your unit tests in the right way.
---
Naming classes, methods, variables are hard, perhaps one of the most difficult things in programming.

> There are only two hard things in Computer Science: cache invalidation and **naming things**. — PhilKarlton

I’m recently reading the book The Art Of Unit Testing, by Roy Osherove, an excellent book on how to develop sustainable, readable and reliable unit tests. Highly recommended to any developer who wanted to start learning about unit testing and delving deeper into the subject.

In one of the chapters, the author give some tips on how to naming unit tests to make them more readable and improve maintenance by other developers. From this reading, I decided to make this post to talk about naming conventions for unit tests, I’m going to talk about the naming convention from the book and two other approaches.

## **Sample Class**

~~~csharp
public class SumCalculator 
{ 
    public int TotalSum { get; set; } 
    public int Sum(int a, int b) 
    { 
        if (a < 0 || b < 0) 
        {
            throw new ArgumentOutOfRangeException("Sum calculator do not accept less than zero values."); 
        }
        TotalSum = a + b; return TotalSum; 
    } 
}
~~~

##  **UnitOfWork_StateUnderTest_ExpectedBehavior**

This is the strategy from the book.

The **unit of work** can be a method’s name, a class, or set of classes that will be tested. Usually the method’s name of the class that’s under test will be used.

**StateUnderTest** is basically like “Invalid Password”, or “Good Email”, the state being tested. You can describe the parameters that will be sent to the method under test, or the initial state of the unit of work when it’s called, such as, “email already exists”, or “no data found”, or “exception throw on service timeout”.

Finally, the expected result or behavior **(****ExpectedBehavior)** that is based on the conditions under test. It can be either be a value as a result (a real value or an exception) or a system state change as a result ( such as adding a new user to the system, so the system will behave differently at the next login).

### **Examples:**

- Sum_BiggerThanZeroNumbers_ReturnCalculatedValue
- Sum_NegativeNumberAs1stParameter_ThrowException
- Sum_NegativeNumberAs2ndParameter_ThrowException

One of the biggest advantages of this approach is the test’s purpose, what is being tested and the expected result are very clear, you just have to look at the test’s name to know the necessary information about it. And one disadvantage is when the method under test has its name changed, the names of the tests should also be updated to match the adopted convention.

## **Given_Preconditions_When_StateUnderTest_Then_ExpectedBehavior**

This approach is based in the Behavior-Driven Development (BDD) naming convention. The main idea is the break the unit test name into three parts, one is for the **preconditions** , the second is to show the conditions being tested**(StateUnderTest)**, and the third is to say the **expected behavior.**

### **Examples:**

- Given_SumCalculator_When_BiggerThanZeroNumbersAsParameters_Then_ReturnCalculatedValue
- Given_SumCalculator_When_NegativeNumberAs1stParameter_Then_ThrowException
- Given_SumCalculator_When_NegativeNumberAs2ndParameter_Then_ThrowException

If you like or work with BDD, this is the way to go.

## **Should_ExpectedBehavior_When_StateUnderTest**

In this approach the name is divided into two pieces. You describe the **expected behavior** and the conditions that are being tested**(StateUnderTest)**.

### **Examples:**

- Should_ReturnCalculatedValue_When_SumWithBiggerThanZeroNumbersAsParameters
- Should_ThrowException_When_SumWithNegativeNumberAs1stParameter
- Should_ThrowException_When_SumWithNegativeNumberAs2ndParameter

## **Which approach tochoose?**

Regardless of your choice, the important thing is that your unit tests should have descriptive names to pass the understanding of what’s being tested. Another important point is to maintain a naming pattern throughout the project, following a pattern allows your tests to become more readable and easier to maintain.

## **Further Reading**

- [The Art of Unit Testing: With Examples in C#](https://www.amazon.com.br/Art-Unit-Testing-Examples/dp/1617290890)
- [Naming standards for unit tests](http://osherove.com/blog/2005/4/3/naming-standards-for-unit-tests.html)

---
layout: post
title: 'TDD Anti-patterns: The Hidden Dependency'
date: 2018-04-23 09:00:38.000000000 -03:00
type: post
categories: unittest
tags: tdd
permalink: "/2018/04/23/tdd-anti-patterns-the-hidden-dependency/"
image: 
  path: /assets/img/blog/TDD-Anti-patterns-The-Hidden-Dependency.png
description: >
  Continuing the series of posts about unit tests anti patterns.
---
_A_ [_Stack Overflow thread_](https://stackoverflow.com/questions/333682/unit-testing-anti-patterns-catalogue) _inspired this series._

When maintaining unit tests, sometimes, we have to update the setup data, because of a new feature, for example. To do this, we go to the setup method and change the data to fit the new requirement. After updating the data, the new unit test is passing. But just to make sure, we run all class tests and now tests are not all green.

Looking into the code, we find out the reason the tests are not passing anymore. It was because of our changes in the setup data method. So we realize that having a unique setup method for a class it is not a good practice. Every change on this method will affect all tests in a class.

This is what we call the hidden dependency.

## **Causes**

The hidden dependency is a test that requires setup data that are not present on the test itself, as described above. Maybe it need a setup script, a change in a configuration file or have to run after a specific test to pass.

## **How To Avoid**

There are some ways to avoid hidden dependencies.

### **Do Not Use External Configuration**

We have to check if our unit test project need some configuration or script file, such as, a connection string or a template file. After identifying it, we need to isolate this code in their own classes, and inject them via constructor.

~~~csharp
public class HiddenDependencyService
{
    //...

    public HiddenDependencyService(DbContext context)
    {
        //....
    }

    //...
}
~~~

In the example above, the _HiddenDependencyService_ class have a constructor which receives &nbsp;a _DbContext_ as parameter. This allow us to create mock or a fake _DbContext_ implementation and we can send it to _HiddenDependencyService_ in our unit test. Thus, we do not need a real _DbContext_ or a database, as result, there is no need to for configurations outside the test.

If it is not possible to get rid of the configuration file, the test related to the dependency are integration tests.

### **Have Everything Related To The Unit Test Within The Unit Test Scope**

Unit test frameworks usually have the Setup method feature. Where we can set up data for all tests in that class and it runs before each test.

I do not recommend using a set up method in this way, because what end up happening is that the method concentrates the whole data set up. The method can become huge and since it runs before each test it will slow down all tests in that class.

The idea is to create our own setup methods and calling them inside the unit tests. Using this technique, we can have multiple setup methods that contains only the setup for a specific object. Inside each test we can call only the methods related to that scope.

### **Unit Tests Needs To Run Independent Of Order**

Unit tests do not have to depend on a particular order to run. This happens when a test is expecting other unit test to change a global variable. If for some reason the test run in a different order than expected, the unit test fails.

To give a context of when this happens, we have unit tests some a class and we are testing create, read, update and delete operations. There is a global list, that stores the data on the test and all tests use it. The test manipulates the list after each run and not do not reset the list. So, to delete a register from the list, it has to be there. For this to happen, the test which is responsible to add a register into the list has to run first. The test will fail if not run in the correct order.

To avoid this we have to approaches:

- Re-create the test data within each test, so everything is independent. 
- Reset the data after each test with a TeadDown method.

## **Further Reading and References**

- [TDD Antipatterns - Agile in a Flash](http://agileinaflash.blogspot.com/2009/06/tdd-antipatterns.html)

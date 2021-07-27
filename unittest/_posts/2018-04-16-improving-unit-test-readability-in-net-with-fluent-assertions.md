---
layout: post
title: Improving Unit Test Readability In .Net With Fluent Assertions
date: 2018-04-16 09:00:45.000000000 -03:00
type: post
categories: unittest
tags: .net fluentinterface fluentassertions readability
permalink: "/2018/04/16/improving-unit-test-readability-in-net-with-fluent-assertions/"
image: 
  path: /assets/img/blog/Improving-Unit-Test-Readability-In-.Net-With-Fluent-Assertions.png
description: >
  How to improve unit test readability.
---
The assertion in unit tests is the phase where we verify if the test result is what we expect. This phase is straightforward, usually it is just one line. But it is not very readable, because it looks something like this: `Assert.Equal(“ExpectedResult”, “ActualResult”)`.

There is no logical order in expected result and actual result parameter, this always confuses me. Other aspect about readability is that it is not human readable. If you try to read a code like this out loud it is not sound that good.

To solve this problem, frameworks to improve readability in assertions were created, like, [Fluent Assertions](https://fluentassertions.com/) framework for .net. As the name say, this framework use a fluent interface to create readable assertions.

In this post, I’m going to show how Fluent Assertions can improve the unit test readability.

## **Code sample**

Take a look at the following sample:

~~~csharp
public class BankAccount
{
    private double balance;

    private bool frozen = false;

    public BankAccount(double balance)
    {
        this.balance = balance;
    }

    public double Balance
    {
        get { return balance; }
    }

    public void Debit(double amount)
    {
        if (frozen)
            throw new Exception("Account frozen");

        if (amount > balance)
            throw new ArgumentOutOfRangeException("amount");

        if (amount < 0)
            throw new ArgumentOutOfRangeException("amount");

        balance -= amount;
    }

    public void Credit(double amount)
    {
        if (frozen)
            throw new Exception("Account frozen");

        if (amount < 0)
            throw new ArgumentOutOfRangeException("amount");

        balance += amount;
    }

    public void FreezeAccount()
    {
        frozen = true;
    }

    public void UnfreezeAccount()
    {
        frozen = false;
    }
}
~~~

Code sample based on a sample project from [Microsoft docs](https://docs.microsoft.com/en-us/visualstudio/test/sample-project-for-creating-unit-tests). `BankAcount` class have functionalities, for instance, debit, credit, frozen account and unfrozen account.

Now I am going to create unit tests for `BankAcount` class using some of the Fluent Assertions features.

## **Simple Assertion**

We are initializing the account with balance equals 100 and crediting plus 100 in the account. The account should have 200 after the operation.

~~~csharp
[Fact]
public void Credit_PositiveAmountAsParameter_BalanceEquals200()
{
    //Arrange
    var balance = 100;
    var amount = 100;

    var sut = new BankAccount(balance);

    //Act
    sut.Credit(amount);

    //Assert
    sut.Balance.Should().Be(200);
}
~~~

In the assert phase, we are checking the balance from the system under test (sut) with Fluent Assertions. A code like this: `sut.Balance.Should().Be(200);` is much more readable than `Assert.AreEqual(sut.Balance, 200);`.

## **Checking The Object Type**

This test does not make too much sense. I created only to show how to validate the object type.

~~~csharp
[Fact]
public void Credit_PositiveAmountAsParameter_BalanceTypeIsDouble()
{
    //Arrange
    var balance = 100;
    var amount = 100;

    var sut = new BankAccount(balance);

    //Act
    sut.Credit(amount);

    //Assert
    sut.Balance.Should().BeOfType(typeof(double));
}
~~~

## **Checking Exception**

When we send a negative amount to a credit operation, the method should throw an exception.

~~~csharp
[Fact]
public void Credit_NegativeAmountAsParameter_ThrowsArgumentOutOfRangeException()
{
    //Arrange
    var balance = 100;
    var amount = -1;

    var sut = new BankAccount(balance);

    //Act
    Action action = () => sut.Credit(amount);

    //Assert
    action
        .Should()
        .Throw<ArgumentOutOfRangeException>();
}
~~~

## **Checking Exception Message**

Sometimes check if an exception has been throw is not enough. Fluent Assertion have us covered, because in addition to the exception, we can also check which message the exception returned.

~~~csharp
[Fact]
public void Debit_AccountIsFrozen_ThrowsExceptionWithMessage()
{
    //Arrange
    var balance = 100;
    var amount = 1000;

    var sut = new BankAccount(balance);
            
    //Act
    sut.FreezeAccount();
    Action action = () => sut.Debit(amount);

    //Assert
    action
        .Should()
        .Throw<Exception>()
        .WithMessage("Account frozen");
}
~~~

In this test, we are trying to execute a debit operation in a frozen account. The method do not allow that, so it throws an exception. In the assert phase, the exception returns a message and we check to see if the message is right.

## **Checking If The Exception Was Not Throw**

There is a case when want to check if an exception was not throw.

~~~csharp
[Fact]
public void Debit_PositiveAmountAsParameter_NotThrowException()
{
    //Arrange
    var balance = 100;
    var amount = 50;

    var sut = new BankAccount(balance);
            
    //Act            
    Action action = () => sut.Debit(amount);

    //Assert
    action
        .Should()
        .NotThrow<Exception>();
}
~~~

## **Conclusion**

Fluent Assertions improve the assertions readability and make it easy to create them. Using a human like language.

I only scratch the surface in this post, there a lot more possibilities in Fluent Assertions. You can check the [documentation](https://fluentassertions.com/documentation/) and [download the nuget package](https://www.nuget.org/packages/fluentassertions).


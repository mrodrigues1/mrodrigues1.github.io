---
layout: post
title: Refactor a Switch Using OOP Techniques
date: 2017-12-25 09:00:59.000000000 -03:00
type: post
categories: designpatterns
tags: oop refactoring
permalink: "/2017/12/25/refactor-switch-statement-using-oop-techniques/"
image: 
  path: /assets/img/blog/Refactor-a-Switch-Using-OOP-Techniques.png
description: >
  Making cleaner switch statements.
---
> _You can check all the code in [this git repository.](https://github.com/mrodrigues1/Switch-Refactor-Demo)_

Have you ever had a switch like this:

~~~csharp
switch (operation)
{
    case "+":
        Add(number, number2);
        break;
    case "-":
        Subtraction(number, number2);
        break;
    case "*":
        Multiply(number, number2);
        break;
    case "/":
        Divide(number, number2);
        break;
    default:
        break;
}
~~~

And what about this code:

~~~csharp
var calculator = CalculatorFactory.CreateCalculator(operation);
double result = calculator.Calculate(number, number2);
~~~

Both snippets solves the same problem, which one do you think it's better to maintain and read?

## **The Problem**

Switch statements can cause problems. First problem is readability, big code's blocks are hard to read, so if you find yourself scrolling to see all the switch code, maybe it's time for a refactoring.

The other problem is duplication, many times in a project we end up writing the same switch more than once. Each time that we make a change in a switch, we have to update all the other ones, this is pain and prone to bugs on the system.

To make ours lives better instead of using a switch, we can refactor to use proper classes with object-oriented techniques.

## **Example**

For this example I'm gonna use the first switch that showed on the post.

~~~csharp
switch (operation)
{
    case "+":
        Add(number, number2);
        break;
    case "-":
        Subtraction(number, number2);
        break;
    case "*":
        Multiply(number, number2);
        break;
    case "/":
        Divide(number, number2);
        break;
    default:
        break;
}
~~~

This is a code's snippet of a console application, in which, we're implementing a basic calculator. We enter two numbers and the operation.

## **The Solution**

The first thing in the refactoring is to create a factory class.

~~~csharp
public static class CalculatorFactory
{
    private static readonly Dictionary<string, Func<ICalculate>> CalculatorMap = new Dictionary<string, Func<ICalculate>>
    {
        { "+", () => new Add() },
        { "-", () => new Subtract() },
        { "*", () => new Multiply() },
        { "/", () => new Divide() },
    };

    public static ICalculate CreateCalculator(string operation)
    {
        return CalculatorMap[operation]();
    }
}
~~~

This class has a dictionary,&nbsp;_CalculatorMap\<string, Func\<ICalculate\>\>_, in which, the key is representing the operation and the value is a function that will execute the operation.&nbsp;

And we need a method to return the right from function from the dictionary, called&nbsp;_CreateCalculatorFromOperation_. This method will return the calculator instance based on the operation.

Also, I've created an interface, _ICalculate_:

~~~csharp
public interface ICalculate
{
    double Calculate(string number, string number2);
}
~~~

This interface has a method _Calculate_ and&nbsp;all operations will inherit it. Each operation implements this interface, such as:

~~~csharp
public class Add : ICalculate
{
    public double Calculate(string number, string number2)
    {
        return double.Parse(number) + double.Parse(number2);
    }
}
~~~

The _Add_ class implements the _Calculate_ method from the _ICalculate_ interface. All other calculator operations implement _ICalculate&nbsp;_in the same fashion.

Now that we set up all the structure, we can use the calculator:

~~~csharp
var calculator = CalculatorFactory.CreateCalculator(operation);
double result = calculator.Calculate(number, number2);
~~~

The usage is simple, we call the _CreateCalculator_ method from the _CalculatorFactory_ class, passing the operation as the argument, to return the calculator instance. After that, we call the _Calculate_ method and get the result.

## **Conclusion**

In conclusion, in this post I showed one way to refactor a switch using object-oriented techniques, there are ways other to make this refactor, but, I think this is the simplest one.

We address the problems that were point out in the beginning, regarding code's duplication and readability, by isolating the switch's logic in a separated class.

You can check all the code in [this git repository.](https://github.com/mrodrigues1/Switch-Refactor-Demo)

## **Further Reading and References**

- [Refactoring a switch statement - BLARGH!!](http://tmont.com/blargh/2011/11/refactoring-a-switch-statement)
- [Pulling out the Switch: It’s Time for a Whooping - Simple Programmer](https://simpleprogrammer.com/2010/08/17/pulling-out-the-switch-its-time-for-a-whooping/)
- [Switch-Refactor-Demo&nbsp;- GitHub](https://github.com/mrodrigues1/Switch-Refactor-Demo)

---
layout: post
title: 'Clean Code: Avoid Too Many Arguments In Functions'
date: 2018-01-29 10:00:39.000000000 -03:00
type: post
categories: cleancode
tags: solid readability 
permalink: "/2018/01/29/clean-code-avoid-many-arguments-functions/"
image: 
  path: /assets/img/blog/Clean-Code_-Avoid-Too-Many-Arguments-In-Functions.png
description: >
  How many arguments is too many for a function?
---
We can agree that a function with zero arguments is a good function, we don't have to worry too much about it. The perfect world will be if all functions have zero arguments, but we know that's not possible. Sometimes we'll end up writing function containing six or more arguments, like this function:

~~~csharp
public void DoStuff(string email, string userName, string country, string city, string street, string complement, string phone)
{
    //.....
}
~~~

In the function above, we have seven arguments in total, and apparently they don't have a logical order. We could try make sense of this arguments, but we can only guess by just looking at the arguments.

Uncle Bob (Robert C. Martin), in the Clean Code book, explains that an ideal argument's number for a function is zero (niladic function). Followed by one (monadic function) and closely by two arguments (dyadic function). Functions with three arguments (triadic function) should be avoided if possible. More than three arguments (polyadic function) are only for very specific cases and then shouldn't be used anyway.

In my opinion this a great guideline to follow to write better code, but they're just guidelines, not strict rules that you have to follow no matter what.

## **When Is Too Many?**

As Uncle Bob said, three is the maximum arguments acceptable. Although I agree with his affirmation, in my opinion is idealistic. Our  functions should have the minimum number of arguments possible, if it have less than four argument, nice. Put a limit on the argument's number is not the ideal, we should strive to keep them as minimal as we can.

One of the problems that could emerge in our application if we have too many arguments is that the function is doing too much things, not respecting the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle). Maybe there are some parameters set that could be related and turned into its own object, using the [Parameter Object](https://refactoring.guru/introduce-parameter-object) pattern (more on this pattern later).

## **How To Reduce The Number Of Arguments**

It's difficult to understand what we're looking for without searching for more information, such as, reading through the function's implementation.

There are two techniques that can be used to reduce a functions' arguments. One of them is to refactor the function, making it smaller, consequently, reducing the arguments' number. The [Extract Method](https://refactoring.guru/extract-method) technique can be use to achieve this goal.

Another technique is the pattern mentioned earlier, the Parameter Object pattern. We can aggregate arguments that are within same context, and then create a plain object containing those arguments.

In next example, I'm going to show how to use the Parameter Object to reduce arguments' number using the sample function showed earlier:

~~~csharp
public void DoStuff(string email, string userName, string country, string city, string street, string complement, string phone)
{
    //.....
}
~~~

~~~csharp
public void DoStuff(User user, Address address)
{
    //.....
}
~~~

We're able to reduce the argument's number to only two. Analyzing the context in which the arguments can be related, the User and Address objects were created.

## **Advantages Of Having Less Arguments**

Reducing the arguments' number in functions bring improvements in the code.

- It makes the code more readable, because probably the functions are smaller and following the single responsibility principle. 
- Makes the code easier to test. By making the function smaller, test problems individually is pretty simple. We can test the paths in the main function and have a collection of smaller tests for each individual function.

## **References and Further Reading**

- [Clean Code: Chapter 3 - Functions - Robert C. Martin](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
- [Too Many Parameters - Wiki C2](http://wiki.c2.com/?TooManyParameters)
- [Single Responsibility Principle - Wikipedia](https://en.wikipedia.org/wiki/Single_responsibility_principle)
- [Extract Method - Refactoring Guru](https://refactoring.guru/extract-method)
- [Parameter Object - Refactoring Guru](https://refactoring.guru/introduce-parameter-object)

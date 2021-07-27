---
layout: post
title: 'Design Patterns and Practices in .Net: Option Functional Type in C#'
date: 2017-09-26 21:29:37.000000000 -03:00
type: post
categories: designpatterns
tags: .net
permalink: "/2017/09/26/design-patterns-and-practices-in-net-option-functional-type-in-csharp/"
description: >
  Dealing with nulls in the code using the optional type.
---
Functional programming uses the option type widely, in languages such as Haskell, F#, Scala and a lot more. In this language it’s a convention to return a value when the function fails. The following example is a function written in F#, to show how the option type works:

~~~haskell
(* This function uses pattern matching to deconstruct Options *)
let compute = function
  | None   -> "No value"
  | Some x -> sprintf "The value is: %d" x

printfn "%s" (compute <| Some 42)(* The value is: 42 *)
printfn "%s" (compute None)      (* No value         *)
~~~

We have no equivalent of the option type in the C# language , but we can try to emulate it.

In my last [post](https://www.matheus.ro/2017/09/25/design-patterns-practices-net-null-object-special-case-object/), I discussed two design patterns, Null Object and the Special Case Object. The goal of these patterns are to return a special object instead of return null. If you didn’t read that post, [go check it](https://www.matheus.ro/2017/09/25/design-patterns-practices-net-null-object-special-case-object/) and come back to this post.

The idea in this post is to show the implementation of the option type in C#.

## **Problem**

The problem is similar to my last post, in fact it’s the continuation of that same problem.

~~~csharp
public IProduct GetProductById(int productId)
{
    Product product = _productRepository.GetProductById(productId);

    if(product == null)
        return new ProductNotFound();

    return product;
}
~~~

Here, we have a method that returns a product based on the id, if the database not founds the product, it returns a special object called `ProductNotFound`.

Both Product and `ProductNotFound` inherits from the same interface named `IProduct`.

This is not a complex problem, I’m using it to illustrate my point.

## **Implementing the Option Type**

The implementation of the option type is pretty straightforward, but before that, I want to say that the implementation that I’m going to show is not mine, but this is one of the best and simpler implementations that I saw for C#.

First time a saw this implementation was in a [pluralsight](https://app.pluralsight.com/) course, [Tactical Design Patterns in .NET: Control Flow](https://app.pluralsight.com/library/courses/tactical-design-patterns-dot-net-control-flow/table-of-contents) by [Zoran Horvat](https://twitter.com/zoranh75), a great course about writing simpler code by applying design patterns and coding practices that affects the control flow of the application. If you have a pluralsight subscription go check the course, and other courses by the great author [Zoran Horvat](https://twitter.com/zoranh75).

So, here’s is the implementation:

~~~csharp
public class Option<T> : IEnumerable<T>
{
    private readonly T[] _data;

    private Option(T[] data)
    {
        _data = data;
    }

    public static Option<T> Create(T element)
    {
        return new Option<T>(new T[] { element });
    }

    public static Option<T> CreateEmpty()
    {
        return new Option<T>(new T[0]);
    }

    public IEnumerator<T> GetEnumerator()
    {
        return ((IEnumerable<T>) _data).GetEnumerator();
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}
~~~

The idea behind this implementation is to have an object that can hold one or none object. A list is almost what we want, list can have multiple elements or zero elements. So, the `Option<T>` implements the `IEnumerable<T>` interface, by doing this we can work with this option type as if we are working with a list (more on that later).

The `Option<T>` is a genetic class and T will be the object that we want to be our option. We have a property `_data`, which is an array that will hold the object.

Create the object with one element or zero element is the goal, to allow it, this class have a private constructor, and two static methods responsible for the object creation. The `CreateEmpty()` method, which creates a new option with an empty object, and the `Create(T element)` method, that receives a T object via parameter and create the option with one object.

To finish this section, we have two more methods in this class, the `GetEnumerator()` methods. Our class inherits these methods from the `IEnumerable` interface and implement both.

## **Using the option type**

Now we can use the option type to solve our problem.

First thing to do is to change the `ProductRepository` method `GetProductById` return’s type to `Option<Product>`:

~~~csharp
public class ProductRepository : IProductRepository
{
    //...
    //..Other Details
    //...

    public Option<Product> GetProductById(int productId)
    {
        var product = _data.FirstOrDefault(x => x.Id == productId);

        if (product == null)
            return Option<Product>.CreateEmpty();

        return Option<Product>.Create(product);
    }
}
~~~

As you can see, if the database not founds the product, we use the `CreateEmpty()` method to create an empty instance of the `Option<Product>` and if the product exists, we use the `Create()` method to create an instance of the `Option<Product>` with the product found. Also, we change the method’s return to `Option<Product>`.

Back in the main method, we need to update it to use of the option type. We make minor changes to the method:

~~~csharp
public IProduct GetProductById(int productId)
{
    Option<Product> product = _productRepository.GetProductById(productId);

    if(product.Any() == false)
        return new ProductNotFound();

    return product.Single();
}
~~~

The difference is subtle, we change the return type from the repository method to `OptionType<Product>`. The verification to see if the product is null, is now verifying if the option is empty, and return the `ProductNotFound` object as usual. In the return the only difference is that we have to explicit say that we want to return the `Product` using `.Single()`, which return the only element in the option type.

Now you may ask: How is this better than returning null?

I reply with: This is the same thing as returning null.

In this code, there no real benefit of using the Option type, because we still have to verify if the option has a value or is empty. Here’s the trick, Linq, with Linq we could make this code in “one” line, using the option type and our special case object.

Linq is a powerful C# library which extends the capability of list object, every object in C# that inherits from IEnumerable can use Linq.

The final version of the `GetProductById` method:

~~~csharp
public IProduct GetProductById(int productId)
{
    return
        _productRepository
        .GetProductById(productId)
        .DefaultIfEmpty<IProduct>(new ProductNotFound())
        .Single();
}
~~~

See how simple the code become?

In one line of code we are able to use the option type and our special case object, and is much easier to understand than the old version.

What’s happening here is that we search the product as usual, and using the `DefaultIfEmpty` method from Linq, we are able to say that if the product does not exist in the database, we return a new instance of the `ProductNotFound` object. Finally, `Single` method return the product instance.

## **Conclusion**

To wrap it up, in this post I show you an implementation of the functional type in C#. There’re other implementations of the option type around the internet but this implementation is good because of its simplicity. Inherit from the IEnumerable interface address the problem of nulls, because a list has an element or is empty. Having two methods, one to create an empty object and other to create the option object with one element, allow us to never return a null reference.

Combining the option type with Linq made our code clean and easy to understand. Linq can provide a lot of functionality for the option type, I only scratch the surface in this post.

## **References and readings**

- [Tactical Design Patterns in .NET: Control Flow: Option Functional Type](https://app.pluralsight.com/player?course=tactical-design-patterns-dot-net-control-flow&author=zoran-horvat&name=tactical-design-patterns-dot-net-control-flow-m6&clip=0&mode=live)
- [StackOverflow : Optional return in C#.Net](https://stackoverflow.com/questions/16199227/optional-return-in-c-net)
- [Codinghelmet: How to Reduce Cyclomatic Complexity: Option Functional Type](http://codinghelmet.com/?path=howto/reduce-cyclomatic-complexity-option-functional-type)
- [Option type — Wikipedia](https://en.wikipedia.org/wiki/Option_type#F.23)

---
layout: post
title: 'Design Patterns and Practices in .Net: Null Object and Special Case Object'
date: 2017-09-25 20:41:02.000000000 -03:00
categories: designpatterns
tags: .net
permalink: "/2017/09/25/design-patterns-practices-net-null-object-special-case-object/"
description: >
  A better way to deal with nulls in the code.
---
In programming null is an awkward thing to deal with, when you return a variable with null value, you have to remember to make null checks all around your code.

The null object and the special case object are patterns that try to reduce this type of boiler plate code.

A null object is an object that doesn’t have a referenced value, is an object with a default behavior. A behavior which represents a missing object.

Special case object is very similar to the null object, but it’s an improvement over the null object, because instead of creating a generic “missing object”, you are creating a more specialized object that can suit better your needs.

The goal of these two pattern is to return a null or a special case version of an object, instead of return a null reference.

## **Problem**

Many times we caught ourselves making this type of check:

~~~csharp
public Product GetProductById(int productId)
{
    Product product = _productRepository.GetProductById(productId);
    if(product == null)
        throw new Exception("Product Not Found");
    return product;
}
~~~

## **How To Implement The Null Object Pattern**

In the code above, the repository method _GetProductById_ returns a _Product_ instance, if the product is not in the database, it returns null. When a product is null, we throw an exception, saying that the product was not found.

The idea here is to create a “Null” version of the _Product_ class and return this object instead of an exception.

Here we have the _Product_ class implementation:

~~~csharp
public class Product
{
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string GetProductInformation()
    {
        return $"Product name:{Name}{Environment.NewLine} Price: {Price}.";
    }
}
~~~

To make it work with the Null Object pattern, we need to make an interface of the _Product _class, them both _Product _and _NullProduct _can implement it.

~~~csharp
public interface IProduct
{
    string Name { get; set; }
    decimal Price { get; set; }
    string GetProductInformation();
}
~~~

The _NullProduct_ class:

~~~csharp
public class NullProduct : IProduct
{
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public string GetProductInformation()
    {
        return string.Empty;
    }
}
~~~

As you can see, this class has no behavior, it represents an empty product object.

Here’s the implementation of the _GetProductById_ method using _NullProduct_:

~~~csharp
public IProduct GetProductById(int productId)
{
    Product product = _productRepository.GetProductById(productId);
    if(product == null)
        return new NullProduct();
    return product;
}
~~~

Instead of throwing an exception, we return _NullProduct_ instead.

## **How To Implement The Special Case Object Pattern**

The special case object is pretty like the null object, the only difference is that it’s a more specialized object. In case of the _GetProductById_ method, we could return a _ProductNotFound_ object instead of the _NullProduct_ object.

~~~csharp
public class ProductNotFound : IProduct
{
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public string GetProductInformation()
    {
        return "Product not found.";
    }
}
~~~

What differ ProductNotFound and _NullProduct_ is the_ GetProductInformation_ method return, which is a message saying that the product does not exists in the database.

Now, we can change _GetProductById_ method to return the _ProductNotFound_ object when a product is null.

~~~csharp
public IProduct GetProductById(int productId)
{
    Product product = _productRepository.GetProductById(productId);
    if(product == null)
        return new ProductNotFound();
    return product;
}
~~~

The consumer of this method doesn’t have to change at all when you return a special case object, the difference is the _GetProductInformation_ method which will tell the user that the product is missing from the database.

## **Pros**

- Result in a cleaner code, less boilerplate code, by reducing null checks.
- Usually result in less catastrophic errors, such as, by returning a special case object instead of an exception.

## **Cons**

- Normally a null check solves the problem, you just have to remember where to put it.
- Sometimes is better to return null or throw an exception, for example, in a critical failure.

## **Conclusion**

To close things up, both Null Object Pattern and Special Case Object Pattern are useful patterns when your code has a lot of null checks, throw exceptions, you could change this by returning a special case object.

You have to think of how “loud” you want your code to be. What I mean by that is if you want your application to expose errors, return null or you if you want a smoother application flow return a special case object instead.

There are cases when the object that you are working doesn’t have a sane default state, so you have to rely on null. For me these are the main points to think when I want to implement both patterns.


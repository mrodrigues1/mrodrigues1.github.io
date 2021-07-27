---
layout: post
title: What are Stubs?
date: 2017-09-25 00:51:18.000000000 -03:00
type: post
categories: unittest
tags: stub
permalink: "/2017/09/25/what-are-stubs/"
description: >
  Do you know are stubs are?
---
The definition of stubs can cause confusion sometimes. There are many definitions that you can find in a lot of places, stubs and mock definitions can be pretty similar sometimes, and have a good understanding of both can make the difference when writing highly maintainable unit tests.

Reading the book The Art Of Unit Testing, in the chapter about stubs, the author Roy Osherove gives a great definition of stubs:

> A stub is a controllable replacement for an existing dependency (or collaborator) in the system. By using a stub, you can test your code without dealing with the dependency directly. — Roy Osherove.

A unit test should exercise the smallest possible unit of code, if the code have external dependencies, you should control those dependencies so so that the code can be isolated.

So, how can I use stubs in my unit tests? You may ask. I’ll show you an example of how can you do this.

## **Code Sample**

The _Stock_ class looks up the stock value of a company on _Nasdaq_ using a web service:

~~~csharp
public class Stock
{
    public int GetStockPrice(string company)
    {
        NasdaqWebService nasdaqWebService = new NasdaqWebService();

        return nasdaqWebService.GetStocksPrice(company);
    }
}
~~~

How can we test this method? Here’s an example:

~~~csharp
[TestMethod]
public void GetStocks_ValidCompanyNameAsParameter_ReturnStockPrice()
{
    //Arrange
    Stock sut = new Stock();
    string company = "microsoft";
    int expected = 100;

    //Act
    int result = sut.GetStockPrice(company);

    //Assert
    Assert.AreEqual(expected, result);
}
~~~

In the test we define that the method _GetStockPrice_ will look for stocks of a certain company, the return is expected to be 100, the value of a stock is very hard to predict, floats during the day, the test will work only a small portion of times because we don’t have the guarantee of the service return.

The _Stock_ class is dependent on the _NasdaqWebService_ service, the first thing that we have to do is to decouple the service from the _Stock_ class. To do this, we need to create an interface for the service, and pass a reference from it in the _Stock_ class constructor.

~~~csharp
public interface INasdaqWebService
{
    int GetStocksPrice(string company);
}
~~~

~~~csharp
public class Stock
{
    private INasdaqWebService _nasdaqWebService;

    public Stock(INasdaqWebService nasdaqWebService)
    {
        _nasdaqWebService = nasdaqWebService;
    }

    public int GetStockPrice(string company)
    {            
        return _nasdaqWebService.GetStocksPrice(company);
    }
}
~~~

The unit test updated:

~~~csharp
[TestMethod]
public void GetStocks_ValidCompanyNameAsParameter_ReturnStockPrice()
{
    //Arrange
    var nasdaqWebService = new NasdaqWebService();
    Stock sut = new Stock(nasdaqWebService);

    string company = "microsoft";
    int expected = 100;

    //Act
    int result = sut.GetStockPrice(company);

    //Assert
    Assert.AreEqual(expected, result);
}
~~~

Here comes the stub, decoupling the service allow us to send an implementation of _INasdaqWebService’s_ interface through _Stock_ class constructor, it means that we can send any implementation of the interface via the constructor. After that, we need to create a fake implementation of the interface so that it can be used in our test.

Doing that allow us to manipulate the return of _GetStockPrice_ method.

~~~csharp
public class NasdaqWebServiceStub : INasdaqWebService
{
    public int PriceToReturn;

    public int GetStocksPrice(string company)
    {
        return PriceToReturn;
    }
}
~~~

As you can see, the stub implements the _INasdaqInterface_ interface and this implementation has a property called _PriceToReturn_ to control the return value.

~~~csharp
[TestMethod]
public void GetStocks_ValidCompanyNameAsParameter_ReturnStockPrice()
{
    //Arrange            
    var nasdaqStub = new NasdaqWebServiceStub();
    nasdaqStub.PriceToReturn = 100;
    Stock sut = new Stock(nasdaqStub);

    string company = "microsoft";

    //Act
    int result = sut.GetStockPrice(company);

    //Assert
    Assert.AreEqual(100, result);
}
~~~

Finally, we have the final version of the unit test, in it we create a new instance of the stub, we define the return value and send the stub via a parameter in the constructor of the _Stock_ class. When we make a call to _GetStockPrice_ method, the test will call the stub, and then return the expected value.

The main idea of a stub is to replace one or more dependencies of the functionality under test, so that it isolates the code and allows a control over the test results.

## **References and Further&nbsp;Readin**** g**

- [The Art Of Unit Testing 2nd Edition](https://www.manning.com/books/the-art-of-unit-testing-second-edition)
- [Test stub — Wikipedia](https://en.wikipedia.org/wiki/Test_stub)
- [Mocks Aren’t Stubs — Martin Fowler](https://martinfowler.com/articles/mocksArentStubs.html)

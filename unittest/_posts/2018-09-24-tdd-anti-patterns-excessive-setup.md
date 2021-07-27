---
layout: post
title: 'TDD Anti-patterns: Excessive Setup'
date: 2018-09-24 09:00:47.000000000 -03:00
type: post
categories: unittest
tags: aaa fakeiteasy fluentbuilder tdd xunit
image: 
  path: /assets/img/blog/TDD-Anti-patterns-Excessive-Setup.jpg
permalink: "/2018/09/24/tdd-anti-patterns-excessive-setup/"
description: >
  There is time that we get a headache by just thinking of creating a new unit test. 
  For that particular class which we have to do so much setup to do an assertion.
---
_A_ [_Stack Overflow thread_](https://stackoverflow.com/questions/333682/unit-testing-anti-patterns-catalogue) _inspired this series._

There is time that we get a headache by just thinking of creating a new unit test. For that particular class which we have to do so much setup to do an assertion.

Imagine that [god object](https://en.wikipedia.org/wiki/God_object) with ten dependencies in the constructor, for example. To being able to create a unit test for it, we need to mock all those dependencies. Even if the test is ordinary, the setup would be massive.

Eventually we'll fall into the trap of the excessive setup.

## **Cause**

When a test needs a reasonable number of lines in the data setup phase(arrange), even a hundred lines or so. Usually with several objects that makes difficult to understand what the test is about due to the noise off that huge setup.

Could happen when a class is bad coupled or tight coupled. A class seems to depend on "all things in the world" to be instantiated. Meaning that it has too many responsibilities and depends on many objects.

Not properly&nbsp;used mocking can be another cause. Trying to mock everything, from objects to methods even tough doesn't make any sense in the unit test scope.

## **How to Avoid**

For code samples, I'm using [xUnit.net](https://xunit.github.io/) as unit test framework and [FakeItEasy](https://github.com/FakeItEasy/FakeItEasy) as mocking framework.

Let me show a hypothetical class that may contain too many responsibilities:

~~~csharp
public class ECommerceService : IECommerceService
{
    private readonly IUserRepository _userRepository;
    private readonly IProductRepository _productRepository;
    private readonly IProductCategoryRepository _productCategoryRepository;
    private readonly ICartRepository _cartRepository;
    private readonly IShippingMethodRepository _shippingMethodRepository;
    private readonly IMailRepository _mailRepository;
    private readonly IPromotionCodeRepository _promotionCodeRepository;
    private readonly IECommerceRepository _eCommerceRepository;

    public ECommerceService(
        IUserRepository userRepository,
        IProductRepository productRepository,
        IProductCategoryRepository productCategoryRepository,
        ICartRepository cartRepository,
        IShippingMethodRepository shippingMethodRepository,
        IMailRepository mailRepository,
        IPromotionCodeRepository promotionCodeRepository,
        IECommerceRepository eCommerceRepository)
    {
        _userRepository = userRepository;
        _productRepository = productRepository;
        _productCategoryRepository = productCategoryRepository;
        _cartRepository = cartRepository;
        _shippingMethodRepository = shippingMethodRepository;
        _mailRepository = mailRepository;
        _promotionCodeRepository = promotionCodeRepository;
        _eCommerceRepository = eCommerceRepository;
    }
}
~~~

Looking at `ECommerceService`&nbsp;class constructor, we can ensure a lot is going on in the class.&nbsp;

Without thinking in any method yet, how difficult would it be to instantiate this class in a unit test manner?

```csharp
[Fact]
public void ExampleTest()
{
    //Arrange
    var userRepositoryFake = A.Fake<IUserRepository>();
    var productRepositoryFake = A.Fake<IProductRepository>();
    var productCategoryRepositoryFake = A.Fake<IProductCategoryRepository>();
    var cartRepositoryFake = A.Fake<ICartRepository>();
    var shippingMethodRepositoryFake = A.Fake<IShippingMethodRepository>();
    var mailRepositoryFake = A.Fake<IMailRepository>();
    var promotionCodeRepositoryFake = A.Fake<IPromotionCodeRepository>();
    var eCommerceRepositoryFake = A.Fake<IECommerceRepository>();

    var sut = new ECommerceService(
        userRepositoryFake,
        productRepositoryFake,
        productCategoryRepositoryFake,
        cartRepositoryFake,
        shippingMethodRepositoryFake,
        mailRepositoryFake,
        promotionCodeRepositoryFake,
        eCommerceRepositoryFake);

    //Act

    //Assert
}
```

It may not look too big, but this code only creates&nbsp;`ECommerceService`&nbsp;class. We'll need to put at least some behavior on the fakes to do a proper unit test.

`ECommerceService`&nbsp;has a method to list all products from a category:

```csharp
public List<Product> ProductsByCategory(Guid userIdentity, int productCategoryId)
{
    var userPreferences = _userRepository.UserPreferences(userIdentity);
    var productCategory = _productCategoryRepository.ProductCategory(productCategoryId);

    return _productRepository.Products(userPreferences, productCategory);
}
```

`ProductsByCategory()` method looks for the user preferences if it's a recurrent user and get the product category from `_productCategoryRepository`. Them it get all products that match user preferences and category.

### **Test With Excessive Setup**

Now, let's see a unit test for `ProductsByCategory()` method:

```csharp
[Fact]
public void ProductsByCategory_UserPrerencesAndProcutCategoryAsParameters_ReturnProductList()
{
    //Arrange
    var userRepositoryFake = A.Fake<IUserRepository>();
    var productRepositoryFake = A.Fake<IProductRepository>();
    var productCategoryRepositoryFake = A.Fake<IProductCategoryRepository>();
    var cartRepositoryFake = A.Fake<ICartRepository>();
    var shippingMethodRepositoryFake = A.Fake<IShippingMethodRepository>();
    var mailRepositoryFake = A.Fake<IMailRepository>();
    var promotionCodeRepositoryFake = A.Fake<IPromotionCodeRepository>();
    var eCommerceRepositoryFake = A.Fake<IECommerceRepository>();

    var userPreferences = new List<UserPreference>
    {
        new UserPreference { Id = 1, Preference = "Order By Price Asc" }
    };

    A.CallTo(() => userRepositoryFake.UserPreferences(A<Guid>.Ignored))
        .Returns(userPreferences);

    var productCategory = new ProductCategory
    {
        Id = 1,
        Category = "Game"
    };

    A.CallTo(() => productCategoryRepositoryFake.ProductCategory(A<int>.Ignored))
        .Returns(productCategory);

    var products = new List<Product>
    {
        new Product { Id = 1, Name = "Rocket League", CategoryId = 1 },
        new Product { Id = 2, Name = "Ragnarok", CategoryId = 1 },
        new Product { Id = 3, Name = "Yazuka 0", CategoryId = 1 }
    };

    A.CallTo(
            () => productRepositoryFake.Products(
                A<List<UserPreference>>.Ignored,
                A<ProductCategory>.Ignored))
        .Returns(products);

    var sut = new ECommerceService(
        userRepositoryFake,
        productRepositoryFake,
        productCategoryRepositoryFake,
        cartRepositoryFake,
        shippingMethodRepositoryFake,
        mailRepositoryFake,
        promotionCodeRepositoryFake,
        eCommerceRepositoryFake);

    var userIdentity = new Guid();
    var productCategoryId = 1;

    //Act
    var result = sut.ProductsByCategory(userIdentity, productCategoryId);

    //Assert
    Assert.Equal(result, products);
}
```

The arrange phase has grown a little.&nbsp;`ProductsByCategory()`&nbsp;have three dependencies in its logic, and we need to address this in the test.&nbsp;

We mock all those dependencies to return their respective objects after each call. Also, created two variables that are the parameters of the&nbsp;`ProductsByCategory()`&nbsp;method.

After all this setup, we follow with the test as usual. Calling&nbsp;`ProductsByCategory()`&nbsp;and checking if the result is the same as the products created in the arrange.

### **Helper Methods**

One thing that we can do to reduce the size of the arrange phase is to separate the mock and sut(system under test) creation. Both occupy a lot of space and don't add value to the test.

The following code represents an idea of how we could do this refactor:

```csharp
[Fact]
public void ProductsByCategory_UserPrerencesAndProcutCategoryAsParameters_ReturnProductList()
{
    //Arrange
    CreateFakes();

    var userPreferences = new List<UserPreference>
    {
        new UserPreference { Id = 1, Preference = "Order By Price Asc" }
    };

    A.CallTo(() => _userRepositoryFake.UserPreferences(A<Guid>.Ignored))
        .Returns(userPreferences);

    var productCategory = new ProductCategory
    {
        Id = 1,
        Category = "Game"
    };

    A.CallTo(() => _productCategoryRepositoryFake.ProductCategory(A<int>.Ignored))
        .Returns(productCategory);

    var products = new List<Product>
    {
        new Product { Id = 1, Name = "Rocket League", CategoryId = 1 },
        new Product { Id = 2, Name = "Ragnarok", CategoryId = 1 },
        new Product { Id = 3, Name = "Yazuka 0", CategoryId = 1 }
    };

    A.CallTo(
            () => _productRepositoryFake.Products(
                A<List<UserPreference>>.Ignored,
                A<ProductCategory>.Ignored))
        .Returns(products);

    ECommerceService sut = CreateSUT();

    var userIdentity = new Guid();
    var productCategoryId = 1;

    //Act
    var result = sut.ProductsByCategory(userIdentity, productCategoryId);

    //Assert
    Assert.Equal(result, products);
}
```

There are two new helper methods,&nbsp;`CreateFakes()` and&nbsp;`CreateSUT()`. All those variables to store fakes are now private variables of the whole test class.&nbsp;`CreateFakes()`&nbsp;is responsible for instantiate `ECommerceService` dependencies without any behavior.

```csharp
private void CreateFakes()
{
    _userRepositoryFake = A.Fake<IUserRepository>();
    _productRepositoryFake = A.Fake<IProductRepository>();
    _productCategoryRepositoryFake = A.Fake<IProductCategoryRepository>();
    _cartRepositoryFake = A.Fake<ICartRepository>();
    _shippingMethodRepositoryFake = A.Fake<IShippingMethodRepository>();
    _mailRepositoryFake = A.Fake<IMailRepository>();
    _promotionCodeRepositoryFake = A.Fake<IPromotionCodeRepository>();
    _eCommerceRepositoryFake = A.Fake<IECommerceRepository>();
}
```

&nbsp;`CreateSUT()`&nbsp;method creates&nbsp;`ECommerceService`&nbsp;instance using the private variable in the class.

With this refactor we were able to thin the unit test and reduce code duplication since all unit test from this class can use both helper methods.

### **Extension Methods To Setup Mocks**

The mocks setup are still there. One practice that I like to follow to make mocking cleaner is to create extension methods for the classes to setup mocks.

In this test we are mocking 3 methods from 3 different classes. We are going to create one class for each repository and put all related method setup in its own class.

```csharp
public static class MockUserRepository
{
    public static IUserRepository SetupUserPreferences(
        this IUserRepository repository,
        List<UserPreference> returnValue)
    {
        A.CallTo(() => repository.UserPreferences(A<Guid>.Ignored))
            .Returns(returnValue);

        return repository;
    }
}

public static class MockProductCategoryRepository
{
    public static IProductCategoryRepository SetupProductCategory(
        this IProductCategoryRepository repository,
        ProductCategory returnValue)
    {
        A.CallTo(() => repository.ProductCategory(A<int>.Ignored))
            .Returns(returnValue);

        return repository;
    }
}

public static class MockProductRepository
{
    public static IProductRepository SetupProducts(
        this IProductRepository repository,
        List<Product> returnValue)
    {
        A.CallTo(
                () => repository.Products(
                    A<List<UserPreference>>.Ignored,
                    A<ProductCategory>.Ignored))
            .Returns(returnValue);

        return repository;
    }
}
```

All code to mock those repository methods now belong to these extension methods. With extensions methods the syntax on the test is simpler and easy to read as you can see in the updated test:

```csharp
[Fact]
public void ProductsByCategory_UserPrerencesAndProcutCategoryAsParameters_ReturnProductList()
{
    //Arrange
    CreateFakes();

    var userPreferences = new List<UserPreference> //...
    
    _userRepositoryFake.SetupUserPreferences(userPreferences);

    var productCategory = new ProductCategory //...

    _productCategoryRepositoryFake.SetupProductCategory(productCategory);

    var products = new List<Product> //...
    
    _productRepositoryFake.SetupProducts(products);

    ECommerceService sut = CreateSUT();

    var userIdentity = new Guid();
    var productCategoryId = 1;

    //Act
    var result = sut.ProductsByCategory(userIdentity, productCategoryId);

    //Assert
    Assert.Equal(result, products);
}
```

Although arrange size hasn't decrease too much in this case, but imagine an unit test with way more methods to mock, will make a difference.

One last thing that we could address is data creation.

### **Fluent Builder Pattern**

We can use Fluent Builder Pattern to deal with data setup. It's a pattern responsible to help object creation, taking away most of the verbosity and letting the code easier to write and read. I already talked about this pattern in another post, so I'm not going into much details, [you check the post here](https://www.matheus.ro/2018/01/08/design-patterns-practices-net-fluent-builder-test-pattern/).

Now, we can check how test will look after implementing Fluent Builder Pattern:

```csharp
[Fact]
public void ProductsByCategory_UserPrerencesAndProcutCategoryAsParameters_ReturnProductList()
{
    //Arrange
    CreateFakes();            

    _userRepositoryFake.SetupUserPreferences(new UserPreferencesBuilder().BuildMany(count: 1));            

    _productCategoryRepositoryFake.SetupProductCategory(new ProductCategoryBuilder());

    var products = new ProductBuilder().BuildMany(count: 3);

    _productRepositoryFake.SetupProducts(products);

    ECommerceService sut = CreateSUT();

    var userIdentity = new Guid();
    var productCategoryId = 1;

    //Act
    var result = sut.ProductsByCategory(userIdentity, productCategoryId);

    //Assert
    Assert.Equal(result, products);
}
```

Each of our models will have one builder class.

The test is now pretty thin comparing to its first version. With a combination of helper methods, extension methods to setup mocks and the Fluent Builder Pattern to create objects, we were able reduce a lot of the verbosity and duplicated code from the arrange phase. The amalgamation of these techniques, made the code cleaner and easier to understand.

## **Further Reading and References**

- [TDD Antipatterns – Agile in a Flash](http://agileinaflash.blogspot.com/2009/06/tdd-antipatterns.html)

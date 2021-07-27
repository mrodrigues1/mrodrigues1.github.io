---
layout: post
title: 'Clean Code: The Boy Scout Rule'
date: 2017-12-11 08:00:01.000000000 -03:00
type: post
categories: cleancode
permalink: "/2017/12/11/clean-code-boy-scout-rule/"
image: 
  path: /assets/img/blog/boy-1822631_1920.jpg
description: >
  Always leave the camp cleaner than you found.
---
In the first chapter of the clean code, Uncle Bob talks about the boy scout rule, which can be summarized as:

**Leave the code cleaner than you found it.**

Meaning that every time that we make a change in our code base, make an improvement in the code.

The rule of the boy scouts is: “Always leave the campground cleaner than you found it”. When you find a mess on the ground, clean it, doesn't matter who did it. Your job is to always leave the ground cleaner for the next campers.

Robert Baden-Powell, the father of scouting, who wrote the original form of this expression, “Try and leave the world a little better than you found it”.

What if we could apply this rule to our code?

## **The Broken Windows Theory**

One broken window, left without repair for a certain amount time, consequently, will leave the impression of negligence. More windows will be broken, graffiti appears, damage in the structure appears and the sense of abandon becomes present. All it takes is just one broken window and everything can crumble.

![]({{ site.baseurl }}/assets/2017/12/lost-places-2745713_1280-2592703991-1512643801265-1024x576.jpg){:.lead}

Broken Windows
{:.figcaption}

Have you ever worked in a code base that fits this description? I certainly have.

We started to write some code for a specific functionality and after sometime the code start to grow. We add more functionalities on top of that, find edge cases that needs to be treated as well. In the end, it becomes unbearable, hard to understand and long past the point of maintainability. When the code reaches this point, it's better to just rewrite the whole thing.

The idea behind this theory is to never let any window broken, every time you found a broken window in your code, just fix it, don't let it spread in your code base.

The boy scout rule takes the broken windows theory to a new level, you not only fix the problems as soon as they appear but you also improve the code. Applying these two practices will lead the code base in to a much  better state in the future.

## **Applying To Code**

There are many ways to apply the boy scout rule to our code, it doesn't have to be something big. You can rename a variable to a more meaningful name, get rid of a small duplication or break up a long function.

Have you ever imagine working in a project where the code got better over time?

Continuous improvement is an integral part of our job as software developers.

## **Further Reading and References**

- [Clean code: Chapter 1 - Boy Scout Rule - Robert C. Martin](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship-ebook/dp/B001GSTOAM)
- [The Boy Scout Rule - Programmer 97 Things](http://programmer.97things.oreilly.com/wiki/index.php/The_Boy_Scout_Rule)
- [The Boy Scout Rule - Arun Sasidharan](https://dev.to/_arunsasi/the-boy-scout-rule)
- [Robert Baden-Powell - Wikipedia](https://en.wikipedia.org/wiki/Robert_Baden-Powell,_1st_Baron_Baden-Powell)

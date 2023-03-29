[Introduction](./Introduction.md) | [Chapter 1. The basics.](./Chapter1.md) | [Chapter 2. HTML properties.](./Chapter2.md) | [Chapter 3. Events.](./Chapter3.md) | [Chapter 4. Basic data storage.](./Chapter4.md) | [Chapter 5. Canvas and images.](./Chapter5.md)

# Purescript web programming basics tutorial: Halogen versus purescript HTML.
# Introduction
 This tutorial is meant as an aid for people who wish to develop web HTML using purescript. We will show how to produce HTML5 code using purescript without Halogen, and compare the results with those made with Halogen. Although Halogen is the current 'standard' for purescript HTML programming, we will show that you may not always need it. And even if you only wish to use Halogen for your web programming in purescript, this tutorial will help you to develop a deeper understanding of Halogen. 
 
 For a more general introduction into purescript we refer to: 'Purescript by Example' (https://book.purescript.org/index.html). This general introduction is highly recommended to those who are not familiar with functional languages like Haskell (https://www.haskell.org/). However, unlike our tutorial, 'Purescript by Example' is focussed on the functional aspects of purescript, not on HTML applications.

This tutorial is inspired by the article: "Functional Programming for the Web: Getting Started with PureScript" by Kevin B. Greene (https://medium.com/@KevinBGreene/functional-programming-for-the-web-getting-started-with-purescript-7387f8888318) and we highly recommend this article. 

**Note:** There is a big difference between this tutorial and Kevin B. Greene's in the sense that in this tutorial we always provide code that you may immediately try yourself on the 'Try Purescript' web site (https://try.purescript.org), just like the Halogen Guide (see below) does, while Kevin B. Greene provides instructions to run the code on your own machine. Both types of code differ slightly and are not always mutually exchangeable.
## The Halogen Guide
The Halogen module framework is explained in detail in the 'Halogen Guide' (https://github.com/purescript-halogen/purescript-halogen/tree/master/docs/guide). This tutorial will often refer to the examples in the Halogen Guide and show our HTML alternatives for comparison.
## Halogen 
Halogen is built around 'components'. However, although 'Components' are the fundamental building block of Halogen, the Halogen Guide states it is also possible to use the more basic Halogen HTML. However, the Guide also states that: 

>"... pure functions that produce HTML lack many essential features that a real world application needs: state that represents values over time, effects for things like network requests, and the ability to respond to DOM events (for example, when a user clicks a button)."

This suggests that purescript HTML in itself would not be able to provide these functions. However, it is perfectly feasible to have functions respond to DOM events, without resorting to Halogen. In the following chapters we will give a short demonstration of 'common [purescript] HTML' functions responding to DOM events and altering DOM elements in response, without any Halogen.

## Purescript pros and cons.
Before we start, it may be wise to look at the advantages and disadvantages of purescript as a replacement for javascript.

**Pros**
- Purescript brings the power of strongly-typed functional programming to the world of JavaScript development (Purescript by Example (https://book.purescript.org/index.html)).
- Purescript code is compiled before actual use and the strong typing control during this compilation will remove many errors that would remain undetected in javascript even in runtime. For example, the infamous typo in a parameter name which would be considered to be just another parameter with a default value by javascript, would be detected immediately by purescript during compilation (or even earlier, depending on your coding environment).

**Cons**
- Purescript is a relatively young language. It was initially designed by Phil Freeman in 2013 (https://en.wikipedia.org/wiki/PureScript). This means that it is still in full development. With as result that the modules on which it is based, change. One approach of dealing with these changes is to 'lock' the versions of the software, but this means that the code in 'locked' projects runs the risk of becoming outdated and cause safety issues. Which, in our opinion, makes it less attractive for production environments. Without locking, projects will fail in time, when the modules are changed. To give an example: At the time of writing this tutorial, one of the examples in the Halogen Guide, the one on HTTP Requests (https://purescript-halogen.github.io/purescript-halogen/guide/03-Performing-Effects.html), fails when you try it in the 'Try Purescript' web site (https://try.purescript.org), because the 'get' function (https://pursuit.purescript.org/packages/purescript-affjax/13.0.0/docs/Affjax#v:get) in the Affjax-module was changed.
- Purescript has quite a small user base. This means that examples and tutorials, like this one, are rare. If you want to try something new, you will not often find examples or tutorials about it.
- In our opinion, Purescript is generally badly documented. The standard source for information: 'Pursuit' (https://pursuit.purescript.org) almost never gives examples for functions, and the parameters of functions generally lack explanations and often even a description.

We cannot solve the above disadvantage of purescripts age and its development, but we can improve its documentation with examples and a tutorial. So, if you still want to try it, we hope you will enjoy the following chapters.


## Table of contents
- [Chapter 1. The basics.](./Chapter1.md)
- [Chapter 2. HTML properties.](./Chapter2.md)
- [Chapter 3. Events.](./Chapter3.md)
- [Chapter 4. Basic data storage.](./Chapter4.md)
- [Chapter 5. Canvas and images.](./Chapter5.md)
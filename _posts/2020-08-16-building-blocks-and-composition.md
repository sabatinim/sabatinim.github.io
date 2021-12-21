---
layout: custom
title: "Building blocks and composition a first taste of functional programming"
date: 2020-08-16
---
## Overview

> Functional programming is very cool! But how can I start to practice?

if you are a developer who has wondered about this, I'd like to share my personal experience.

I strongly believe in learning by doing. For this reason I decided to use a kata (little coding exercise) to practice.

I choose the Martin Fowler kata video store. Not the refactoring version but the one from scratch. In this way I had the opportunity to have a blank sheet to start with and think about.

This Kata is very simple. I want to have focus on application's domain rather than technical details such as, for example, the use of a DB for persistence or integration with an external service via HTTP.
The purpose of the kata is creating a system able to rent different types of Movies and print the receipt in different formats (plain text and HTML).

### Test first
I started writing a test suite about a specific movie type price calculation:

``` typescript
it('rent new Release Movie for one day', () => {
        expect(moviePriceFor(new Rental(1, newReleaseConfiguration("UNUSED")))).toEqual(3.0)
});
it('rent Children Movie for four day', () => {
        expect(moviePriceFor(new Rental(4, childrenConfiguration("UNUSED")))).toEqual(3.0)
});
```
Writing these tests emerged the concepts of:
- **Rent**
- **Movie type**
- **additional price calculation for each extra day**
- **single movie price calculation**

This is the production code able to makes these tests green:

``` typescript
const additionalCostFor = (rental: Rental): MoviePrices => {
  let additionalCost = 0.0;
  if (rental.rentalDays > rental.mc.minRentDays) {
    const additionalDays = rental.rentalDays - rental.mc.minRentDays
    additionalCost = rental.mc.additionaCostPerDay * additionalDays;
  }
  return new MoviePrices(additionalCost, rental.mc.price);
}

const priceFor = (moviePrices: MoviePrices): number => {
    return (moviePrices.movieBasePrice + moviePrices.additionalCost).toPrecision(5) 
};
```
The first function calculates the additional price and the second adds the price and scale to five decimal places.

We can notice that I have the **'building block'** I can compose
to have a function that calculate the fully price for a single movie type (tadaaa!!!).

>
Let's go and apply composition!

### Composition
At this point I decided to implement the compose function. Obviously we have to write a test before:

``` typescript
it('compose two function', () => {

  let f = (x: string): string => `f(${x})`
  let g = (x: string): string => `g(${x})`

  let gfx: (x: string) => string = compose(f, g)

  expect(gfx("value")).toEqual("g(f(value))")
});
```
Inside the test I define two functions 'f' and 'g' that take an input parameter and return a string with this parameter interpolated.
Composing them I can get a string concatenation.

This is the production code:

``` typescript
export const compose = <A,B,C>(
  f: (x: A) => B,
  g: (y: B) => C):
  (x: A) => C => {
    
    return (x) => g(f(x))
};
```
Using typescript generics I can use it indiscriminately for each pair of functions whose output type of one is the input for the other.

This is the resulting function:

``` typescript
const additionalCostFor = (rental: Rental): MoviePrices => {...}

const priceFor = (moviePrices: MoviePrices): number => {...}

const moviePriceFor: (x: Rental) => number = compose(additionalCostFor, priceFor)
```
The type system is telling me I have a function that takes a Rental and gives back a number that represent price per movie (Maybe I should also have typed the outgoing concept and not leave the primitive obsession :) ).

We can notice that I didn't even have to write a test before bringing out this design because it came out independently and it is the compiler that tells me that the two functions compose (WOOOOW!).

>
Try to compose again!

## Curry
By creating basic functions (building blocks) it is possible to compose them by creating more complex functions in an automatic and natural way, this pushes to have a code in which the responsibilities are very clear and isolated and makes for an excellent degree of cohesion and coupling.

In fact, for the total price calculation I just had to reuse the calculation of the single Movie after having inject it by the curry and apply it with map reduce.

``` typescript
const additionalCostFor = (rental: Rental): MoviePrices => {...}

const priceFor = (moviePrices: MoviePrices): number => {...}

const moviePriceFor: (x: Rental) => number = compose(additionalCostFor, priceFor)

export const totalPrice = (moviePriceFor:(r:Rental) => number):(rentals:Rental[])=> number =>{
  return (rentals) => rentals.map(r=>moviePriceFor(r)).reduce((x,y)=>x+y);
}
```
Curry only partially applies the function and return a configured function.

## Software Modularization
The total price calculation function is exported from the pricing module because they are used by the module responsible to print the receipt in html and by the module responsible to print the receipt in plain text.

This means that I have defined the public interface between the modules (usin), which among other things can help to test the various modules individually.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dv1kpxtmcevjwufdqvv2.jpg)

## Considerations
The building blocks are the leaves of our software that can be composed to have more complex functions.
With functional programming, You are dealing with Functions as the basic building block. Each function can be thought of as a lego Bricks

A pure function is by definition isolated. Unlike Encapsulation where an Object is trying to hide things from you, a pure function can not do anything It did not declare in its interface (or signature). You could say that a Pure function is "honest".

This causes a paradigm shift because You need to think of solving problems by breaking them down into these small isolated functions and then re-assembling them at your application entry point.
This might seem counter-intuitive at first but then when you open you mind to the possibilities, It fundamentally changes how you think about building software.

Originally published at [https://sabatinim.github.io/] (https://sabatinim.github.io/) on August 16, 2020.

## Next
[Second round](https://dev.to/sabatinim/building-blocks-and-composition-2nd-round-c1i)

## References
[Github code](https://www.github.com/sabatinim/video-store-kata/tree/master/typescript)
[Scott Wlashin the power of composition](https://www.slideshare.net/mobile/ScottWlaschin/the-power-of-composition) 




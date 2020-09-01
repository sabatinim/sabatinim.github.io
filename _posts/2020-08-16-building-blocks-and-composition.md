---
layout: default
title: "Building blocks and composition a first taste of functional programming"
date: 2020-08-16
---
## Overview
How many of us developers have ever tried to put functional programming principles into practice? If you are among them or you are curious and you want to do it, this article can probably be a starting point.

In information technology the best return on investment in training is try to apply principles on a real example. For this reason I decided to use a kata (little coding exercise) to practice. 

I choose the Martin Fowler kata video store. Not the refactoring version but the one from scratch. In this way I had the opportunity to have a blank sheet to start with and think about.

This Kata is very simple but it forces us to think about the domain of the application to be created rather than technical details such as, for example, the use of a DB for persistence or integration with an external service via HTTP or similar protocols. The purpose of the kata is create a system able to rent different types of Movies and print the receipt in different formats. 

### Test first
The first thing I did was to deal with the problem of renting a video by writing a series of tests relating to the calculation of the price of the single type of movie, such as:

``` typescript
it('rent new Release Movie for one day', () => {
        expect(moviePriceFor(new Rental(1, newReleaseConfiguration("UNUSED")))).toEqual(3.0)
});
it('rent Children Movie for four day', () => {
        expect(moviePriceFor(new Rental(4, childrenConfiguration("UNUSED")))).toEqual(3.0)
});
```

Writing these tests emerged the concepts of **Rent**, **type of Movie**, **additional price calculation for each extra day** and **single movie price calculation**. 

So in my production code I have these two function: 

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

The first function calculates the additional price and the second add the price and scale to five decimal places.

We can notice that I have the **'building block'** I can compose 
to have a function that calculate the fully price for a single movie type. Let's go and apply composition! 

### Composition
At this point I decided to implement the compose function writing a test before production code:

``` typescript
it('compose two function', () => {

  let f = (x: string): string => `f(${x})`
  let g = (x: string): string => `g(${x})`

  let gfx: (x: string) => string = compose(f, g)

  expect(gfx("value")).toEqual("g(f(value))")
});
```
Inside the test I define two functions 'f' and 'g' and I compose them obtaining a concatenation of strings.

The production code is the following:

``` typescript
export const compose = <A,B,C>(
  f: (x: A) => B,
  g: (y: B) => C):
  (x: A) => C => {
    
    return (x) => g(f(x))
};
```

The compose function does nothing but compose two functions by generalizing the types of inputs and outputs.
In this way I can use it indiscriminately for each pair of functions whose output type of one is the input for the other.

Returning to our case of calculating the pricing of a movie, I can compose the functions as follows:

``` typescript
const moviePriceFor: (x: Rental) => number = compose(additionalCostFor,priceFor)
```

In this way the type system is telling me I have a function that takes a Rental and gives back a number that represent price per movie (Maybe I should also have typed the outgoing concept and not leave the primitive obsession :) ).

We can notice that I didn't even have to write a test before bringing out this design because it came out independently and it is the compiler that tells me that the two functions compose. 
Try to compose again!  

## Curry
By creating basic functions (building blocks) it is possible to compose them by creating more complex functions in an automatic and natural way, this pushes to have a code in which the responsibilities are very clear and isolated and makes for an excellent degree of cohesion and coupling.
In fact, for the total price calculation I just had to reuse the calculation of the single Movie after having inject it by the curry and apply it with map reduce.

``` typescript
export const totalPrice = (moviePriceFor:(r:Rental) => number):(rentals:Rental[])=> number =>{
  return (rentals) => rentals.map(r=>moviePriceFor(r)).reduce((x,y)=>x+y);
}
```
Curry only partially applies the function and return a configured function. 

## Software Modularization
The price calculation functions for single Movie and calculation of the total are exported from the pricing module because they are used by the module responsible to print the receipt in html and by the module respondible to print the receipt in plain text.

This means that I have defined the public interface between the modules, which among other things can help to test the various modules individually.

## Considerations
The building blocks are the leaves of our software that can be composed to have more complex functions. 
With functional programming, You are dealing with Functions as the basic building block. Each function can be thought of as a lego Bricks

A pure function is by definition isolated. Unlike Encapsulation where an Object is trying to hide things from you, a pure function can not do anything It did not declare in its interface (or signature). You could say that a Pure function is "honest".

This causes a paradigm shift because You need to think of solving problems by breaking them down into these small isolated functions and then re-assembling them at your application entry point.
This might seem counter-intuitive at first but then when you open you mind to the possibilities, It fundamentally changes how you think about building software.

## Next
Monads :) 

## References
[Github code](https://www.github.com/sabatinim/video-store-kata/tree/master/typescript) 
[Scott Wlashin the power of composition](https://www.slideshare.net/mobile/ScottWlaschin/the-power-of-composition) 


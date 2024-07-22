---
layout: custom
title: "Building blocks and composition a first taste of functional programming"
date: 2020-08-16
---

## Overview

Functional programming is fascinating, but how can you start practicing it? If you're a developer who has been pondering
this question, I'd like to share my personal journey and experiences.

I firmly believe in the "learning by doing" approach. That's why I decided to use a coding exercise known as a kata to
practice functional programming. Specifically, I chose the Martin Fowler kata for the video store, not the refactoring
version but the one starting from scratch. This approach provided me with a clean slate to work with, enabling me to
focus on the application's domain rather than getting bogged down in technical details. I opted for TypeScript to
leverage the functional capabilities offered by the language.

The chosen kata is rather straightforward. My aim was to concentrate on the application's domain logic rather than
dealing with technical intricacies like database persistence or external HTTP service integration. The primary objective
of this kata was to create a system capable of renting different types of movies and generating receipts in various
formats, such as plain text and HTML.

### Test first

My journey began by writing a test suite to calculate the price for specific movie types:

``` typescript
it('rent new Release Movie for one day', () => {
        expect(moviePriceFor(new Rental(1, newReleaseConfiguration("UNUSED")))).toEqual(3.0)
});
it('rent Children Movie for four day', () => {
        expect(moviePriceFor(new Rental(4, childrenConfiguration("UNUSED")))).toEqual(3.0)
});
```

While writing these tests, I discovered key concepts like rental, movie types, additional price calculations for extra
days, and individual movie price calculations.

I implemented the following production code to pass these tests:

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

The first function calculates the additional cost, while the second function adds the base price and rounds it to five
decimal places.

At this point, I realized that I had the essential building blocks needed to compose a function that calculates the
total price for a single movie type.

>
Let's go and apply composition!

### Composition

Next, I decided to implement a compose function, but of course, I wrote a test first:

``` typescript
it('compose two function', () => {

  let f = (x: string): string => `f(${x})`
  let g = (x: string): string => `g(${x})`

  let gfx: (x: string) => string = compose(f, g)

  expect(gfx("value")).toEqual("g(f(value))")
});
```

In this test, I defined two functions, 'f' and 'g', which take an input parameter and return a string with that
parameter interpolated. By composing them, I achieved string concatenation.

The production code for the compose function looks like this:

``` typescript
export const compose = <A,B,C>(
  f: (x: A) => B,
  g: (y: B) => C):
  (x: A) => C => {
    
    return (x) => g(f(x))
};
```

Using TypeScript generics, I created a versatile compose function that can be used for any pair of functions where the
output type of one matches the input type of the other.

With this compose function in place, I was able to compose the additionalCostFor and priceFor functions like so:

``` typescript
const additionalCostFor = (rental: Rental): MoviePrices => {...}

const priceFor = (moviePrices: MoviePrices): number => {...}

const moviePriceFor: (x: Rental) => number = compose(additionalCostFor, priceFor)
```

Thanks to the type system, I didn't even need to write a test for this specific composition because it naturally
emerged, and the compiler confirmed that the functions could be composed successfully.

>
Try to compose again!

## Leveraging Curry

By creating these basic building blocks, I could easily compose them to create more complex functions. This approach
encourages clear and isolated responsibilities, leading to excellent cohesion and loose coupling.

For the total price calculation, I reused the calculation for individual movies by currying the function and applying it
using map and reduce:

``` typescript
const additionalCostFor = (rental: Rental): MoviePrices => {...}

const priceFor = (moviePrices: MoviePrices): number => {...}

const moviePriceFor: (x: Rental) => number = compose(additionalCostFor, priceFor)

export const totalPrice = (moviePriceFor:(r:Rental) => number):(rentals:Rental[])=> number =>{
  return (rentals) => rentals.map(r=>moviePriceFor(r)).reduce((x,y)=>x+y);
}
```

Currying allowed me to partially apply the function and return a configured function, making composition even more
powerful.

## Modularization of Software

To maintain clean and modular code, I exported the total price calculation function from the pricing module. This
function was used by modules responsible for printing receipts in HTML and plain text formats.

By doing so, I defined a clear public interface between the modules. I also had the flexibility to mock this function to
facilitate testing of the printing modules (HTML and plain text).

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dv1kpxtmcevjwufdqvv2.jpg)

## Final Thoughts

In functional programming, functions are the fundamental building blocks. Each function can be thought of as a Lego
brick, and pure functions are inherently isolated. Unlike encapsulation, where an object tries to hide information, pure
functions can only do what they declare in their interface or signature, making them "honest."

This paradigm shift encourages problem-solving by breaking them down into small, isolated functions and then
reassembling them at the application's entry point. While this approach may seem counterintuitive initially, it
fundamentally changes how you think about building software.

## Next

If you'd like to explore this topic further, check out
the [second round](https://dev.to/sabatinim/building-blocks-and-composition-2nd-round-c1i) of my journey.

## References

Originally published at [https://sabatinim.github.io/] (https://sabatinim.github.io/) on August 16, 2020.
[Github code](https://www.github.com/sabatinim/video-store-kata/tree/master/typescript)
[Scott Wlashin the power of composition](https://www.slideshare.net/mobile/ScottWlaschin/the-power-of-composition)
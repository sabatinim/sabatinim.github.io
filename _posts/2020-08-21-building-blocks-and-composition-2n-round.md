---
layout: custom
title: "Building blocks and composition a 2nd round"
date: 2020-08-21
---
In my [previous post](https://sabatinim.github.io/blog/2020/08/16/building-blocks-and-composition), I talked about functional programming's fundamental building blocks and introduced the Open-Closed Principle (OCP). Today, I want to explore software modularization and how we can implement the OCP using functional programming techniques.

The Open-Closed Principle (OCP) may sound simple, but it can be challenging to implement effectively: "Software should be open for extension but closed for modification." Achieving this principle is crucial for maintaining agility and adaptability in your development process.

Let's revisit the context of our Video Store Kata. We've already implemented the ability to print receipts in plain text. However, a new requirement has arisen: we need to print receipts in HTML format without altering our existing codebase. This is where the Open-Closed Principle comes into play.

To fulfill this requirement, we'll start by examining our receipt module:


``` typescript
class PrintableMovie {
    title: string;
    priceRepresentation: string;

    constructor(title: string, priceRepresentation: string) {
        this.title = title;
        this.priceRepresentation = priceRepresentation;
    }
}

const printableMovieWith =
    (calculateMoviePrice: (r: Rental) => number) =>
        (r: Rental) => new PrintableMovie(r.mc.title, calculateMoviePrice(r).toPrecision(2));

export const printableMovie: (r: Rental) => PrintableMovie =
    printableMovieWith(calculateMoviePrice);
```
This module is designed to be generic. It defines a PrintableMovie data type for items that need to be printed. It also includes two functions:

1) **printableMovie** transforms a Rental into a **PrintableMovie**.
2) **printableMovieWith** takes a price calculation function as input and prints the price with a precision of two digits.

This serves as a contract between the pricing module and the receipt module. Defining this contract using functions allows us to test pricing and receipt modules as separate black boxes.

With this foundation, we can now generalize the print receipt function:

``` typescript
export const genericReceipt =
    (header: (user: string) => string,
     body: (rentals: Rental[]) => string,
     footer: (rentals: Rental[]) => string,
     rentalPoint: (rentals: Rental[]) => string) =>

        (user:string, rentals:Rental[]) =>
            header(user) +
            body(rentals) + "\n" +
            footer(rentals) + "\n" +
            rentalPoint(rentals)
```
Now, we can implement both the plain text and HTML templates.

For plain text:

``` typescript
const textMovieReceipt = (m: PrintableMovie): string =>
     `- ${m.title} ${m.priceRepresentation}`

const textMoviesReceiptWith = (
    movieReceiptFunc: (x: Rental) => string) =>
     (rentals: Rental[]) => rentals.map(r => movieReceiptFunc(r)).join("\n")

const textFooterReceiptWith = (
    totalPrice: (rentals: Rental[]) => number) =>
     (rentals: Rental[]) => `Total ${totalPrice(rentals).toPrecision(2)}`

const textFooterRentalPointReceiptWith = (
    calculateRentalPoint: (rentals: Rental[]) => number) =>
     (rentals: Rental[]) => `Total Rental points ${calculateRentalPoint(rentals)}`

//WIRING HERE
const textFooterRentalPointReceipt =
    textFooterRentalPointReceiptWith(calculateRentalPoints);

const textFooterReceipt: (rentals: Rental[]) => string =
    textFooterReceiptWith(calculateTotalMoviesPrice);

const textMoviesReceipt: (rentals: Rental[]) => string =
    textMoviesReceiptWith(compose(
        printableMovie,
        textMovieReceipt))

const textHeader = (user: string) => `Hello ${user} this is your receipt\n`;

//WIRING THE PRINT FUNCTION WITH PLAIN TEXT BEHAVIOUR
export const printTextReceipt: (user: string, rentals: Rental[]) => string =
    genericReceipt(
        textHeader,
        textMoviesReceipt,
        textFooterReceipt,
        textFooterRentalPointReceipt)
```
And for HTML:

``` typescript
const htmlMovieReceipt = (m: PrintableMovie): string =>
    `<li>${m.title} ${m.priceRepresentation}</li>`

const htmlMoviesReceiptWith = (
    htmlMovieReceipt: (x: Rental) => string) =>
    (rentals: Rental[]) => `<ul>\n${rentals.map(r => htmlMovieReceipt(r)).join("\n")}\n</ul>`

const htmlFooterReceiptWith = (
    calculateMoviesTotalPrice: (rentals: Rental[]) => number) =>
    (rentals: Rental[]) => `<br>You owed ${calculateMoviesTotalPrice(rentals).toPrecision(2)}`

const htmlFooterRentalPointReceiptWith = (
    calculateRentalPoint: (rentals: Rental[]) => number) =>
    (rentals: Rental[]) => `<br>You earned ${calculateRentalPoint(rentals)} frequent renter points\n</body>\n</html>`

//WIRING HERE
const htmlFooterRentalPointReceipt: (rentals: Rental[]) => string =
    htmlFooterRentalPointReceiptWith(calculateRentalPoints);

const htmlFooterReceipt: (rentals: Rental[]) => string =
    htmlFooterReceiptWith(calculateTotalMoviesPrice);

const htmlMoviesReceipt: (rentals: Rental[]) => string =
    htmlMoviesReceiptWith(compose(
        printableMovie,
        htmlMovieReceipt))

const htmlHeader = (user: string) =>
    `<!DOCTYPE html>\n` +
    `<html>\n` +
    `<head>\n` +
    `<title>Video store - statement for ${user}</title>\n` +
    `</head>\n` +
    `<body>\n` +
    `<h1>Rental Record for ${user}</h1>\n`

//WIRING THE PRINT FUNCTION WITH HTML TEXT BEHAVIOUR
export const printHtmlReceipt: (user: string, rentals: Rental[]) => string =
    genericReceipt(
        htmlHeader,
        htmlMoviesReceipt,
        htmlFooterReceipt,
        htmlFooterRentalPointReceipt)
```
The code structure remains consistent for both templates. We've successfully achieved the Open-Closed Principle – our code is open for extension (to add new templating formats) but closed for modification, which ensures robustness.

A key takeaway is the importance of an emerging design. My initial version was different from the current one, and I had to refactor my code before implementing the new HTML receipt feature. Continuous refactoring is essential for maintaining a resilient architecture.

Originally published at https://sabatinim.github.io/

## References
[Github code](https://www.github.com/sabatinim/video-store-kata/tree/master/typescript)
[Scott Wlashin the power of composition](https://www.slideshare.net/mobile/ScottWlaschin/the-power-of-composition) 

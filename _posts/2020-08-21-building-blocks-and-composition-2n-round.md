---
layout: custom
title: "Building blocks and composition a 2nd round"
date: 2020-08-21
---

Hi guys, again here after my previous [post](https://sabatinim.github.io/blog/2020/08/16/building-blocks-and-composition). 
I want to add some hints regarding software modularisation. The example is always our video store kata. 
I'd like to share with you how in functional style manage the OCP principle. 
The OCP definition say: **software should be open for extension, but closed for modification.**
This is very simple to understand but very hard to achieve and for me it's the base for the team agility (resiliency to changes).
Coming back to our example we have to print the receipt in plaint text but we also have the secret requirement to print it also in HTML. In order to do this change, we don't want to change our code but we want to extend it in perfect OCP style :)
I'm going to show you the receipt module:

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
This module is quite generic. 
I implemented a **PrintableMovie** data type in order to represent something that should be printed. 
I also implemented two function, the **printableMovieWith** is a function that wants the price calculation function as currying and **printableMovie** that is responsible to transform a Rental into a PrintableMovie. 
This is the contact point between pricing module and receipt module.

Is very useful to have this decoupling because we could test pricing and receipt like they're two black box (for example inject a price function as stub and testing only the templating).

At this point we have to generalise the print receipt function like:

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
Ok we can notice some duplication like **(rentals: Rental[]) => string** but we could accept it now :)

Now we're able to implement the plain text template and the html one. Let's go and show me the code. 

For plain text we have: 

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
Instead for HTML we have:

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

Ok the code is more or less the same. The only things I had to do were implement the different templating functions and wire them using the **genericReceipt** function. 
This means that my code is OPEN for extension and CLOSE for modification.

This brings a lots of benefits because is very easy inject new behaviours (different templating format). 

The fundamental thing to understand is this: we have to make sure that our design is **emerging.**
My first version was very different from the actual design. I had to refactor my code before I had to implement the new feature (HTML receipt). 
This is why **continuous refactoring** practice is very important for our architecture.

## References
[Github code](https://www.github.com/sabatinim/video-store-kata/tree/master/typescript) 
[Scott Wlashin the power of composition](https://www.slideshare.net/mobile/ScottWlaschin/the-power-of-composition) 


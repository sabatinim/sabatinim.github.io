---
layout: custom
title: "FP Architecture"
date: 2021-11-22
---
I've been coding the [birthday greeting](http://matteo.vaccari.name/blog/archives/154.html) in kotlin. This exercise is very useful if you want to learn more about architect your code separating domain from infrastructure code.
I also had the purpose of making some practice with functional programming, in particular I wanted to use some functional data structure like monads.

## Start
I started with some doubts: what kind of monad do I have to use? IO monad? Either monad? Both?
I can't find a response. I wanted something that made my code easily readable; I decided to use Either monad because my aim was to make the method signatures explicit.

I used the [arrow](https://arrow-kt.io/docs/0.11/apidocs/arrow-core-data/arrow.core/-either/) implementation of the data structure. Docs is very clear and full of examples.

Either monad is like:
``` kotlin
// This is the data structure
Either<ErrorType, ResponseType>

// Success case
Either.Right(ResponseType())

// Error case
Either.Left(ErrorType("This is the Error!"))
```
It helps to manage cases like a service that may result in a connection issue, or any unexpected JSON response.

## Kata
Basically, my software will be a composition of functions.
The exercise asks for a software that is able to find employees whose birthday is today and send them emails.
I need to load employees from a data source (a file for this exercise), filter them, and send email.

My software could get an error when it's loading employees or sending Email.
It's curious to notice that when we filter employees (is your birthday?) nothing could go wrong because is something related our domain logic (pure function).

``` kotlin
loadEmployees: () -> Either<MyError, Employees>

filterEmployees: (Employees) -> BirthdayEmployees

sendBirthdayNotificationTo: (BirthdayEmployees) -> Either<MyError, Unit>
```
These are the functions that I will compose into my use case. It's very clear what will be the purpose of code and when I could get an error.
I just need to inject the implementations of the functions using dependency injection.
As a result, I have a function responsibly to execute the composition (tadaaa!).

``` kotlin
fun sendGreetingsWith(
    loadEmployees: () -> Either<MyError, Employees>,
    filterEmployees: (Employees) -> BirthdayEmployees,
    sendBirthdayNotificationTo: (BirthdayEmployees) -> Either<MyError, Unit>
): () -> Either<MyError, Unit> = {

    loadEmployees()
        .map(filterEmployees)
        .flatMap(sendBirthdayNotificationTo)
}
```
I can specify the types of errors I can get during execution, modelling them using sealed classes:

``` kotlin
sealed class MyError {
    data class LoadEmployeesError(val msg: String) : MyError()
    data class SendMailError(val msg: String) : MyError()
}
```
Moreover in this situation the code is quite obvious and we don't need any comment to understand what is the purpose.

## Testing
It's very curious to see how easily is testing after separating infrastructure from the domain.
For example: in this acceptance test I could inject the 'load employee logic' using in memory implementation and running the test without having additional configurational stuff for the infrastructure part.

``` kotlin
    @Test
    internal fun oneEmployeeIsBornTodayAnotherIsNotBornToday() {

        val inMemoryLoadEmployee: () -> Either<Nothing, Employees> = {
            Right(
                Employees(
                    listOf(
                        employeeBirthday(dateOfBirth(TODAY), "TODAY_EMPLOYEE"),
                        employeeBirthday(dateOfBirth(TOMORROW), "TOMORROW_EMPLOYEE")
                    )
                )
            )
        }

        val sendGreetings: () -> Either<MyError, Unit> =
            sendGreetingsWith(
                inMemoryLoadEmployee,
                todayBirthdayEmployees,
                sendMailWith("localhost",9999)
            )

        val result = sendGreetings()

        val message = mailServer.receivedEmail.next() as SmtpMessage
        assertThat(message.body).isEqualTo("Happy birthday, dear TODAY_EMPLOYEE!")

        assertThat(result).isEqualTo(Right(Unit))
    }

```
Just inject the in memory behaviour and execute the test (voila').

### Do I need to test infrastructure?
Yes of course!
After separating domain logic from the infrastructure logic, It's very easy testing infrastructure code in isolated mode using an integration test like these:

``` kotlin
    @Test
    internal fun loadEmployeeFromFile() {

        val loadEmployeeFromFile = loadEmployeeFrom("./target/test-classes/employees.txt")

        assertThat(loadEmployeeFromFile()).isEqualTo(Right(Employees(listOf(Employee("Marco","Sabatini", DateOfBirth(5,3,1983),
            EmailAddress("address@email.com")
        )))))
    }

    @Test
    internal fun employeeNotValid() {

        val loadEmployeeFromFile = loadEmployeeFrom("./target/test-classes/employeesNotValid.txt")

        assertThat(loadEmployeeFromFile()).isEqualTo(Left(MyError.LoadEmployeesError("Error For input string: \"address@email.com\"")))
    }

    @Test
    internal fun fileNotFound() {

        val loadEmployeeFromFile = loadEmployeeFrom("NOT_EXIXSTING_FILE")

        assertThat(loadEmployeeFromFile()).isEqualTo(Left(MyError.LoadEmployeesError("File NOT_EXIXSTING_FILE doesn't exist")))
    }
```
Imagine that we have to load data from database. We just could write the jdbc implementation of loadEmployees and we could use a docker container for making integration test and voila' ([see on my post](https://dev.to/maverick198/tests-infrastructure-1gko)).
In this way all these integration tests will become a tests container or if we have to integrate with external services (like API) we could write just contract tests.

## End
From a design point of view I have a use case responsible to orchestrate the flow.
We can see three main responsibilities:
- Load data
- Execute some domain logic
- Send email messages

The software is quite generic and using dependency injection is very easy loading data from another data source or sending another kind of message like for example a mobile notification, just implementing these functions:

``` kotlin
loadEmployees: () -> Either<MyError, Employees>

sendBirthdayNotificationTo: (BirthdayEmployees) -> Either<MyError, Unit>
```
This is the power of separating domain logic from infrastructure.

Regarding functional programming, we can say 'Either' monad make my code more readable even if I can achieve the same result using Object Oriented programming with exceptions.

I would think about monads like a meta-language on top of our programming language, able to create something understandable for developers (a sort of dev ubiquitous language).
If we had developed the exercise using 'Either' monad with another language probably we'll have the same result.

### References
- [Exercise code](https://github.com/sabatinim/birthday-greetings)
- [Arrow](https://arrow-kt.io/)
- [Accelerate Development with Simple Design - Matteo Vaccari](https://www.youtube.com/watch?v=5-HWNVoFLX8)
- [Reinventing the Transaction Script - Scott Wlaschin](https://www.youtube.com/watch?v=USSkidmaS6w)

#### P.S.
This is an exercise. I could resolve it in a lot of different ways.
Don't focus on technical stuff but on design and testing!


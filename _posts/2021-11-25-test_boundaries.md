---
layout: custom
title: "Test Boundaries"
date: 2021-11-25
---
## Overview
Separating infrastructure from domain permit to identify the boundaries of our system like the *"part of software"* responsible to communicate with *"external world"* (DBMS, external API, web). We'll analyze testcontainers technique to build a test suite for our system boundaries.
Starting from [test containers](https://www.testcontainers.org/) docs we know that it makes the following kinds of tests easier:

- **Data access layer integration tests:** use a containerized instance of a MySQL, PostgreSQL or Oracle database to test your data access layer code for complete compatibility, but without requiring complex setup on developers' machines and safe in the knowledge that your tests will always start with a known DB state. Any other database type that can be containerized can also be used.

- **Application integration tests:** for running your application in a short-lived test mode with dependencies, such as databases, message queues or web servers.

- **UI/Acceptance tests:** use containerized web browsers, compatible with Selenium, for conducting automated UI tests. Each test can get a fresh instance of the browser, with no browser state, plugin variations or automated browser upgrades to worry about. And you get a video recording of each test session, or just each session where tests failed.

Well! we're going to try it on a pet code. We'll not use testcontainers framework, just docker with some script to configure it. Let's start!

## Kata
Regarding [my previous post](https://dev.to/maverick198/fp-architecture-2o5i), we want to write the DB access layer for this function:

``` kotlin
loadEmployees: () -> Either<Error, Employees>
```
I also want to use docker infrastructure for testing it.
I remember that I have some integration tests for FileLoadEmployee function:

``` kotlin
@Test
internal fun loadSomeEmployees()

@Test
internal fun employeeNotValidEmail()
```
They are quite clear so we can avoid to describe them.
I'd like to use them for the DB implementation but I don't want to duplicate test code. I'm going to create an abstraction of these tests that could be used for both implementations (File and DB).

``` kotlin
abstract class LoadEmployeeTest {

    @Test
    internal fun loadEmployee() {
        val loadEmployees = instance()

        assertThat(loadEmployees()).isEqualTo(
            Either.Right(
                Employees(
                    listOf(
                        Employee(
                            "Marco", "Sabatini", DateOfBirth(5, 3, 1983),
                            EmailAddress("address@email.com")
                        )
                    )
                )
            )
        )
    }

    @Test
    internal fun employeeNotValid() {
        val loadEmployees = wrongInstance()

        assertThat(loadEmployees()).isEqualTo(Either.Left(Error("Error For input string: \"wrong\"")))
    }

    abstract fun instance(): () -> Either<Error, Employees>
    abstract fun wrongInstance(): () -> Either<Error, Employees>
}
```
Basically I'll have two abstract methods to implement. These methods must return the relative function implementation (File or DB) for happy path and corner cases.

For File access layer I will have (tadaaa!):

``` kotlin
override fun instance(): () -> Either<Error, Employees> =
        loadEmployeeFrom("./target/test-classes/employees.txt")

    override fun wrongInstance(): () -> Either<Error, Employees> =
        loadEmployeeFrom("./target/test-classes/employeesNotValid.txt")

    @Test
    internal fun fileNotFound() {

        val loadEmployeeFromFile = loadEmployeeFrom("NOT_EXIXSTING_FILE")

        Assertions.assertThat(loadEmployeeFromFile())
            .isEqualTo(Either.Left(Error("File NOT_EXIXSTING_FILE doesn't exist")))
    }
```
For DB access layer I will have (tadaaa!):

``` kotlin
  @AfterEach
    fun cleanupTest() {
        execute("DELETE from employees")
    }

    override fun instance(): () -> Either<Error, Employees> {
        execute("INSERT INTO employees VALUES ('Marco', 'Sabatini','05/03/1983','address@email.com')")

        return loadEmployeeWith()
    }

    override fun wrongInstance(): () -> Either<Error, Employees> {
        execute("INSERT INTO employees VALUES ('', '','wrong','')")

        return loadEmployeeWith()
    }

    private fun execute(sql: String) {
        val stmt = connection().createStatement()
        stmt!!.executeUpdate(sql)
        stmt.close()
    }
```
This technique is called [contract test](https://blog.thecodewhisperer.com/permalink/getting-started-with-contract-tests) and is very useful when we have to define a specific behaviour in our codebase and we have different implementation of it.
In this case I have defined loadEmployees behaviour between my domain code and my infrastructure code and I have different implementations tested. It's very useful if we want to have in memory implementation of external 'port' and use them in our acceptance test (super fast!!!).

## Docker && Docker Compose
Now it's time to move to infrastructure part. I need a DBMS mysql instance and a jvm maven runtime where my kotlin test code can run. These containers have to communicate each other.
This is the docker compose file configuration I used:

``` yaml
services:
  mysql:
    image: mysql
    container_name: mysql_server
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=pwd
    networks:
      - db-network

   db_client:
    build:
      context: ./containers/client
    networks:
      - db-network
    environment:
      - WAIT_HOSTS=mysql:3306
      - WAIT_HOSTS_TIMEOUT=300
      - WAIT_SLEEP_INTERVAL=30
      - WAIT_HOST_CONNECT_TIMEOUT=30

  maven:
    image: maven
    container_name: builder
    volumes:
      - ${PWD}:/tmp
    networks:
      - db-network

networks:
  db-network:
```
- **mysql**: DBMS
- **db_client**: is a mysql client command line interface. It is responsible to create schema and could be used also for other purposes (ex. load demo test data etc.).
  It is using an entrypoint docker to create schema or load demo data and this [plugin](https://github.com/kassambara/docker-compose-wait-for-container) that helps waiting for mysql instance is started.

``` bash
case "$@" in
  schema)
    /wait && mysql --host=mysql -uroot -ppwd < employees.sql
    echo "EMPLOYEES schema created!"
  ;;
  demo)
    /wait && mysql --host=mysql -uroot -ppwd < demo.sql
    echo "Could load demo data!"
  ;;
  *)
    exec "$@"
  ;;
esac
```

[Here](https://github.com/sabatinim/test-containers/tree/master/containers/client) you can see docker container configuration details.
- **maven**: is the container where my kotlin test code is executes.

All the CI pipeline is orchestrate from this bash script:

``` bash
#!/bin/bash
docker-compose up -d mysql
docker-compose build db_client
docker-compose run db_client schema
docker-compose run maven mvn --quiet -f /tmp clean install
docker-compose run maven mvn -f /tmp surefire-report:report -DshowSuccess=false
docker-compose down --remove-orphans
```
You can build the project directly executing it on your computer or we can use it to create a CI pipeline.

## Setting CI pipeline
I have all the pieces to build my CI on github using GH actions.
Under *.github/* folder I have this configuration:

``` yaml
name: docker-compose-actions-workflow
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: CI
        run: script/ci
      - name: DEMO
        run: script/demo
```
This means: on pushing, start a build that execute the ci and demo script.
Under [tab actions](https://github.com/sabatinim/test-containers/actions) I can see all the build with the output:

``` bash
[INFO] 
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.kata.testcontainers.infrastructure.DBLoadEmployeeTest
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.561 s - in com.kata.testcontainers.infrastructure.DBLoadEmployeeTest
[INFO] Running com.kata.testcontainers.infrastructure.FileLoadEmployeeTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.002 s - in com.kata.testcontainers.infrastructure.FileLoadEmployeeTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
containers ---
```
Demo output:

``` bash
######## Employees ########
First Name: Marco
Last Name: Sabatini
Email: EmailAddress(value=address@email.com)
BirthDay: DateOfBirth(day=5, month=3, year=1983)
....
```
Keep in mind that I could add validation, performance or quality gate steps (wooow!)

## Considerations
Having infrastructure under the project codebase helps to understand the overall ecosystem and cut the distance between dev and operation. In this situation I could deploy my container on every container cloud service (ex. ECS on AWS) and be quite sure the behaviour is the same I'm seeing during CI or testing demo.
Of course from an organisational point of view this also means having people that are able to develop code, choose and create the necessary infrastructure.
As developers we have not think about be only code IDE "users" or system engineer!

### References
- [Exercise Github repository](https://github.com/sabatinim/test-containers)
- [Testcontainers](https://www.testcontainers.org/)
- [Getting Started with Contract Tests](https://blog.thecodewhisperer.com/permalink/getting-started-with-contract-tests)


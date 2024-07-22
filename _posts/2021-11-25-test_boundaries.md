---
layout: custom
title: "Test Boundaries"
date: 2021-11-25
---

## Overview
Separating the infrastructure from the domain is a smart move to define clear system boundaries, especially when dealing
with components responsible for interacting with the external world, such as databases, external APIs, or web services.
In this exploration, we'll delve into the TestContainers technique for constructing a robust test suite that aligns with
these system boundaries.

According to the [TestContainers](https://www.testcontainers.org/) documentation, this tool simplifies various types of
tests, making them more manageable:

Data access layer integration tests: TestContainers allows us to use a containerized instance of databases like MySQL,
PostgreSQL, or any other DBMS. This makes it easy to test data access layer code, ensuring compatibility without the
need for intricate setups on developers' machines. The assurance that tests always begin with a known database state is
a significant benefit. Additionally, any containerizable database type can be utilized.

Application integration tests: This involves running your application in a transient test mode with dependencies such as
databases, message queues, or web servers. TestContainers facilitates this process, making it efficient and reliable.

UI/Acceptance tests: For automated UI tests, TestContainers provides containerized web browsers compatible with
Selenium. Each test can spawn a fresh browser instance, eliminating concerns about browser state, plugin variations, or
automated browser upgrades. Moreover, TestContainers records video sessions for each test, aiding in debugging and
analysis.

Now, our approach will be a bit different. Instead of using the TestContainers framework directly, we'll leverage Docker
along with some bash scripts for configuration. Let's dive into this pet code adventure!

## Kata
Regarding [my previous post](https://dev.to/maverick198/fp-architecture-2o5i), we want to write the DB access layer for
this function:

``` kotlin
loadEmployees: () -> Either<Error, Employees>
```

And hey, why not use Docker for testing the infrastructure? I've got these integration tests hanging around for the
FileLoadEmployee function:

``` kotlin
@Test
internal fun loadSomeEmployees()

@Test
internal fun employeeNotValidEmail()
```

No need to go into the nitty-gritty details of those tests—they're pretty straightforward.
Here's the plan: I want to repurpose them for the DB implementation, but let's not go down the road of duplicating code.
My trick? Creating an abstraction for these tests that can serve both the File and DB implementations.

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

So, here's the deal: I'll whip up two abstract methods—instance() and wrongInstance(). These are the brainiacs
responsible for conjuring up the function instances that handle loading employees. They'll serve up the goods for both
the happy path and those pesky corner cases.

Now, for the File access layer:

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

For DB access layer:

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

You know, this trick I'm pulling off is called
a [contract test](https://blog.thecodewhisperer.com/permalink/getting-started-with-contract-tests). It's like our secret
sauce when we need to lay down the law for specific behavior in our code. Especially handy when we've got different
implementations floating around.

So, what's cooking here? I'm defining the loadEmployees behavior between my domain code and the infrastructure code.
It's like setting the rules of engagement. And the cool part? I'm testing different implementations to make sure they
play nice. It's the go-to move if you want to throw in an in-memory implementation of an external 'port' and speed up
those acceptance tests. Quick and snappy!

## Docker && Docker Compose
Now it's time to move to the infrastructure part. I need a DBMS MySQL instance and a JVM maven runtime where my Kotlin
test code can run. These containers have to communicate with each other.
This is the docker-compose file configuration that I used:

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

- **MySQL**: DBMS
- **db_client**: is a MySQL client command line interface. It is responsible for creating schema and could be used also
  for other purposes (ex. load demo test data etc.).
  It uses an entrypoint docker to create a schema or load demo data and
  this [plugin](https://github.com/kassambara/docker-compose-wait-for-container) helps to wait for MySQL instance to be
  started.

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

[Here](https://github.com/sabatinim/test-containers/tree/master/containers/client) you can see docker container
configuration details.

- **maven**: is the container where my Kotlin test code is executed.

All the CI pipeline is orchestrated from this bash script:

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

So, here's the lowdown: when we hit that push button, it kicks off a build that runs the CI and demo scripts. Just a
smooth ride through the [GitHub Actions tab](https://github.com/sabatinim/test-containers/actions), where you can catch
all the action and check out the build outputs.

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

Keep in mind that I could add validation, performance, or quality gate steps.

## Considerations
Having the infrastructure right there in the project codebase? It's like having a backstage pass to the entire
ecosystem. No more long-distance vibes between dev and ops. Now, in my world, I can toss my container onto any cloud
service—ECS on AWS, anyone? And the best part? I'm pretty darn confident the behavior will be just as groovy as what I
see during CI or testing demos.

Now, here's the real talk from an organizational angle. Sure, having the infrastructure mingling with the code means
having folks who can sling code, pick the right tools, and craft the necessary infrastructure magic. We're not just code
jockeys; we're like the wizards who know their way around systems.

### References

- [Originally Posted on](https://sabatinim.github.io/blog/2021/11/25/test_boundaries)
- [Exercise Github repository](https://github.com/sabatinim/test-containers)
- [Testcontainers](https://www.testcontainers.org/)
- [Getting Started with Contract Tests](https://blog.thecodewhisperer.com/permalink/getting-started-with-contract-tests)
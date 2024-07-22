---
layout: custom
title: "Event storming, and then what?"
date: 2024-07-01
---
### References
- [Github Repo](https://github.com/sabatinim/mars-rover-ddd)
- [Event Storming](https://www.eventstorming.com/)
- [Ticino SW craft sessions](https://www.youtube.com/@ticinosoftwarecraft/streams)

### Overview
Some time ago, I read a fascinating book titled *Introducing EventStorming: An Act of Deliberate Collective Learning by Alberto Brandolini*.
This book, filled with concrete examples, discusses the event-storming technique for modeling the business processes underlying a digital product to be implemented through software.

What truly captivated me and sparked my curiosity about this methodology is its ability to model software architecture and reveal **organizational limitations** within a company. In this article, I will demonstrate with a concrete example how event-storming can effectively bridge the gap between business and software development.

### Mars Rover Kata
The exercise I used as a reference is the [mars rover Kata](https://kata-log.rocks/mars-rover-kata). I used Python to solve it.

Its requirements involve implementing software for a rover to receive commands for movement. The rover can move forward, rotate left and right, and has constraints related to the surface it can traverse and potential obstacles it might encounter.

Before starting to implement the software, I conducted an Event Storming session. Of course, this came with significant limitations: I was the only participant and not particularly experienced with the technique and this session was just to implement the software (there are several levels of event-storming abstraction we should consider).
I used basic elements to model a business process, including commands, events, aggregates, policies, and projections. The definitions of these components provided in the book are particularly enlightening:

- **Command**: A decision made by the user.
- **Aggregate**: Information necessary for making decisions.
- **Event**: A state transition mapped somewhere.
- **Projections**: Tools to support the decision-making process in the user's brain.
- **Policy**: Triggers that respond whenever something happens

Regarding the flow, we can see how these components interact with each other in the picture below:

![Components interactions](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3q2wblsaiyjkpoofgmcs.png)

Everything will start with a command that triggers actions on a specific aggregate. This will generate an event and by listening to the event we can continue to trigger commands using Policies or create a view using Projections.

Using these building blocks, I attempted to model the Mars Rover design Event Storming.

### Event Storming
Below, we can see the first iteration of the process.

![Event Storming](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dv0umogwgg7eia3ki0d2.png)

First, I thought the rover needed to be powered on and set with a starting point and direction. The aggregate that came to mind here is the *Mars Rover* itself.
Once somebody starts it, it will be in "Started" mode and ready to move. Next, the rover can receive commands to turn right, turn left, and move forward. Depending on the presence of obstacles, the rover can either continue moving or encounter obstacles.
According to the exercise, in case of an obstacle, the rover should "shut down." Thus, I used a **policy** to react to the "ObstacleFound" event with a command instructing the rover to shut down.

From the first iterations, I noticed how intuitive it was to think in terms of Commands, Events, Aggregates, and Policies. I also used Projections to create datasets for future analysis, which I will discuss during the implementation phase.

From a modeling perspective, this technique is extremely useful. I could present the process to any product owner or domain expert or even implement it directly with them (as described in the book).
I am confident that in a very short time, we could create a dictionary of common terms understandable to all stakeholders involved (both business and software). Now, let's move on to the implementation phase.

### Implementation
You can find the solution code [here](https://github.com/sabatinim/mars-rover-ddd/tree/main).

Upon opening the project, you'll find a package named **ddd**. In this package, I have included the basic elements described earlier:

``` python
class Command:
    pass

class Event:
    pass

@dataclasses.dataclass(frozen=True)
class AggregateId:
    value: str

    @classmethod
    def new(cls):
        return cls(value=str(uuid4()))

@dataclasses.dataclass
class Aggregate:
    id: AggregateId
    version: int

class CommandHandler:
    def handle(self, command: Command) -> Event:
        pass

class Policy:
    def apply(self, event) -> Command:
        pass

class Projection:
    def project(self, event):
        pass
```
I reused the same names to create a common base linking the modeling and implementation parts. I also implemented an in-memory repository responsible for loading and saving an Aggregate object and a very simple in-memory *command dispatcher*.

The *command dispatcher* receives a series of commands as input and applies command handlers, policies, and projections based on its construction.
For this exercise, the implementation is in-memory, but you could consider implementing it with remote queues.

Here is our command dispatcher:

```python
class InMemoryCommandDispatcher:
    def __init__(self,
                 command_handlers: Dict[Type[Command], CommandHandler],
                 projections: Dict[Type[Event], List[Projection]],
                 policies: Dict[Type[Event], List[Policy]]):
        self.command_handlers = command_handlers
        self.projections = projections
        self.policies = policies

        self.commands: List[Command] = []

    def submit(self, commands: List[Command]):
        for c in commands:
            self.commands.append(c)

    def run(self):
        while self.commands:
            command = self.commands.pop(0)
            print(f"[COMMAND] {command}")

            event: Event = self.command_handlers[type(command)].handle(command)

            if event:
                print(f"[EVENT] {event} generated")
            event_policies = self.policies.get(type(event), [])
            for policy in event_policies:
                new_command = policy.apply(event)
                if new_command:
                    self.commands.append(new_command)

            for projection in self.projections.get(type(event), []):
                projection.project(event)
```

At this point, I have all the necessary components to build the model defined during the Event Storming session. Therefore, I implemented the commands, events, aggregate, and command handlers, as shown in the function defined to build the process.

```python

def create_command_dispatcher(mars_rover_repo: MarsRoverRepository,
                              mars_rover_storage: List[MarsRoverId],
                              path_projection_storage: List[Dict],
                              obstacles_projection_storage: List[Dict]) -> InMemoryCommandDispatcher:
    turn_right_command_handler = TurnRightCommandHandler(repo=mars_rover_repo)
    turn_left_command_handler = TurnLeftCommandHandler(repo=mars_rover_repo)
    move_command_handler = MoveCommandHandler(repo=mars_rover_repo)
    start_command_handler = StartMarsRoverCommandHandler(repo=mars_rover_repo)
    turn_off_command_handler = TurnOffCommandHandler(repo=mars_rover_repo)
    notify_obstacle_command_handler = NotifyObstacleCommandHandler()

    rover_path_projection = MarsRoverPathProjection(repo=mars_rover_repo, storage=path_projection_storage)
    rover_start_projection = MarsRoverStartProjection(repo=mars_rover_repo,
                                                      paths_storage=path_projection_storage,
                                                      mars_rover_storage=mars_rover_storage)
    rover_obstacles_projection = MarsRoverObstaclesProjection(storage=obstacles_projection_storage)

    obstacle_found_policy = NotifyObstacleFoundPolicy()
    turn_off_policy = TurnOffPolicy()

    return (InMemoryCommandDispatcherBuilder()
            .with_command_handler(TurnRight, turn_right_command_handler)
            .with_command_handler(TurnLeft, turn_left_command_handler)
            .with_command_handler(Move, move_command_handler)
            .with_command_handler(StartMarsRover, start_command_handler)
            .with_command_handler(TurnOff, turn_off_command_handler)
            .with_command_handler(NotifyObstacle, notify_obstacle_command_handler)
            .with_projection(MarsRoverStarted, rover_start_projection)
            .with_projection(MarsRoverMoved, rover_path_projection)
            .with_projection(ObstacleFound, rover_obstacles_projection)
            .with_policy(ObstacleFound, obstacle_found_policy)
            .with_policy(ObstacleFound, turn_off_policy)
            .build())
```
As you can see, this function creates the dispatcher by configuring the process modeled during the Event Storming session. Specifically, it associates commands with their respective command handlers, as well as policies and projections. It is straightforward to understand the actions associated with commands and events.

#### Command Handlers
Let's take a look at how I manage a command using a **command handler** for controlling the Rover's movement.

```python
class MoveCommandHandler(CommandHandler):
    def __init__(self, repo: MarsRoverRepository):
        self.repo = repo

    def handle(self, command: Move) -> MarsRoverMoved:
        mars_rover: MarsRover = self.repo.get_by_id(command.id)
        event = mars_rover.move()
        self.repo.save(mars_rover)
        return event
```
The class uses the repository to load the MarsRover Aggregate into memory and then calls the **"move()"** function to change its state.
Following this, the event is emitted and the aggregate's state is saved

#### Domain
Regarding the domain part, the figure below shows the objects used to model it.

![Domain Model](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/reofsorasx8mk59m80ud.png)

Let me show also the MarsRover **Aggregate** methods contract:

```python
@dataclasses.dataclass
class MarsRover(Aggregate):
    actual_point: Point
    direction: Direction
    world: World
    status: MarsRoverStatus

    def start(self) -> MarsRoverStarted:
        ...

    def turn_off(self) -> MarsRoverTurnedOff:
        ...

    def turn_right(self) -> MarsRoverMoved | None:
        ...

    def turn_left(self) -> MarsRoverMoved | None:
        ...

    def move(self) -> MarsRoverMoved | ObstacleFound | None:
        ...
```
It contains all the possible actions the Rover can do and the logic to change its internal state, starting from the actual point and direction to the grid/world in which the rover is moving.

#### Services
To orchestrate everything, I implemented a service.
Below you can see the service being used in the end-to-end tests developed:

```python
class TestE2E(unittest.TestCase):
    def test_execute_some_commands(self):
        repo = MarsRoverRepository()
        paths = []
        obstacles = []
        mars_rover_ids = []

        runner = (
            MarsRoverRunner(repository=repo,
                            path_projection_storage=paths,
                            obstacles_projection_storage=obstacles,
                            mars_rover_projection_storage=mars_rover_ids)
            .with_initial_point(x=0, y=0)
            .with_initial_direction(direction=Direction.NORTH)
            .with_world(world_dimension=(4, 4),
                        obstacles=[])
        )
        runner.start()

        id = mars_rover_ids[0]

        runner.run(id, "RMLMM")

        actual: MarsRover = repo.get_by_id(MarsRoverId(id))
        self.assertEqual("1:2:N", actual.coordinate())
        self.assertEqual("MOVING", actual.status.value)

        expected_path = ["0:0:N", "0:0:E", "1:0:E", "1:0:N", "1:1:N", "1:2:N"]
        self._assert_paths(expected=expected_path, actual=paths)

        self.assertListEqual([], obstacles)
```
In this test, I set the starting point, the grid on which the Rover moves, and its initial direction.
After that, the Rover is started calling **runner.start()** and receives commands in string format calling **runner.run(id, "RMLMM")**. I need to pass the rover id just to retrieve it using the repository. I can retrieve the RoverId from a projection created when we started it before.

Regarding assertions, I used the datasets generated by the paths projections, which are also developed with an in-memory version to facilitate interactions.

Below there are the logs of Commands and Events generated during the flow:

```
_e2e.py::TestE2E::test_execute_some_commands 
[COMMAND] StartMarsRover(initial_point=Point(x=0, y=0), initial_direction=<Direction.NORTH: 'N'>, world=World(dimension=(4, 4), obstacles=Obstacles(points=[])))
[EVENT] MarsRoverStarted(id=MarsRoverId(value='aea5e69c-4f08-40db-bf44-097b5ae36380'))
[COMMAND] TurnRight(id=MarsRoverId(value='aea5e69c-4f08-40db-bf44-097b5ae36380'))
[EVENT] MarsRoverMoved(id=MarsRoverId(value='aea5e69c-4f08-40db-bf44-097b5ae36380'))
[COMMAND] Move(id=MarsRoverId(value='aea5e69c-4f08-40db-bf44-097b5ae36380'))
[EVENT] MarsRoverMoved(id=MarsRoverId(value='aea5e69c-4f08-40db-bf44-097b5ae36380'))
[COMMAND] TurnLeft(id=MarsRoverId(value='aea5e69c-4f08-40db-bf44-097b5ae36380'))
[EVENT] MarsRoverMoved(id=MarsRoverId(value='aea5e69c-4f08-40db-bf44-097b5ae36380'))
[COMMAND] Move(id=MarsRoverId(value='aea5e69c-4f08-40db-bf44-097b5ae36380'))
[EVENT] MarsRoverMoved(id=MarsRoverId(value='aea5e69c-4f08-40db-bf44-097b5ae36380'))
[COMMAND] Move(id=MarsRoverId(value='aea5e69c-4f08-40db-bf44-097b5ae36380'))
[EVENT] MarsRoverMoved(id=MarsRoverId(value='aea5e69c-4f08-40db-bf44-097b5ae36380'))
```

Basically the rover is getting as input this commands "RMLMM" and they are translated during the flow execution.


#### Path Projection
I used a paths projection in order to store the coordinates every time the rover is moved.
As we can see in the before session we are able to assert the entire Rover path:

```python
expected_path = ["0:0:N", "0:0:E", "1:0:E", "1:0:N", "1:1:N", "1:2:N"]
self._assert_paths(expected=expected_path, actual=paths)
```
Therefore, for this exercise I implemented an in memory projection that is able to store path data in a list of dictionary:

```python
class MarsRoverPathProjection(Projection):
    def __init__(self, repo: MarsRoverRepository, storage: List[Dict]):
        self.repo = repo
        self.storage = storage

    def project(self, event: MarsRoverMoved):
        mars_rover: MarsRover = self.repo.get_by_id(event.id)

        raw = {"id": mars_rover.id.value, "actual_point": mars_rover.coordinate()}

        self.storage.append(raw)
```
This class takes as input the event **MarsRoverMoved** because is triggered on this, loads the aggregate and is able to create a specific read model.
In this case the read model contains the paths traveled by the rover. The good thing of this design is that the projection is totally decoupled by the aggregate state change and could be easy bind on a specific event that represents the Aggregate state change.

#### Policy
At this point, when the Rover encounters an obstacle, it must automatically shut down.
To achieve this, I implemented a **shutdown Policy**. This Policy takes the **ObstacleFound** event as input and generates a shutdown event, which is then handled by its own *command handler*, as we saw earlier.
Below you can see the TurnOffPolicy implementation:
```python
class TurnOffPolicy(Policy):
    def apply(self, event: ObstacleFound) -> Command:
        return TurnOff(id=event.id)
```

### Additional requirement
I tried to push the design further by considering what would happen if I had an additional requirement, such as sending a notification when the Rover encounters an obstacle.
First, I reviewed the initial design event storming model and I integrated the notification feature, resulting in this new version:

![Event Storming Second Iteration](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nz5i3uoi36js6ps6y7xn.png)

As you can see from the diagram, all I had to do was define a new Policy based on the **ObstacleFound** event, which was responsible for creating the command to notify the obstacle found.

```python
class NotifyObstacleFoundPolicy(Policy):
    def apply(self, event: ObstacleFound) -> Command:
        return NotifyObstacle(message=f"Rover {event.id.value} hit obstacle")
```

After the creation of the command we will have the command handler that will manage the notification flow.
Regarding the Policy this is initialized within the builder of the command dispatcher, which effectively configures the process.
Here, I simply added the policy without needing to change or refactor existing code.
The test below is just to show how we can make an end to end test setting up obstacles:

```python
 def test_hit_obstacle(self):
        repo = MarsRoverRepository()
        paths = []
        obstacles = []
        mars_rover_ids = []

        runner = (
            MarsRoverRunner(repository=repo,
                            path_projection_storage=paths,
                            obstacles_projection_storage=obstacles,
                            mars_rover_projection_storage=mars_rover_ids)
            .with_initial_point(x=0, y=0)
            .with_initial_direction(direction=Direction.NORTH)
            .with_world(world_dimension=(4, 4),
                        obstacles=[(2, 2)])
        )

        runner.start()

        id = mars_rover_ids[0]

        runner.run(id, "RMMLMMMMMM")

        obstacles_found = [o["obstacle"] for o in obstacles]
        self.assertEqual([(2, 2)], obstacles_found)
```
It's easy setup obstacles and assert they are found checking the in memory obstacle projection storage. I created an obstacle projection in order to collect where is the obstacle found by the rover during the travel. It was easy because I had **ObstacleFound** event and I just needed to listen for this event and project it.

Below there are the logs of the Commands and Events generated during the test. As you can see during the travel the rover found an obstacle. After that it was turned off and the obstacle was notified.

```
test/test_e2e.py::TestE2E::test_hit_obstacle 
[COMMAND] StartMarsRover(initial_point=Point(x=0, y=0), initial_direction=<Direction.NORTH: 'N'>, world=World(dimension=(4, 4), obstacles=Obstacles(points=[Point(x=2, y=2)])))
[EVENT] MarsRoverStarted(id=MarsRoverId(value='649d914f-967c-41ae-b3ca-ffede9ca9e7d'))
[COMMAND] TurnRight(id=MarsRoverId(value='649d914f-967c-41ae-b3ca-ffede9ca9e7d'))
[EVENT] MarsRoverMoved(id=MarsRoverId(value='649d914f-967c-41ae-b3ca-ffede9ca9e7d'))
[COMMAND] Move(id=MarsRoverId(value='649d914f-967c-41ae-b3ca-ffede9ca9e7d'))
[EVENT] MarsRoverMoved(id=MarsRoverId(value='649d914f-967c-41ae-b3ca-ffede9ca9e7d'))
[COMMAND] Move(id=MarsRoverId(value='649d914f-967c-41ae-b3ca-ffede9ca9e7d'))
[EVENT] MarsRoverMoved(id=MarsRoverId(value='649d914f-967c-41ae-b3ca-ffede9ca9e7d'))
[COMMAND] TurnLeft(id=MarsRoverId(value='649d914f-967c-41ae-b3ca-ffede9ca9e7d'))
[EVENT] MarsRoverMoved(id=MarsRoverId(value='649d914f-967c-41ae-b3ca-ffede9ca9e7d'))
[COMMAND] Move(id=MarsRoverId(value='649d914f-967c-41ae-b3ca-ffede9ca9e7d'))
[EVENT] MarsRoverMoved(id=MarsRoverId(value='649d914f-967c-41ae-b3ca-ffede9ca9e7d'))
[COMMAND] Move(id=MarsRoverId(value='649d914f-967c-41ae-b3ca-ffede9ca9e7d'))
[EVENT] ObstacleFound(id=MarsRoverId(value='649d914f-967c-41ae-b3ca-ffede9ca9e7d'), coordinate=(2, 2))
[COMMAND] NotifyObstacle(message='Rover 649d914f-967c-41ae-b3ca-ffede9ca9e7d hit obstacle')
Rover 649d914f-967c-41ae-b3ca-ffede9ca9e7d hit obstacle
[EVENT] ObstacleNotified()
[COMMAND] TurnOff(id=MarsRoverId(value='649d914f-967c-41ae-b3ca-ffede9ca9e7d'))
[EVENT] MarsRoverTurnedOff(id=MarsRoverId(value='649d914f-967c-41ae-b3ca-ffede9ca9e7d'))
```



### Evolutionary Architecture

Regarding the notification, I did not proceed further in the exercise. This is a typical example of what happens every day in our work: we create something that may have follow-up actions if it proves valuable.

Interestingly, the model already decouples this new concept (the notification), which we could potentially develop into a dedicated product line, with its own aggregate. This could become an external system that reads the ObstacleFound event from a queue and generates the notifications.
If we need to scale up because the notification system requires product enhancements, we could create a dedicated team to handle this domain.

This example is just to illustrate how such a design approach not only helps evolve our architecture but also guides it to support product development and organizational changes.

Some time ago, we had [Nicola Moretto](https://www.linkedin.com/in/nicola-moretto-ba197040/) speak at our [meetup](https://www.meetup.com/ticino-software-craft/) about *product development consistency.*

![Architecture Business Organization](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vluwvvzuqd7eqvsdgmgt.png)

He showed us this slide, which I find very significant. He explained that *Architecture*, *Business*, and *Organization* should go hand in hand with product needs and must be easily modifiable to manage product increments.

How often do we encounter these situations?

- Implement features on legacy architectures.
- Implement features but can't do it end-to-end because we depend on other teams.
- Implement features whose business value is uncertain.

To address these types of situations, the *architect* must act as the point of contact between the business and the organization needed to support it, by implementing an architecture that is most easily adaptable to the problem at hand.
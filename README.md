# A simple Quarkus demo

## Useful IDE shortcuts

- ⌘⌥L to format

## Scaffolding the app
1. Go to https://code.quarkus.io
2. Just choose the top option, Rest Easy Reactive
3. Explain that Quarkus has a reactive core, and will not be doing any reactive programming
4. Copy the folder path from the Downloads folder
5. In a terminal run
```
idea <path>
```

## Touring the generated code

1. **MAKE THE FONT BIGGER**
2. Show the generated dockerfiles, because Quarkus is Kubernetes-native. 
3. Explain that actually we prefer jib because it's easier to keep it up to date, compared to a file which is generated once.

## Running the application 

1. Launch in dev mode
```
mvn quarkus:dev
```

1. Click ‘r’ to run tests

2. Visit http://localhost:8080, which will show a get static page
3. Navigate to the Dev UI
4. Show the configuration and continuous testing tabs

5. I didn't write this code, what even is my endpoint? 
6. Find the endpoint by viewing the Rest Easy extension panel's endpoint scores
7. Visit http://localhost:8080/hello. It should work!

## Live coding 

1. Can we make it more exciting? Add a queryparam in `GreetingResource`

```
public String hello(@QueryParam("name") String name) {
return "Hello " + name;
}
```
1. Visit http://localhost:8080/hello?name=yourname
2. If you forget the space first time, no worries! It shows the reload when fixed.

## Continuous testing 

1. We got so excited by the UI, we did not notice a test is failing. Fix it using

```
.body(containsString("Hello"));
```

## Adding persistence 

1. To add the persistence libraries, run
```
quarkus ext add hibernate-orm-panache jdbc-postgresql
```
1. Now do some persistence. Make a greeting:
```
@Entity
public class Greeting extends PanacheEntity {
String name;
}
   ```
2. This is using Panache, a layer on top of hibernate. `PanacheEntity` allows us to use the active record pattern and eliminate a lot of boilerplate.
3. Then change the `Get` method to persist it. Construct the entity, add `@Transactional` and persist.

We're aiming for the following:

```
@GET
@Transactional
@Produces(MediaType.TEXT_PLAIN)
public String hello(@QueryParam("name") String name) {
   Greeting greeting = new Greeting();
   greeting.name = name;
   greeting.persist();
   return "Hello " + name;
}
```


1. Is the data going in? Add a second method. It needs to be in two steps to avoid a cast.

```
@GET
@Path("names")
@Produces(MediaType.TEXT_PLAIN)
public String names() {
List<Greeting> greetings = Greeting.listAll();
   String names = greetings.stream().map(g-> g.name)
   .collect(Collectors.joining (", "));
   return "I've seen " + names;
}
```

## Dev services

1. This is being stored in a database, but we didn’t set one up.
2. Run `podman ps`
3. Kill the container running the database.
4. Quarkus is unhappy, but everything comes back if Quarkus is restarted.
5. Run `podman ps` again to show it’s back

### Initialising services 

What if we want the database not to be empty on a restart?


1. Let’s make it look like we have more friends.

2. Make a `src/main/resources/import.sql` file.
3. Populate it with the following:

```
INSERT INTO Greeting(id, name)
VALUES (nextval('Greeting_SEQ'), 'Alice');
INSERT INTO Greeting(id, name)
VALUES (nextval('Greeting_SEQ'), 'Bob');
```

(An IntelliJ live template called `sql` is handy for this.)

1. The data will update without a restart.
2. We can also go to the dev UI, and navigate to hibernate, persistence units, and then click on it, to get a create script.



## Building a native app 

To build a native application, we can run
```
quarkus build --native -DskipTests
```

Then use tab complete for `target/code-with-quarkus-1.0.0-SNAPSHOT-runner`

However, this would fail at runtime, because no database is configured.



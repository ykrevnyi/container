
**Service Container** is a tool for managing dependencies.  
Behind the scenes it performs **Dependency Injection** *(a fancy phrase that means: if you see dependencies - try to "inject" them into the module)*.

![alt Inversed highlevel overview][inversed-overview]


It acts like **central registry** in order to manage the components of the system and to act as a mediator whenever a module needs to load a dependency.  

Using **Service Container**, each dependency, instead of being hardcoded into the module is received from the outside. This means that the module can be configured to use any dependency and therefore can be reused in different contexts.  

Behind the scenes it performs *Dependency Injection* (a fancy phrase that means: if you see dependencies - try to "inject" them into the class).

The main advantage of this approach is an improved decoupling, especially for modules depending on stateful instances.



#### Pros and Cons
- Decouple module from a particular dependency instance
- Reuse each module with minimal effort
- Testing module that uses DI is also greatly simplified - we can easily provide mocked dependencies and test our modules
- In isoltion from the state of the reset of the system.


#### Did you know?  
> Important aspect is that dependency wiring responsibility is shifted from the bottom to the top of architecture. The idea is that high-level components are by nature less reusable than low-level components, and thats because the more we go up in the layers of an application the more a component becomes specific.  
> The conventional way to see an application architecture, where high-level components own their lower-level dependencies, can be inverted, so that the lower-level components depend only on an interface, while the ownership of defining the implementation of a dependency is given to the hight-level components.







[inversed-overview]: https://raw.githubusercontent.com/ykrevnyi/container/docs/docs/ioc_container-1.jpg "Highlevel overview"



Simple and easy service container.

## Introduction

Service container is a powerful tool for managing class dependencies and performing dependency injection. Dependency injection is a fancy phrase that essentially means this: class dependencies are "injected" into the class via the constructor or, in some cases, "setter" methods.

Lets take a simple exaple:

```javascript
class Order {
  
  /**
   * Create a new order instance.
   *
   * @param  {Object} Database Mongo implementation
   * @param  {Object} Mailer   Mailer implementation
   * @return {void}
   */
  constructor(Database, Mailer) {
    this.db = Database;
    this.mail = Mailer;
  }

  /**
   * Create new order.
   *
   * @return {void}
   */
  create() {
    // ..
  }

}
```

In this example, the Order class needs to store information in the database and send e-mails when a order is create. So, we will inject a service that is able to interact with database and send e-mails. Since the services are injected, we are able to easily swap them out with another implementations. We are also able to easily "mock", or create a dummy implementation of the database and mailer when testing our application.

A deep understanding of the service container is essential to building a powerful, large application.

## Binding
The most basic container method is `.bind`. Using this method you can *bind* class to the service container.

```javascript
class Mailer {}
app.bind('Mailer', Mailer);

app.make('Mailer') instanceof Mailer; // true
```

#### Singleton
You are able to bind a class that will be resolved only once using `singleton` method.
```javascript
class Mailer {}
app.singleton('MailerSingleton', Mailer);

app.make('MailerSingleton') === app.make('MailerSingleton'); // true
```

#### Binding instances
You can also bind existing object instance using `instance` method.
```javascript
class Mailer {}
let googleMailer = new Mailer('google');

app.instance('GoogleMailer', googleMailer);
```


#### Bind interface to implementation.
A very powerful feature of the service container is its ability to bind an interface to a given implementation. For example, let's assume we have an Database interface and a MongoDatabase implementation. Once we have coded our MongoDatabase implementation of this interface, we can register it with the service container like so:

```javascript
class MongoDatabase {}

app.bind('Database', MongoDatabase);
```

This tells the container that it should inject the MongoDatabase when a class needs an implementation of Database.

```javascript
Class Order {
  
  /**
   * Create a new order instance.
   *
   * @param  {Object} Database Mongo implementation
   * @return {void}
   */
  constructor(Database) {
    this.db = Database;
  }

}
```

This way if for some reasons `mongo` server go down - we can swap it with different implementation:

```javascript
app.bind('Database', MysqlDatabase);
```

#### Lets take a look at testing example:
```javascript
const app = new Container;

app.bind('Database', Database);
app.bind('Mailer', Mailer);
app.bind('Order', Order);

// Create real order and send emails
app.make('Order').create();
// Injecting database..
// Injecting mailer..
// Creating order..
// Create db record..
// Send emails..

// Lets fake/mock database and mailer
app.bind('Database', fakeDatabase);
app.bind('Mailer', fakeMailer);

app.make('Order').create();
// Injecting fake database..
// Injecting fake mailer..
// Creating order..
// Faking db creation..
// Faking mailers work..
```


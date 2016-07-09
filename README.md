
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
```javascript

In this example, the Order class needs to store information in the database and send e-mails when a order is create. So, we will inject a service that is able to interact with database and send e-mails. Since the services are injected, we are able to easily swap them out with another implementations. We are also able to easily "mock", or create a dummy implementation of the database and mailer when testing our application.

A deep understanding of the service container is essential to building a powerful, large application.

Lets take a look at testing example:
```javascript
let app = new Container;
app.bind('Database', database);
app.bind('Mailer', mailer);
app.bind('Order', Order);

app.make('Order').create();
// Injecting database..
// Injecting mailer..
// Creating order..
// Create db record..
// Send emails..

app.bind('Database', fakeDatabase);
app.bind('Mailer', fakeMailer);

app.make('Order').create();
// Injecting fake database..
// Injecting fake mailer..
// Creating order..
// Faking db creation..
// Faking mailers work..
```javascript

## Binding
The most basic container method is `.bind`. Using this method you can *bind* class to the service container.

```javascript
class Mailer {}
app.bind('Mailer', Mailer);

app.make('Mailer') instanceof Mailer; // true
```javascript

#### Singleton
You are able to bind a class that will be resolved only once using `singleton` method.
```javascript
class Mailer {}
app.singleton('MailerSingleton', Mailer);

app.make('MailerSingleton') === app.make('MailerSingleton'); // true
```javascript

#### Binding instances
You can also bind existing object instance using `instance` method.
```javascript
class Mailer {}
let googleMailer = new Mailer('google');

app.instance('GoogleMailer', googleMailer);
```javascript


#### Bind interface to implementation.
A very powerful feature of the service container is its ability to bind an interface to a given implementation. For example, let's assume we have an Database interface and a MongoDatabase implementation. Once we have coded our MongoDatabase implementation of this interface, we can register it with the service container like so:

```javascript
class MongoDatabase {}

app.bind('Database', MongoDatabase);
```javascript

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
```javascript

This way if for some reasons `mongo` server go down - we can swap it with different implementation `app.bind('Database', MysqlDatabase);`.

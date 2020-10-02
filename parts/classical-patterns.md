# Classical Patterns

Angular is essentially is an object-oriented framework. This nature forces you to do most of your development in certain ways.

By focusing on the previously mentioned blocks (Components, Services, Pipes, etc...) ensure you are building a good architecture. This is similar to what the Laravel framework does in the PHP land, or Ruby on Rails for Ruby. The main objective of frameworks is to make your life easier and boost up development time.

This chapter will dive into the discussion of the following classical patterns:

- Components
- Singletons
- Observers

## Factory method

```typescript
class User {
  constructor(private fname: string, private lname: string) {}

  greeting() {
    console.log(`Hi, my fullname is ${this.fname} ${this.lname}`);
  }
}
```

During this part, we'll assume we have the User component showing above. This component has two private fields _fname_ and _lname_ associated with a _greeting_ method.

Most of the time, we get our users through a JSON API. So its shape is likely to be:

```json
[{ "fname": "Hassan", "lname": "Muhamad" }]
```

So it would be an easy task to convert the response we got into an object by the following snippet.

```typescript
let userFromAPI: User = JSON.parse(
  '[{ "fname": "Hassan", "lname": "Muhamad" }]'
)[0];
```

Typescript wouldn't fire any error or warning at the moment. But the moment you try to access the _greeting_ method, the problem will begin.
You'll get an error saying _TypeError: userFromAPI.greeting is not a function_

Sound weird, huh?

Let's break down what happened. We have specified the type of our variable at the very early stage by telling the Typescript compiler that it must be a User type. The reason no error was fired is that _JSON.parse( )_ method return _any_ type. So the conversion flow from _any_ to _User_ is possible. You'd end with an _Object_ instead of _User_. This situation highlights the fact that _Typescript types_ doesn't exist in _JavaScript_.

To solve this problem, we should duplicate the _map callback_ function every time we receive a JSON user.

Hopefully, The _Factory pattern_ is here to save us. A Factory is used for objects without exposing the instantiation logic to the client.

The example below shows an implementation of the pattern:

```typescript
export class UserFactory {
  /**
   * Build a User instance based on a JSON respone
   * @param {any} _json
   * @return {User}
   */
  static buildUser(_json: any): User {
    return new User(_json.fname, _json.lname);
  }
}
```

The example contains a _static_ method, named _buildUser_ that has a single parameter _\_json_. The method job is to extract values from the _\_json_ argument and use them to invoke the _User_ class constructor.

> All the methods of a Factory instance should be static.

A factory must encapsulate the creation of the dedicated object type. Nothing more.

## Observer

The Observer pattern job is to allow an object, called the subject, to keep track of other objects states which we call observers.
So if the subject state changes, every observer will be notified.

We need an example to clarify the mechanism.

PS: this example is written in pure TypeScript.

```typescript
interface Observer {
  notify();
}

export default Observer;
```

We define an interface of the observer we will implement.

```typescript
class ExampleObserver implements Observer {
  constructor(private name: string) {}

  notify() {
    console.log(`${this.name} notified.`);
  }
}

export default ExampleObserver;
```

Now, we need to implement the subject class. This class will manage the observers' list.

```typescript
class Subject {
  private observers: Array<Observer> = [];

  attachObserver(_observer: Observer): void {
    this.observers.push(_observer);
  }

  detachObserver(_observer: Observer): void {
    let index: number = this.observers.indexOf(_observer);

    if (index > -1) {
      this.observers.splice(index, 1);
    } else {
      throw "Unknown Observer";
    }
  }

  protected notifyObservers() {
    for (let i = 0; i < this.observers.length; ++i) {
      this.observers[i].notify();
    }
  }
}
```

Inside the example above, we have three methods:

- attachObserver: the method job is to push a new observer into the _observers_ field.

- detachObserver: remove an observer from the _observers_ list.

- notifyObservers: this method will iterate the _observers_ list and invokes their notify method.

The class below allows us to have an implementation of the mechanism.

```typescript
class MovieStore extends Subject {
  private moviesList: Array<string> = [];

  public addMovie(_movie: string) {
    this.moviesList.push(_movie);
    this.notifyObservers();
  }
}

export default MovieStore;
```

In order to make the whole mechanism function correctly, we need to create;
a Subject and an Observer.
Then, we need to attach the Observer instance to the Subject instance and try to change the Subject state by invoking the _addMovie_ method.

This can be done as showing in the code snippet below:

```typescript
let store: MovieStore = new MovieStore();

let myObserver: ExampleObserver = new ExampleObserver("sample observer");

store.attechObserver(myObserver);

store.addMovie("pulp fiction");
```

Run the code and go to your console. You should get "sample observer notified.".

### Unlock the full power of observable with TypeScript parameters

The implementation we saw is the basic implementation of the observer pattern. It has some weak points such as in order to know if something has changed we need to iterate over all the subjects and runs a test between its current state and its previous one.

A better approach is to modify the _notify_ method of the observer to get more details. As an example we could add optional parameters like the example shows:

```typescript
export interface Observer {
  notify(value?: any, subject?: Subject);
}

export class ExampleObserver implements Observer {
  constructor(private name: string) {}

  notify(value?: any, subject?: Subject) {
    consoloe.log(`${this.name} received ${value} from ${subject}`);
  }
}
```

This way the _notify_ method now accepts two optional parameters;

- value: which indicates the new state of the _subject_ instance.

- subject: a reference to the _Subject_ instance itself.

This would be helpful if we need at a certain stage to differentiate between subjects.

Another step is required. We need to change the Subject class and MovieStore a bit so they use the new _notify_ method.

```typescript
export class Subject {
  private observers: Array<Observer> = [];

  attachObserver(_observer: Observer): void {
    this.observers.push(_observer);
  }

  detachObserver(_observer: Observer): void {
    let index: number = this.observers.indexOf(_observer);

    if (index > -1) {
      this.observers.splice(index, 1);
    } else {
      throw "Unknown observer";
    }
  }

  protected notifyObservers (value? any): void {
    for (let i = 0; i < this.observers.length; ++i) {
      this.observers[i].notify(value, this);
    }
  }
}

export class MovieStore extends Subject {
  private movieList: Array<string> = [];

  public addMovie (_movie: string) {
    this.movieList.push(_movie);
    this.notifyObservers(_movie);
  }
}
```

This implementation is more expressive than the previous one.

Using this mechanism — _observer_ pattern for asynchronous programming — we ask for something, and we don't need to wait during its processing. Instead, we do a subscribe to the response event, so we get notified when the response comes.

This pattern and mechanism are widely used in Angular. So it's a good point to understand it.

## Promises

In addition to the Observable, the promise is another useful asynchronous concept. It function the same way as the _Observer_:
process something and, asynchronously, notify the caller that an answer is available.

### When we shall use Observer and Promise?

There is one thing the _Observer_ pattern can allow, the _Promise_ can not. It is the ability to _unsubscrube_. So, if you are not willing to unsubscribe to an event, you are better off using _Promise_.

### Promise usage

A possible way to use _Promise_ is through the _fork/join paradigm_. indeed, it is possible to launch many processes (named fork) and wait for all the promises to complete before sending the aggregated result to the caller (named join).

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

During this part, we'll asume we have the User component showing above. This component have two private fields _fname_ and _lname_ associated with a _greeting_ method.

Most of time, we get our users through a JSON API. So the its shape is likely to like this example.

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

Let's break down what happened. We have specify the type of our variable at the very early stage by telling Typescript compiler that it must be a User type. The reason no error was fired is that _JSON.parse( )_ method return _any_ type. So the conversion flow from _any_ to _User_ is possible. You'd end with an _Object_ instead of _User_. This situation highlight the fact that _Typescript types_ doesn't exist in _JavaScript_.

To solve this problem, we should duplicat the _map callback_ function every time we receive a JSON user.

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

The example contains a _static_ method, named _buildUser_ that have a single parameter _\_json_. The method job is to extract values from the _\_json_ argument and use them to invoke the _User_ class constructor.

> All the methods of a Factory instance should be static.

A factory must encapsulate the creation of the dedicated object type. Nothing more.

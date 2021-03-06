# Architectural overview

At this stage, we will dive into the main blocks that construct an Angular application such as:

- Component
- Service
- Directive
- Pipes
- Template

It a plus if you have some experience with Angular — which probably is true — and you are willing to apply good practices and design patterns. That can not be reached without a clear understanding of these main blocks mentioned above.

A quick recap for the general architecture of Angular won't hurt. It will refresh your memory and make sure you have a solid architectural basis to build a pattern upon.

The figure below shows how the main building blocks interact with each other:

![High-level architecture](../figures/high-level-architecture.png)

## Component

A component represents the views of an Angular application. They are responsible for what, when, and how the user-interface element they describe should be displayed on the client screen.

A component is a simple class that defines the logic required which based on it, the view will be built.

## Template

In angular, there's a separation between the logic and the HTML which it manipulates, unlike React.
The HTML code related to a component is known as the template.
The relation between a component and a template is indicated by using metadata.
The metadata is some sort of a decoration pattern that tells Angular how to interpret and process the related component class.

```typescript
@Component({
  selector: "my-component", // the component tag
  templateUrl: "./my-component.html", // the template path
})
export class MyComponent {
  constructor() {}
}
```

## Services

At this cornerstone, we've reviewed half of the main blocks of each Angular application. Now we address the Services. A service in Angular is a class with a unique purpose. Its role is to provide a well-defined service to other parts of the application.

You need to understand that a component shall only be responsible for the user experience, no more. Anything else must be delegated to services.

How to use a service in a component?

It's simple! You need to use dependency injection. Dependency injection is the process that a requesting -- at this context is a component -- gets a fully formed instance of a requested class dynamically (service class).

There are two ways to use dependency injection with Angular. You either need to define a provider for the Service class, in "app.module.ts".

```typescript
import { SampleService } from "./path/to/service";

bootstrap(SampleComponent, [SampleService]);
```

Or alternatively, by defining a provider in the component annotations.

```typescript
import { SampleService } from "./path/to/service";

@Component ({
    // other fields...
    providers: [SampleService]
})
// SampleComponent class
```

What is the difference between the two approaches?

If you apply the first approach, then the same instance of the service will be served across every class that requests it.
However the second approach, an instance will be served to the component each time the component is instantiated.

Which one is the optimal option?

Well, it depends on what your components and services are meant to do.
Some services are meant to be used everywhere, others are specific to a particular part of the application.

An important point to note is that you need to pass the service as an argument in the component constructor.

```typescript
constructor (private sampleService: SampleService) {
    // your logic here...
}
```

## Directives

Directives essentially interacting with a template and their related component by reading property and event bindings.

The example below shows the usage of directive that adds styles to an element in the template.

consider we have the following component.

```typescript
import { Component } from "@angular/core";

@Component({
  selector: "example",
  templateUrl: "./example.component.html",
  styleUrls: ["example.styles.css"], // adding css styles to our component
})
export class ExampleComponent {
  constructor() {}
}
```

You must notify the new attribute in the component annotation. the "styleUrls" represent an array of strings. each string is a path to a CSS stylesheet. In our case we have a single CSS file, so the array length is 1.

"example.styles.css" will contain the following code.

```css
button.btn {
  margin: 2em;
  padding: 0.5em 1em;
  border: none;
  border-radius: 5px;
}
```

associated with the following component template.

```html
<div class="container">
  <button class="btn" ExampleDirective>Hover on me</button>
</div>
```

this the implementation of the directive we talked about.

```typescript
import { Directive, ElementRef, Input, HostListener } from "@angular/core";

@Directive({
  selector: "[ExampleDirective]",
})
export class ExampleDirective {
  @Input()
  bgColor: string = "#28df99"; // 🟢

  constructor(private element: ElementRef) {
    this.element.nativeElement.style.backgroundColor = this.bgColor;
  }

  private highlight(color: string) {
    this.element.nativeElement.style.backgroundColor = color;
  }

  @HostListener("mouseenter")
  mouseEnterHandler() {
    this.highlight("#fddb3a"); // 🟡
  }

  @HostListener("mouseleave")
  mouseLeaveHandler() {
    this.highlight("#ec0101"); // 🔴
  }
}
```

An essential step to make this work is to register the Directive in the specific module that contains our component. In our case, we will register the SampleDirective in the high-level module "app.module.ts". This makes it available to all components across the whole app. Remember, in other cases, you would register your directive within child modules.

```typescript
// [app.module.ts]

// other imports

import { SampleDirective } from "./path/to/SampleDirecitve";

@NgModule({
  // other attributes...
  declarations: [SampleDirective],
  // 🚨 if there are other elements in the declarations don't delete them.
})
export class AppModule {}
```

## Pipes

Pipes are very helpful. They allow us to create a class that takes any input and transforms into to the desired output.
Pipes in angular works the same way as Unix pipes programming paradigm. We mean that information can be passed from one process (pipe) to another in a chain.

The following is an example:

```typescript
@Pipe({
  name: "my-special-pipe",
})
export class MySpecialPipe implements PipeTransform {
  transform(value: string): string {
    return `🅰️ [${value}]`;
  }
}
```

Pipes uses the @pipe annotation that behaves the same way a component and directive decorators work. It provides the metadata related to the pipe class.
The special part is that we need to implement an interface provided by the Angular framework. _PipeTransform_ is the interface that defines a single method which is _transform(value: any, args?: any): any_.

As you notice, the signature of the _transform_ method indicates that any type can be accepted.

> pipes by default don't restrict a specific data type.

To use the pipe you need to call it as follow:

```html
<p>{{{ 'some text' | my-special-pipe }}}</p>
```

Remember to import the pipe class its relative module and register it in the "declarations" attribute.

Pipes can have parameters so its behavior can be customized.

```typescript
@Pipe({
  name: "custom-pipe",
})
export class CustomPipe implements PipeTransform {
  transform(value: string, numb: number): string {
    return `[${numb}]: ${value}`;
  }
}
```

```html
<p>{{{ 'some text' | my-special-pipe: { numb: 88 } }}}</p>
```

the first argument is set to the element we associate the pipe to. So, we need to specify the rest arguments as attributes of a JavaScript object.

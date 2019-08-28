# Front-end design patterns for a modular architecture
*The purpose of this document is to summarize best practices for developing a front-end architecture of modular, loosely coupled and highly composable building blocks, to create codebases as scalable, reusable, flexible and maintainable as possible.*

## Compositional software techniques
### Thinking with modules
To make our software composable we create components that are highly modular.

> Modularization is organization by function instead of type.

A module:

* performs a distinct and logically discrete function.
* is independent (all necessary information/context is passed as parameters/inputs) and contains everything necessary to execute only one aspect of the desired functionality.
* interacts with other modules through well-defined interfaces.
* is interchangeable. 

In practice, that means:

* Functions can’t refer this, they can’t refer anything other than the arguments they are given.
* Classes can't refer to methods or member variables of another class, they can't refer anything other than the inputs or injections they are given.

### Component composition
Do organize code into a declarative view hierarchy of loosely coupled components.

```html
<petrol-car>
	<engine></engine>
	<fuel-tank></fuel-tank>
</petrol-car>

<electric-car>
	<engine></engine>
	<battery></battery>
</electric-car>

<hybrid-car>
	<engine></engine>
	<battery></battery>
	<fuel-tank></fuel-tank>
</hybrid-car>
```
Avoid tightly coupled sub-typed hierarchies.

```typescript
export class Car {
	// implementation
}

export class PetrolCar extends Car {
	// implementation (+ Car implementation)
}

export class ElectricCar extends Car {
	// implementation (+ Car implementation)
}
```

**Avoid class inheritance** - Inheritance (or sybtyping) leads to inherent tight coupling, makes code resistant to change, and prevents composability.

**Favor composition** - Loose coupling helps combining and reusing simple components to build various complicated ones to satisfy specific user requirements.

> https://medium.com/humans-create-software/composition-over-inheritance-cb6f88070205


### Container and Presentation Components
Do divide components into **Container** and **Presentation** components to decouple and encapsulate shareable and reusable code.

Separating out concerns into individual components can be more code - but it is cleaner, more flexible code.

#### Container components
Container components (or “smart” components) know how to get data and work with services - they are the middle-man between the APIs & your app’s business logic and subcomponents.

Container components are often top-level feature components.

* contain UI logic.
* subscribe and dispatch to observable store services.
* pass parts of the state (transformed or unmodified) as immutable data to subcomponents.
* pass callbacks to subcomponents for interaction.
* are mostly domain-specific
* are usually not intended to be reusable, but can contain shared logic usable by multiple types of subcomponents.

#### Presentation components
Presentation components are pure (or 'dumb'), simple UI components, and their only goal is to render a template properly.

Presentation components are decoupled from the app’s business logic, have no clue about app’s state structure or any service. They define an interface of how to communicate with them via typed inputs and outputs.

That way, presentation components are completely independent from their runtime context (they don’t know where they’re used nor why), which makes them reusable..

* are only for presentation, no business logic.
* have no knowledge or dependencies to their surrounding components or store service, and only interact through @Input()/@Output() bindings. 
* never mutate @Input() values and have no side-effects (besides mutating the DOM).
* are usually intended to be reusable.
 
Make your component dumb whenever possible.

```HTML
<!--
Container component has shared logic for selecting, mapping and binding car data from store service, based on car type.
Subcomponents subscribe to relevant data via @Input() and present it.
-->
 
<car-container type="TeslaX">	
	<electric-car [charge]="charge">
	</electric-car>
</car-container>
 
<car-container type="FordT">
	<petrol-car [fuel]="fuel">
	</petrol-car>
</car-container>
 
<car-container type="ToyotaPrius">
	<hybrid-car [fuel]="fuel" [charge]="charge">
	</hybrid-car>
</car-container>
 
<!-- Because of loose coupling we can also do e.g.: -->
	
<car-container type="ToyotaPrius">
	<engine [horsepower]="horsepower"></engine>
	<battery [charge]="charge"></battery>
	<fuel-tank [fuel]="fuel"></fuel-tank>
</car-container>
 
<car-container type="ToyotaPrius">
	<electric-car [charge]="charge">
	</electric-car>
	<petrol-car [fuel]="fuel">
	</petrol-car>
</car-container>
```

> How to write pure components: https://medium.com/@jtomaszewski/how-to-write-good-composable-and-pure-components-in-angular-2-1756945c0f5b

> Component Architecture: https://blog.angulartraining.com/component-architecture-with-angular-6f7bc9165443

> Feature and Presentation Components: https://coryrylan.com/blog/angular-design-patterns-feature-and-presentation-components

## State management patterns
### Thinking in Reactive
Do use a Declarative pattern for retrieving data.

Do create data streams of anything. Streams are cheap and ubiquitous. Anything can be a stream: variables, user inputs, properties, caches, data structures, etc. 

Reactive Programming raises the level of abstraction of your code so you can focus on the interdependence of events that define the business logic, rather than having to constantly fiddle with a large amount of implementation details.

 

> Introduction to Reactive Programming: https://gist.github.com/staltz/868e7e9bc2a7b8c1f754

> Patterns of managing state: https://medium.com/@alialhaddad/https-medium-com-alialhaddad-redux-vs-parent-to-child-2583c8e29509

You are able to create data streams of anything: variables, user inputs, properties, caches, data structures, etc.
On top of that, you are given an amazing toolbox of functions to combine, create and filter any of those streams.
A stream can be used as an input to another one. Even multiple streams can be used as inputs to another stream.
You can merge two streams.
You can filter a stream to get another one that has only those events you are interested in.
You can map data values from one stream to another new one.


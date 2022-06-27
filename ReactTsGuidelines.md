# React with TypeScript Coding Guidelines for Unigow projects

## Why Coding Guidelines?

The aims of this set of guidelines center around **managing the error-proneness**
of the language and common practices of it. Aswell as
**providing a common framework for programmers** to work and reason about the codebase.
With the hope of making projects' codebase easier to manage, easier to interact with,
and increasing the productivity of the project members.
While imposing reasonable restriction.

These guidelines are a **constant work in progress** and any rule may be changed,
dropped, or added as the project members see fit. Of course, given proper justification
(and alternatives if needed).

Note: **this does not, and does not attempt to, replace other best practices** like
testing and static analysis. And rather boosts the efficiency of other practices
like code-reviews and peer-programming by giving a set of bullet points to
look-out for when interacting with the codebase.

## Table of Contents

Section
--- |
[Components](#components)
[Type System with React](#type-system-with-react)
[Naming](#naming)
[Formatting](#formatting)
[Comments and JSDoc](#comments-and-jsdoc)
[References](#references)

## Goals of the Coding Guidelines

- **The presence of a rule must outweigh its absence**: It's benefit must be large
enough to justify its usage. Every rule must have a reason to be. The benefit
is measured with the problems that would arise with its absence.

- **Optimizing for the reader, not the writer**: The code is read fare more
often than it is written. It is preferable to enhance the reading, maintaining,
and debugging experience.

- **Avoiding dangerous practices**: As mentioned, it is easy to misuse
JavaScript and its features, ending up with security, safety,
and overall unpleasent bugs worth ours of work. This guidelines try to push for
type and memory safety practices, among other "good" practices.

- **Optimizing when necessary**: Performance optimizations can sometimes be
necessary and appropriate, even when they conflict with other principles of
this document. Justifications must accompany this optimizations (including measurements
of the obtained benefit/s).

## TypeScript guidelines

Every rule mentioned here is an extension over the already written
[TypeScript guidelines](https://github.com/juarez-gonza/tsguideline/blob/master/TsGuidelines.md).

If something:

- is not mentioned in this guideline, the ts guideline takes over.
- is mentioned in both guidelines, this guideline takes over since it is specific
for React projects.
- is a problem not mentioned in any guideline but should be covered, notify it ;).

## Components

React is a component library. Components are the building blocks of a
react project. The framework eases composability and reusability. But
care must be taken with the approach used to manage the component's tree
and state across it. It's quite easy to end up with hard to read/maintain code.

### Class or functions

Use functional components.

Declare functional components with the `function` keyword. Don't
use a variable assigned to a function expression or to an arrow.

### Component Composition and HoC

One recurring task of a programming in a React project is the one of managing state.
State across components can sometimes get tricky and difficult to manage.

Sometimes lifting state to an upper component can be an acceptable solution in
order to pass data around components. But most often than not it tends to
highly couple components with each other.

A good amount of flexibility can be achieved with [higher order components](https://www.robinwieruch.de/react-higher-order-components/)
(HoC) and other patterns that leverage this and other functional concepts to
deal with component composition and data passing in a clean way. It is customary
in React to name every HoC using the `with` keyword, such as `withLoading` or `withCondition`.
So **every HoC must be prefixed by `with` word**

Some other patterns are:

- [Container Pattern](https://www.jondjones.com/frontend/react/design-patterns/container-pattern-react-design-patterns-explained/):
separation of concerns between logic and rendering.
- [Render Props](https://www.robinwieruch.de/react-render-props/): commonly used
as an alternative to lifting state.

### State in Components

[Component composition](#component-composition-and-hoc) provides patterns
to cleanly manage data passing across components. HoC can sometimes compose
too well, and meaning of the program may stop being so simple to follow. For
many cases using a *carefully thought and contained* stateful solution can
lead to a "cleaner" solution.

### Conditional Rendering

Conditional rendering is a very common task. Sometimes a component my render
a child, sometimes due to a certain condition it may not.

Use `withCondition` HoC to conditionally render a single element.
MaybeComponent is an HoC that will take a react component that will
be conditionally displayed as an argument.
`withCondition` returns the conditionally rendered component wrapped
as a new component that takes a predicate `doRender` of type `boolean` and
the `props` of the original container.

```TypeScript
function Hello(props: {title: string}) {
  return <h1>{props.title}</h1>
}

const MaybeHello = withCondition(Hello);

function App() {
  return (
    <div className="App">
      <MaybeHello title={"Hello"} doRender={true}/>
    </div>
  );
}
```

NOTE: a similar component named withLoading could be written where
the returned embelished component by the HoC takes an `isLoading` param and
renders the loader if isLoading is true, the original component otherwise.

`withCondition` component is shown as an example in [Generic Programming section](#generic-programming).

For multiple conditional renderings in react, use a javascript
function containing a `switch` case over different status (most likely managed
via `useState`). that returns the component to be rendered. Then call it from
inside JSX or save it to a variable that then goes to the JSX.

```TypeScript
function SomeComponent() {
  const [displayState, setDisplayState] = useState(displayStates.initialQuery);

  ... some code

  function render(displayState: displayStates) {
    switch (displayState) {
      case displayStates.initialQuery:
        return <SomeOtherComponent1/>
      case displayStates.fieldFilling:
        return <SomeOtherComponent2/>
      case displayStates.loading:
        return <SomeOtherComponent3/>
      default:
        throw new Error("This is unreachable code");
    }
  }

  return render(displayState);
}
```

There is arguably a cleaner way to do this kind of multiple conditional rendering.
That would be via constant objects (called enums in JS, not the same as
TypeScript enums). The problem with this approach comes when each
conditionally rendered element has different props. TypeScript type cohersion
can't help us there since the rendered component is known at runtime.

This approach eases prop passing, though it might not look as clean.
More on conditional rendering [here](https://www.robinwieruch.de/conditional-rendering-react/).

### useCallback and useMemo

Both of these hooks can sometimes be optimizations for functional components.
Use them as needed, but no more.

useCallback will avoid creating a function when its dependencies have
not changed (the array as second parameter).

useMemo will memoize the result of a function and only re-run the function
when its dependencies change (again, the array as second parameter).

[Article on usage and differences](https://medium.com/@jan.hesters/usecallback-vs-usememo-c23ad1dc60)

### Custom hooks

Only write them for reusability purposes. That is, when many components
are using the same effectful and/or stateful logic.

### Hook dependencies

**NEVER omit a dependency**. This can be reinforced by tooling, but still
it's important to make the statement. If a component works properly when
the array of dependencies of some hook/s is not complete, then it's a broken
component and must be fixed.

### Ordering of hooks

The ordering of hooks in a function component is as following:

1. useState
2. useReducer (or the equivalent library reducer hook, if any)
3. useContext (or the equivalent library context hook, if any)
4. useEffect
5. custom hooks
6. useCallback
7. useMemo

### Outside of Components

Feel free to put some code outside of React components. Yes components
are the most used part of a react project, clearly. But extracting
common functionality to a `common` folder or a `utility` is just
as good of a method to improve reusability as any weird pattern
done with components. Another reason could be that a function, while
only used inside of a component, can be taken out of it, because it
does not depend on the component itself (ANY pure function is an example
of this). For the former case, taking such function out of the component
but keeping it in the same file can be enough.

### Disallowed

- re-renders manipulating `window` object or any programatic
redirection aside from the react routing library being used.

## Type System with React

### Function typing

#### props and React.FC

Do not use `React.FC` for typing props of a function. Use interface instead.

`React.FC` is not typing only for arguments, it's the typing of a funciton.
Function types are hard to apply to functions defined with the `function` keyword.
And since React components already have a known return type (i.e:
`React.ReactElement | null`), the precise typing just includes the props.

`React.FC` assumes `children` props always. Using an `interface` requires
explicitly mention of children (of type `React.ReactNode`) on the interface.
This extension of interface with children will probably be repetitive, so
it is going to be extracted into it's own interface extension.

There are some other problems with `React.FC`, that caused it to even
be removed from the TypeScript template of *create-react-app*.

```TypeScript
import React from "react";

interface SomeProps {
  a: number;
  b: string;
};

// BAD: React.FC imposes use of variable assigned to anonymous function
// React.FC always accepts children props, the caller could call this
// with children that will never be used

const Foo0: React.FC<SomeProps> = ({a, b}) => {
  ...
}

// GOOD

function Foo1({a, b}: SomeProps) {
  ...
}

// Now with children, using the WithChildren type extension

interface SomeProps {
  a: number;
  b: string;
}

function Foo2({a, b, children}: WithChildren<SomeProps>) {
  ...
}
```

where WithChildren type extension is defined as

```TypeScript
type WithChildren<T = {}> = T & { children?: React.ReactNode }
```

#### Function return-type

Do not type the return type of a component, it is known since react imposes
it in order to run.

Do use typing on the return type of other functions. It helps preventing
unexpected behaviour after future changes.

### Generic programming

Do not abuse generic programming.

Though sometimes it is a really useful tool for reusable components. HoC most
likely use generic programming.

Here is an example of the withCondition [HoC](#component-composition-and-hoc)
mentioned in [conditional rendering section](#conditional-rendering)

```TypeScript
/**
 * @type {T} - component props type
 * @param {React.ComponentType<T>} Component
 */
function withCondition<T>(Component: React.ComponentType<T>) {
  return function (props: Omit<T, "doRender"> & { doRender: boolean }) {
    const {doRender, ...childProps} = props;
    return doRender ? return <Component {...(childProps as unknown as T)}/> : null;
    // explanation for double cast: TS lost info of childProps as type T
    // after destructuring props parameter.
  }
}
```

More on utility types like `Omit` in [here](https://isamatov.com/typescript-utility-types-for-react/)

## Formatting

## Comments and JSDoc

## References

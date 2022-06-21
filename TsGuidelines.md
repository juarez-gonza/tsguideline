# TypeScript Coding Guidelines for Unigow projects

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
[`import`/`export` and Source File Structure](#importexport-and-source-file-structure)
[Type System](#type-system)
[JavaScript Features](#javascript-features)
[Asynchronous Programming](#asynchronous-programming)
[Exceptions](#exceptions)
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

## `import`/`export` and Source File Structure

### Organize by Feature

Organize packages by feature, not by type. For example, an online shop
should have packages named `products`, `checkout`, `backend`, not `views`,
`models`, and `controllers`. *Project structure should communicate purpose*.

### Imports

#### Import paths

Use ES6 `import` keyword for importing modules.

##### Import order

Every TypeScript file should import modules in the following order.

- Third party modules (similar modules, correlated modules in consecutive lines).
- Project modules whose path is outside of current file folder.
- Modules in the current file folder.
- Modules nested below the current file folder.

##### Relative or absolute

imports of the current project in javascript can be done as absolute paths
(`src/something/project1.ts`) or with relative paths (`../something/project1.ts`).
**Use relative paths**. *'But paths get too long :('*, if paths get to long maybe
that's a signal that the module is missplaced and perhaps it
does not correspond to that folder.

#### Import Types

- `import * as abc from '...'`. Generally acceptable option, allows for namespacing.
- `import {A, B, C} from '...'`. Generally acceptable option, allows short name for
commonly used stuff.
- `import A from '...'`. Only if external (third party) code requires it.
- `import '...'`. Use when only the side-effects of importing are needed

```TypeScript
///// file1.ts /////

import * as parser from '../parser.ts'; // allows for namespacing

function isRecognized(pattern: string, source: string) {
  // see match in this function has no conflict with parser.match
  // in fact, one is a variable and the other one a function.
  const match: string | null = parser.match(pattern, source);
  return match !== null;
}

///// file2.ts /////

import {compose} from './functional.ts'; // easy way to reference a commonly
                                         // used name.

function duplicate(x: number) { return 2 * x; }
function consecutive(x: number) { return 1 + x; }
function squared(x: number) { return x ** 2; }

function operate(x: number) {
  // this would be a bit more tedious to write and read if compose was namespaced.
  compose(duplicate, compose(consecutive, squared))(x);
}
```

### Exports

#### Default exports

Do not use default exports. Default exports make it hard to track
usage of the exported object since it may change names across it's users.

```TypeScript
// BAD: foo is not named foo in neither of its clients, and now all clients
// have buggy code, but realising foo() is the buggy code will take longer
// than it should just for naming confusions.

///// file1.ts /////
export default foo() { ...some buggy stuff }

///// file2.ts /////
import bar from './file1.ts'

...200 lines of code

bar();

///// file3.ts /////
import f from './file1.ts'

f();
```

```TypeScript
// GOOD: we all know who foo is

///// file1.ts /////
export foo() { ...some buggy stuff }

///// file2.ts /////
import {foo} from './file1.ts'

...200 lines of code

foo();

///// file3.ts /////
import {foo} from './file1.ts'

foo();
```

#### API visibility

Export only what's needed. As little as possible. Reducing the API surface
of modules is a good thing.

#### Mutable exports

Do not export mutable objects. Debugging this will be hard, specially if there
are re-exports. Instead, export some accesor (aka: getter) that has access to
the mutable value.

```TypeScript
// BAD: mutable export
export let foo = 3;
window.setTimeout(() => foo = 4, 1000);

// GOOD: access through function to mutable state.
let bar = 3
window.setTimeout(() => bar = 4, 1000);
export function getBar() { return bar; };
// NOTE: returned bar is just a value so there is no problem with
// reference access. This becomes a concern if the mutable state
// is a referenced object (such as arrays and dictionaries in JavaScript).
```

#### Namespaces vs. Modules

Namespaces are disallowed unless needed for a declaration file for some
critical library that does not provide it.

#### Container Classes and Namespacing

Do not creat container classes with static methods/properties just for namespacing.
Use the [module import syntax for namespacing instead](#import-types)

```TypeScript
///// file1.ts /////

// BAD: class just for namespacing
export class Container {
  static FOO = 1;
  static bar() { return 1; }
}

///// file2.ts /////
// GOOD
export const FOO = 1;
export function bar() { return 1; }
```

### Extra stuff about imports and exports

#### `import/export type ...`

Just don't.

## Type System

### Type Inference

### Null vs Undefined

### `any` and `unknown`

### `interface` vs. `type`

### Function return-types

### Type-casting and Coercions

### Generics

### Indexables

## JavaScript Features

### Local variable declarations

### Array literals and Array manipulation

### Object literals and Object manipulation

### String literals

### Other literals

### Functions

### Control structures

### Equality Checks

#### null and undefined

### Disallowed

## Asynchronous Programming

### Callbacks vs. `try/catch` vs. `then/catch`

- As an asynchronous option, always prefer promises to callbacks.
- Use `try/catch` instead of `then/catch`.
- Use **callbacks only in third party APIs that DO NOT provide an alternative
using promises**.

### Promises

**Always call return for reject() and resolve()**. See [Exceptions in Promises](#exceptions-in-promises).

## Exceptions

Exceptions should be used whenever exceptional cases occur (duh).
Always throw `Error` or subclasses of such (i.e: don't throw strings or
weird stuff). **Always use `new` when constructing an error**.

```TypeScript
// GOOD: calls Error with new keyword
function nthFibonacci(n: number) {
  if (n < 1)
    throw new Error(`[Error @ nthFibonnaci]: n must be greater than 0. n = ${n}.`);
  // ... rest of the function
}

// BAD: calls Error without new keyword
function nthFibonacci1(n: number) {
  if (n < 1)
    throw Error(`[Error @ nthFibonnaci1]: n must be greater than 0. n = ${n}.`);
  // ... rest of the function
}

// BAD: does not throw an error
function nthFibonacci2(n: number) {
  if (n < 1)
    throw "[Error @ nthFibonnaci2]: n must be greater than 0. n = ${n}.";
  // ... rest of the function
}

// BAD: does not even throw
function nthFibonacci3(n: number) {
  if (n < 1)
    return undefined;
  // ... rest of the function
}
```

### Error Message Formatting

Having a consistent error message structure that offers as much information
as possible in a concise format, allows for quicker solution of problems and
better documentation overall.

The anatommy of an error message goes as following:

`throw new Error("[ErrorSubclass @ (moduleName.)?(className.)?functionName]:
Helpful error message and display of useful values as needed.")`.

See code examples of [this section](#exceptions) to check the format in practice.

where:

- *ErrorSubclass*: is the name of the error class being used. Ex.:
  - *Error*
  - *NotFoundException*
- *(moduleName.)?(className.)?functionName*: is a string with the name of the function
for the given error. The prefix refers to the possibility of prepending the
name of the module and/or class name where the function lies if name clashing
is a risk. Ex.:
  - *getAvailableSlots*
  - *SchedulerService.getAvailableSlots*
  - *scheduler.SchedulerService.getAvailableSlots*
- *Helpful error message and display of useful values*: Display a descriptive message
of what brought the error condition. And values of variables if needed for
better comprehension.

### Exceptions in Promises

Rejection values should also reject with an `Error` or subclass of `Error`.
**DO NOT forget returning the resolved or rejected promise**

```TypeScript
// GOOD: returns the rejection/resolution
// and the reject() parameter is an Error (or an Error subclass) instance.
function asyncStuff() {
  return new Promise((resolve, reject) => {
    const samples: number[] = [];

    // ...sample collection

    if (samples.length === 0)
      return reject(new Error("[Error @ asyncStuff]: Could not collect samples"));

    // ...some work with the samples

    return resolve(samples);
  });
}

// BAD: does not return the rejection, code keeps running.
function asyncStuff1() {
  return new Promise((resolve, reject) => {
    const samples: number[] = [];

    // ...sample collection

    if (samples.length === 0)
      reject(new Error("[Error @ asyncStuff1]: Could not collect samples"));

    // ...some work with the samples

    return resolve(samples);
  });
}

// BAD: does not reject an instance of Error (or an Error subclass).
function asyncStuff2() {
  return new Promise((resolve, reject) => {
    const samples: number[] = [];

    // ...sample collection

    if (samples.length === 0)
      return reject("[Error @ asyncStuff2]: Could not collect samples");

    // ...some work with the samples

    return resolve(samples);
  });
}
```

## Naming

Style | Language Feature
--- | ---
UpperCamelCase | class, interface, type, decorator, type parameters.
lowerCamelCase | variable, parameter, function, method, property, module alias.
CONSTANT_CASE | global constant values, enum values.

### Descriptive Naming

Code is read many more times than it is written, be clear. Do not use
ambiguous variable names.

- Exception: Variables that are used for less than 10 lines and are not
part of an exported API, can contain short names.

### Type Informaton in Names

Do not write type information in names:

- `ISomething` for an interface for something. Just `interface Something {...}`.
- `opt_` or similar for optional parameters. Use TypeScript
[type system](#type-system) for that.
- class private fields having a name starting with `_`, use `private`
keyword for that.
- Actually, don't prefix or suffix with `_` at all. For array destructuring
ignoring parameters, just add a comma `const [a,,c] = [1,2,3];
// value at idx 2 is ignored`.

## Formatting

## Comments and JSDoc

## References

- [Airbnb JS style guide](https://github.com/airbnb/javascript)
- [Google JS style guide](https://google.github.io/styleguide/jsguide.html)
- [Google TS style guide](https://google.github.io/styleguide/tsguide.html)
- [iTwin.js TS coding guidelines](https://www.itwinjs.org/learning/guidelines/typescript-coding-guidelines/)

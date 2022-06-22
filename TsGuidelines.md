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

Code can rely on TypeScript's type inference.

**Exception**: [function return types](#function-return-types).

```TypeScript
const x = 5; // GOOD: type inferred

const y: number = 11; // BAD: typing here does not add readability.
```

### `any` type

TS `any` is a super and subtype of all types. It allows dereferncing all properties.
In other words, `any` does not allow TypeScript to be TypeScript.

Unknown on the other hand. Just tells TypeScript that we do not know the
type of a value. But it does not allow assigning an unknown value or
assigning to an unknown value.

**Consider not using `any`**. Try, in this order, on of:

- Prividing a more specific type.
- Writing [generic code](#generics).
- Using unknown.
- Supress the warning and document why.

### `interface` vs. `type`

use `interface` whenever possible. sometimes `type` comes in handy.
there are [technical reasons to prefer `interface`](https://ncjamieson.com/prefer-interfaces/).

### Function return types

Always annotate return types even if TypeScript can infere them. This
helps you as a programmer in knowing what you are looking for (recommendation:
write the return type before writing the body of the function), and the
future modifier of the code to know what's the "endgoal" of the function.

Otherwise, inadvertently modifying return type will almost surely break its callers.

If you consider creating a whole interface for the return type of a function
is "too much", inline the return type as in
`function foo(): { message: string , codeStatus: number } { ... }`.

### Function Implementing Interface

If a function implements an interface, it is acceptable to declare it as
a `const` qualified variable assigned to an anonymous arrow function

```TypeScript
interface SortFunction {
  (source: number[]): number[];
}

const quickSort: SortFunction = (source) => { ... } // GOOD
const mergeSort: SortFunction = function (source) { ... } // BAD: not an arrow function
function insertionSort(source) { ... } // BAD: implements interface but does
                                       // declare itself as such
```

### Type-casting and Coercions

- Do not cast just to lie to the type system.
- Only typecast when you absolutely know that an object corresponds to a given
type. Put a small comment saying why you are so sure of your casting.
- use `as` syntax not `<>`. The latter does not play nicely sometimes.

### usage of `?:` or `| undefined`

Use optional `?:` syntax for optional properties or parameters instead of `|undefined`.

NOTE: For optional parameters try to always provide a default value. Always
position optional parameters after needed parameters.

```TypeScript
// BAD: uses `|undefined`
interface Client {
  name: string;
  surname: string;
  age: number | undefined;
}

// GOOD: uses TS optional syntax
interface Client {
  name: string;
  surname: string;
  age?: number;
}

// BAD: uses `|undefined`
function doSomething(x: number, y: number|undefined) { ... }

// GOOD: uses TS optional syntax
function doSomething(x: number, y?: number) { ... }

// BETTER: provides default value and optional parameter after needed parameter
function doSomething(x: number, y=1) { ... }
```

### enum types

Always use `enum` and not `const enum` *in exported enums*.
Both cannot be mutated. `const enum` is an optimization that require a
declaration file when exported. This prevents --isolatedModules option
sometimes required (for example.: by create-react-app).

For non exported enums, `const enum` is a nice optimization. But be sure
of what you are doing because it may be exported in the future.

### Indexables

Sometimes indexable objects with `[]` such as dictionaries are very useful.

Your options are:

- ES6 `Map` and `Set`. This is the preferable option.
- indexable types as `{[key: SomeType]: SomeOtherType }`. The second best option.

Record<Keys, ValueType>, allows constructing types with a statically defined set
of keys. But it's an option.

### Generics

Don't go crazy on generics. But feel free to use them when suitable.

## JavaScript Features

### Local variable declarations

#### `var` declarations

Don't use `var`. Reasoning: `var` declared variables are function-scoped
instead of block-scoped. `let` and `const` are block-scoped, use these instead.

```TypeScript
// BAD: uses function-scoped declarations
function foo() { // start of function block
  var something = "Hello ";

  if (true) { // start of if-statement block
    var something = "World!"; // something here refers to the same variable
                              // declared outside of the if-statement
  } // end of if-statement block

  console.log(something); // outputs "World!"
} // end of function block

// GOOD: uses block-scoped declarations
function bar() { // start of function block
  const something = "Hello ";

  if (true) { // start of if-statement block
    const something = "World!"; // something here refers to a different variable
                                // from the one declared outside of the if-statement
  } // end of if-statement block

  console.log(something); // outputs "Hello "
} // end of function block
```

#### `const` by default

Use `const` keyword for variable declarations by default. Immutable values
are much easier to reason about.

### Array literals and Array manipulation

#### Literal Array Syntax

Use literal syntax `[]` instead of `new Array()`.

#### Assignment and copying

Use `someArray.push(something)` instead of direct assignment
(`someArray[someArray.length] = something`).

Spread operator (`...`) for copying arrays instead of `Array.from()`.

#### Array.prototype methods

**Use these methods extensively** for array manipulation.

- **Always use return** statements in array methods callbacks (exception: `forEach`).
- Don't use iterator loops as `for ... of` and `for ... in`.
  - Use `map()`, `every()`, `filter()`, `find()`, etc. for iterations
  over arrays.
- Use `Object.keys()`, `Object.values()`, `Object.entries()` to produce arrays
from objects so you can iterate over objects.

```TypeScript
const numbers = [1, 2, 3, 4, 5];

// BAD: uses `for ... of` syntax for iterators
let sum = 0;
for (let num of numbers) {
  sum += num;
}

// GOOD: uses Array.prototype.forEach
let sum = 0;
numbers.forEach((num) => {
  sum += num;
});

// BETTER: clean and concise functional form
const sum = numbers.reduce((total, num) => total + num, 0);

// BAD: conventional loop
const increasedByOne = [];
for (let i = 0; i < numbers.length; i++) {
  increasedByOne.push(numbers[i] + 1);
}

// GOOD: uses Array.prototype.forEach
const increasedByOne = [];
numbers.forEach((num) => {
  increasedByOne.push(num + 1);
});

// BETTER: clean and concise functional form
const increasedByOne = numbers.map((num) => num + 1);
```

### Object literals and Object manipulation

#### Literal Object Syntax

Use literal syntax `{}` instaed of `new Object()`.

#### Dot notation, computed property names, quoted identifiers

- Use dot notation (`.`) for accessing properties. Do not use bracket notation (`[]`)
except for the special case where it is an [indexable]( #indexable )
- Use computed property names `{[nameComputation()]: value}` when creating objects
with dynamic property names. But minimize usage of such objects unless strictly needed.
- Use quoted identifiers for identifiers that would be unvalid otherwise. Do not
mix quoted and unquoted identifiers.

```TypeScript
///// Quoted and unquoted keys

// BAD: uses quoted identifiers unnecessarily
const myObj = {
  'key1': {
    'key1': 3
  }
};

// BAD: mixes quoted and unquoted
const myObj = {
  key1: 2,
  'key-2': 3
};

// GOOD: is consistent with necessarily quoted keys
const myObj = {
  'key-1': 2,
  'key-2': 3
};

///// Property name computation

// BAD: adds calculated key in two steps
const myObj = {
  key1: 2,
};
obj[nameComputation('something')] = 3;

// GOOD: adds calculated key using computed property name notation
const myObj = {
  key1: 2,
  [nameComputation('something')] = 3
}

// BETTER: does not use quoted identifiers
const myObj = {
  key1: { key1: 3 }
};

///// Property access

// BAD: uses [key] notation unnecessarily
const val = myObj['key1']['key1'];

// BETTER: uses dot notation
const val = myObj.key1.key2;
```

#### `Object.prototype.foo`

Do not use Object.prototype functions directly (such as `hasOwnProperty()`).
This may not only be obfuscated by own property names in the object itself but
can also complicate tasks for the compiler such as string literal renaming/obfuscation.

### String literals

- Use double quotes for strings.
- Long strings should not be separated across multiple lines. It makes
them harder to find by tooling.
- Use template strings with \`\` notation instead of concatenation.

### Other literals

Do not use other wrapper objects of primitive types (`Boolean`, `Number`, `String`,
`Symbol`) without `new` keyword. These are *NOT* the same type as their lowercased
counterparts `boolean`, `number`, `string`, etc. Prefer using the lowercased ones.

### Functions

#### Named functions vs function declarations

Use `function foo() { ... }` to declare named top-level functions instead of
`const foo = function() { ... }` or `const foo = () => { ... }`.

TypeScript disallows rebinding functions, so preventing overwriting a function declaration
by using `const` is unnecessary.

One exception of assigning a function expression to a variable is explained
['Function Implementing Interface' section](#function-implementing-interface).

#### Arrow functions

- Use arrow functions for anonymous functions such as *higher-order functions* (callbacks
and functions returned from functions) and immediatly invoked functions.

- Use arrow functions to avoid `this` keyword rebinding. If you *need* `this`
keyword rebinding, think twice about your code since that can get quite complicated
to understand in a rather near future.

- DO NOT use arrow functions as class/object properties.

```TypeScript
///// as higher-order function

num = [1, 2, 3];
duplicatedNum = num.map(n => n * 2); // arrow as callback

const duplicate = x => y => x * y; // arrow as function returned from function
// the above is equivalent to:
//  const duplicate = (x) => { return (y) => { return x * y; } }

///// to avoid 'this' rebinding

// BAD: uses function expression and behaves unexpectedly with 'this keyword'
class LazyNumToMultiply {
  constructor(private readonly x: number) {}

  multiply(y: number) {
    return function() { return this.x * y; };
  }
}

const partialMult = new LazyNumToMultiply(2);
const lazyMul = partialMult.multiply(3);
console.log(lazyMul()); // undefined behaviour. most likely an error.

// GOOD: uses arrow function to avoid 'this' rebinding
class LazyNumToMultiply {
  constructor(private readonly x: number) {}

  multiply(y: number) {
    return () => this.x * y;
  }
}

const partialMult = new LazyNumToMultiply(2);
const lazyMul = partialMult.multiply(3);
console.log(lazyMul()); // prints 6
```

### Class

#### `prototype` vs. `class`

Just use `class`.

#### `constructor()`

Constructors are optional. A default constructor which calls `super()` to the parent
class is provided.

Subclasses with a non-default constructor must call `super()` prior to any access
to the instance (ex.: via the `this` keyword).

**Never ommit `()` when calling a constructor**

#### Property visibility

- no `#private`, use TypeScript `private` modifier.
- use `readonly` for properties that are never reassigned outside of the constructor.
- use [TypeScript parameter properties](https://www.typescriptlang.org/docs/handbook/2/classes.html#parameter-properties)
if possible. This makes for concise class definitions.

#### Getters and setters

Do not use JS `get`/`set` as they cause unexpected side effects and are harder
to test, maintain, reason about. Use accesors named something like `getVal()`
and `setVal` if absolutely needed.

#### Static methods

- Where it does not interfere with readability, prefer module-local functions
over private static methods.
- Static methods should only be called on the base class itself. A subclass
**cannot** call a parent's class static method.

### Control structures

#### Iterations

Check [this section](#array-literals-and-array-manipulation)

#### `switch/case` statements

Use braces `{}` for each of the cases.
This together with `const` and `let` block-scope will ensure each case has
it's own lexical declarations.

#### Ternary expresion

Ternary expressions must not be nested.

```TypeScript
// BAD
const foo = maybe1 > maybe2
  ? "bar"
  : value1 > value2 ? "baz" : null;

// GOOD: split into 2 separated ternary expressions
const maybeNull = value1 > value2 ? 'baz' : null;

const foo = maybe1 > maybe2 ? 'bar' : maybeNull;
```

### Equality Checks

**Always use `===` and `!==`**, the correspondent two symbol alternatives (`==`
and `!=`) use type coercion and that is not a desired behaviour
(exception: using `== null` to check both for `undefined` and `null`).

### null and undefined

Always use `null` for internal code.

### Disallowed

- `eval()`, it's unsafe.
- `with`
- relying on automatic semicolon insertion
- non-standard features
- modifying builtin objects

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

// BAD: exceptional case (argument precondition not accomplished) does not throw.
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
// value at index 1 is ignored`.

## Formatting

## Comments and JSDoc

## References

- [Airbnb JS style guide](https://github.com/airbnb/javascript)
- [Google JS style guide](https://google.github.io/styleguide/jsguide.html)
- [Google TS style guide](https://google.github.io/styleguide/tsguide.html)
- [iTwin.js TS coding guidelines](https://www.itwinjs.org/learning/guidelines/typescript-coding-guidelines/)

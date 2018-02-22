[🏠](README.md) › Node.js Design patterns & best practices


# Design patterns

- [Principals](#principals)
- [Modules](#modules)
- [Revealing module pattern](#revealing-module-pattern)
- [Prefer ES6](#prefer-es6)
- [Prefer functional over classical](#prefer-functional-over-classical)
- [Promises for async control flow](#promises-for-async-control-flow)
- [Debugging](debugging)


## Principals  
General programming principals apply for writing modern, functional-esque Node.js code:  

- **Single responsibility**. Otherwise known as do one thing and do it well, it keeps functions composable & testable.Functions maybe doing too much if it is hard to name or it has too many dependencies or arguments.  
- **Optimise for reading not writing**. Good code should be self-documenting as opposed to relying too heavily on commenting (which can go stale).  
- **Less code means less bugs**. Common patterns are abstracted to [resources/utils](https://gitlab.grit.work/resources/utils).  
- **High cohesion low coupling**. Abstractions should be informed by the cohesiveness of the code and to reduce coupling.  
- **Avoid unnecessary dependencies**. Prefer simple functional utilities when sufficient.  
- **Late optizimation**. Don't anticipate future modifications/additions with abstractions; prefer comments to revisit code.  

### Colocation
Prefer colocating (inline code) over indirection when it does not make the code anymore DRY. Colocate:
  - Variable assignment with usage.  
  - function declaration (keeping in outer most scope) with usage (Don't split implementation across modules).  

**Further Reading**  
- [Straight-line code over functions - FunFunFunction ](https://www.youtube.com/watch?v=Bks59AaHe1c)
- [React core developers on colocation; unit tests colocated with source ](https://twitter.com/dan_abramov/status/762658867327172608?lang=en)


## Modules  
- Modules are cheap, they can never be too small but they can be too large.  
- Modules should be highly cohesive and decoupled. They should do one thing and do it well.  
- Leverage npm as the largest package management system; don't try and compete with OSS where a package has real world usage in the wild and community contributions.
- Dependencies can increase deployment time, so limit them where possible, or only install dependencies that are stand alone packages that do not introduce [transitive dependencies](http://blog.npmjs.org/post/145724408060/dealing-with-problematic-dependencies-in-a).
- Prefer es6 and later versions of Node over dependencies.  
- Seperate modules by features: The boundaries of responsibility for each module should be informed by the underlying domain (business rules), and not from a developer centric perspective such as MVC.
- Implementation should be isolated from dependencies through Dependency Injection using [Imperative Shell, Functional Core](nodejs-testing.md#imperative-shell-functional-core) paradigm.  
- Avoid requiring individual files from within a module. This violates the [Principle of Least Knowledge](https://en.wikipedia.org/wiki/Law_of_Demeter).
- Structure modules into sections where possible: constants, utility functions, promise handlers, api.
- Sort arguments alphabetically, declare functions in their order within promise chain.

### Bad
The `.js` extension in the require statement is a code smell.

```javascript
const bambi = require('animals/deer.js').create();
```

### Best
The animals module exposes an API with the giraffe property.

```javascript
const { deer } = require('animals');
const bambi = deer.create();
```

- Keep function signatures flexible; once you have more than 2 or 3 arguments use a configuration object instead.

### Bad
This function signature is rigid and will be hard to refactor.  

```javascript
const clients = require(gender, age, location, height);
```

### Best
Using the es6 object literal shorthand, we can provide a lightweight flexible object as a single argument.  

```javascript
const clients = require({ gender, age, location, height });
```

**Additional reading**  
[Igor Soarez, Anti-patterns in Node.js](https://www.youtube.com/watch?v=pGFQ02qtJ7w)


## Revealing Module Pattern
- Favor the Revealing Module Pattern, as it:
  - Defaults to private assignments
  - Functions are declared in their outer most scope (Avoid nesting!)
  - The return statement is self documenting API

### Bad
Adds cognitive overhead by requiring you to scroll through the business logic to map all the Object properties to the API.

```javascript

const Obj = {
  foo(){}
  bah(){}
}
return Obj;
```

### Best
Take advantage of ES6 object property shorthand by keeping the naming of your functions consistent with your API naming convention.

```javascript
const foo = () => {}
// api
return {
  foo,
};
```

**Further Reading**
- [Essential Javascript Design Patterns](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#revealingmodulepatternjavascript)


## Prefer ES6
ES6 allows javascript to be even more expressive and economical in its syntax.

### Destructuring

#### Bad
es5 longhand for assigning alias to object property.  

```javascript
var myObj = require('myObj');
var foo = myObj.foo;
var bah = myObj.bah;
```

#### Best
es6 shorthand much better!

```javascript
const { foo, bah } = require('myObj');
```

#### Fat arrows  
- Are lightweight with no `function`, implied `return` and optional indentation instead of braces.
- Share the lexical this with parent scope.  
- Downside: Fat arrow functions can’t be used as generators or recursion (They're anonymous).

#### Bad
es5 longhand for anonymous callback

```javascript
getUser('Grant', function(grant){
  return grant;
})
```

#### Best
es6 shorthand much better!

```javascript
getUser('Grant', grant => grant);
```

- Fat arrows enables functions to be chained expressions, so closures can appear more readable.

This is a closure. It returns a function that has access to the options variable.
```javascript
module.exports = options => (t, cb) =>
...
```

The chained fat arrows are the equivalent of
```javascript
module.exports = function(options){
  return function(t, cb){}
}
...
```

## Prefer functional over classical

- fp (functional programming) makes programming easier to reason about then classical/OOP (Object Orientated Programming).  

> Favor composition over inheritance

> <cite>Gang of Four (Design Patterns)</cite>

### Factory functions for complex Object creation  
- Use factory functions and not constructors/ES6 classes. The `new` keyword is a [code smell in javascript](https://medium.com/javascript-scene/javascript-factory-functions-vs-constructor-functions-vs-classes-2f22ceddf33e#.7c7g0zr1t).
- Prefer `const` (es6) to assign variables over `var`. Immutable data makes programs easier to reason about as they don't change over time.

### pure functions  
- Keep functions pure where possible. Functions that are referentially transparent (can be replaced with a value) are more testable and easier to understand than functions that operate as sub routines (Performing operations on data that is not the function's input).

### Composition over inheritance  
- Compose simple functions to construct more complex behavior.  
- Avoid nested conditions prefer composition and ternary operators. [flat code is healthy code](https://twitter.com/tjholowaychuk/status/753315132982251520).
- If a functions is hard to name, it could be that it is not adhering to the [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle).

### Avoid global state  
- Avoid global variables, keep variables local to functions. Prefer to trickle data down a promise chain than have promise handlers refer to variables outside of their scope.

### Avoid "this" and "new"  
- Avoid using "this". Javascript allows functions/methods to be invoked with dynamic context using `apply` and `call`. Likewise avoid new as it is problematic, prefer to create new objects through concatenation and factory functions.

### Unadorned data  
- Keep data in its most primitive format: arrays, numbers, strings etc. Do not adorn data with special methods for mutating i.e. sequalize models.

### Recursion > looping  
- `for in` and `for each` loops should be avoided in favor of higher order functions `map`, `reduce` and `filter`.


**additional reading**  
[Tangible Data](https://github.com/omcljs/om/wiki/Conceptual-overview#tangible-data)


## Promises for async control flow
- Promises have been widely adopted in the javascript community and are preferable to the verbose error handling of callbacks.  
- Prefer to keep implementation outside of long promise chains.
- A promise chain should be higher level, sequential control flow of named functions. Keep it declarative, describe what is happening but not how it happens (implementation).
- flatten exported promise chain where possible or reduce promise chains in promise handlers.


### Referentially transparent
- Promise handlers should be referentially transparent.
- Return a copy of the input data it receives (will often be a state object).  

```javascript
// objects
// add items to object
deepmerge(data, { newKey: 'newValue' });
// remove top level property from object
Object.keys(data).reduce((acc, key) =>
  (key === 'removeMe') ? removeKeys(acc, [key]), data) : data);
// arrays
// remove/mutate items in array
deepmerge(data, { arr: data.arr.filter() }, dontMergeArrays);
// add items to array
deepmerge(data, { arr: ['newItem'] });
```

### Unnecessary enveloping
- Remove unnecessary enveloping (don't return an object with single key, move key to value that is returned)

## good
```javascript
const process = data =>
  createCmd(data)
    .then(maybeAppendInputRecords)
    .then(execScud)
    .then(parseResult);

return {
  process,
};
```

# best
- less indirection
```javascript
return data => createCmd(data)
  .then(maybeAppendInputRecords)
  .then(execScud)
  .then(parseResult);
```

#### Bad
Promise chain interrupted with implementation details

```javascript
maybeCacheList(request.params.list)
  .then(value => {
    if (isCycleData(value)){
      return Promise.resolve(value);
    }
    else {
      return Promise.resolve(cycleData(value))
    }
  })
  .then(writeDomainsList)
```

#### Best
Promise chain of sequential named functions

```javascript
maybeCacheList(request.params.list)
  .then(maybeLogCycleData)
  .then(writeDomainsList)
  ...
```

### Complex branching  
Promise chains are less useful for complex branching. Here are two ways to handle basic forms of branching:

**1. Split into two chains based on ternary evaluation**  
```javascript
const initDevelop = () =>
  fetchOrLoad()
    .then(maybeProcessMineUpdate)
    .then(flattenData)
    .then(setEnvironment);

const initProd = () =>
  fetchData()
    .then(flattenData)
    .then(setEnvironment);

const init = (isDevelopment) ? initDevelop : initProd;
```

**2. Prefix optional promise handlers with maybe**  
```javascript
maybeFecthData()
  .then(maybeProcessMineUpdate)
  .then(flattenData)
  .then(setEnvironment);
```

### If using promises, have all implementation within promises.  
Place all implementation within promises for a single point of failure.  

#### Bad
Expression performed outside of promise, will not be handled by catch handler.  

```javascript
const myFn = () =>
  // if this line throws an error may terminate program & not handled by catch
  const secrets = JSON.parse(decrypt({ str, password }));
  return new Promise((resolve, reject) => {
    /* perform operation with secrets */
  })
  .catch(console.error);
```

#### Best
Abstract expression within a promise to be handled by catch handler.

```javascript
const getSecrets = str => new Promise((resolve, reject) => {
  try {
    resolve(JSON.parse(decrypt({ str, password })));
  } catch (e) {
    reject(e);
  }
});

const myFn = () =>
  getSecrets(key)
    .then(secrets => /* perform operation with secrets */ )
    .catch(console.error)
```

## Linting and Debugging
- Gritcode has a sharable eslint configuration on github.com [module-eslint-config-gritcode](https://github.com/gritcode/module-eslint-config-gritcode).  
- Prefer rawkit over `node --inspect`, copies chrome url // alias
colocate channel & username constants. Prefer env vars to locating values likely to change at runtime at top of files and avoid indirection if it doesn't meet one of these conditions:  
    1. Readability of being able to put a config object on a single line.  
    2. Keeping code dry for values used more than once.  
    3. Deconstructing assignment or constant assignments from global objects i.e. process.env


### Resources

- [rawkit](https://www.npmjs.com/package/rawkit)

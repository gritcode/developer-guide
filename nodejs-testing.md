[🏠](README.md) › [Node.js](nodejs.md) › Testing


# Testing

- [Conventions](#conventions)  
- [Imperative Shell, Functional Core](#imperative-shell-functional-core)
- [Test framework](#test-framework)
- [Common test cases](#common-test-cases)
- [Gotchas](#gotchas)


## Conventions

- Prefer a single assertion per test, not many assertions for a single test. This will make it easier to read failing tests.  
- Keep the name of the test descriptive and ensure you only describe what is actually being tested.  

## Imperative Shell, Functional Core  
- The Imperative Shell, Functional Core paradigm isolates business rules in a functional manner using dependency injection. It is breaks a module down into 3 separate concerns:  

1. `index.js` **Module Context.** The scope that the module belongs to, i.e. if the module functionality belongs to, or is a hapi plugin, this file will contain Hapi-centric implementation such as route configuration. This file requires the the shell, not the core directly.

2. `shell.js` **Dependencies** Provide dependencies to the core.js and returns the core api. It is a wrapper for the core with resolved dependencies and any options or settings passed in.  

3. `core.js` **Business logic** Isolated units of business logic. Dependencies are injected so there should be no require statements.


**additional reading**  
- [Unit testing: How to get your team started - FunFunFunction #2](https://www.youtube.com/watch?v=TWBDa5dqrl8)


## Test framework  
- Use [node-tap](http://www.node-tap.org) as a testing framework. The benefits are:
  - code coverage out of the box
  - A minimal API surface.
  - Maintained by NPM CEO.
  - Outputs a common, widely used protocol in TAP.
  - Compatible with node debugging tools like `devtool`, `denode` & nodejs native debugger module so you can set breakpoints while testing (impossible in AVA).
- Stick to the single assertion `t.true` where possible so no special knowledge of an assertion library is required.
- Stick to `t.end`, don't need to use `t.plan`.
- Keep tests lightweight, let tests fail through timing out no need to explicitly call `t.fail`. The only exception is for Promises, as they pass if returned without a resolved/rejected handler. See gotchas below.

### Bad  
```javascript
test('getNumber returns a number', t =>
  getNumber().then(value => t.true(typeof value === 'number')));
```

### Best
```javascript
test('getNumber returns a number', t =>
  getNumber().then(value => t.true(typeof value === 'number'))
  getNumber().catch(reason => t.fail()));
```

- When code is pushed to gitlab, tests are run based on the `gitlab-ci.yml` file.
- The simplest `gitlab-ci.yml` file looks like this:  

```yml
cache:
  paths:
  - node_modules/

stages:
  - test

unit_integration_tests:
  stage: test
  script:
  - npm install
  - npm test
```


**additional reading**  
https://medium.com/javascript-scene/why-i-use-tape-instead-of-mocha-so-should-you-6aa105d8eaf4#.ubnmyaiub


## Common test cases

**Passes test**  
```javascript
test('pass this', t => {
  t.pass();
  t.end();
});
```

**Throws error**  
```javascript
test('Throws error', t => {
  try {
    fnThrowsError()
  } catch (e) {
    t.pass();
    t.end();
  }
});
```

**Start and stop hapi server (basic)**  

```javascript
const Hapi = require('hapi');
const server = new Hapi.Server();
server.connection({ port: 3000 });

test('first test', t => {
  server.start()
    .then(() => {
      server.stop();
      t.pass();
      t.end();
    });
});
```

**Start and stop hapi server (configured)**  



**Returns Promise**  
```javascript
test('Promise value', t =>
  const isPromise = fnReturnsPromse();
  t.true(isFunction(isPromise.then));
}));
```

Promise resolves

**Joi validation error**  

```javascript
test('Joi validation error', t =>
  fnRejectsJoiValidation()
    .catch(reason =>
      t.true(reason.toString().indexOf('ValidationError') !== -1)));
```

### stubbing & spying with sinon

**spy**  
Spys provide insight into a functions behavior with altering it.    
```javascript
const keyringInit = sinon.spy(keyring, 'init');
process.kill(process.pid, 'SIGINT');
process.on('SIGINT', () => {
  t.true(keyringInit.calledOnce);
  keyring.init.restore();
});
```

**stubs**  
Stubs enhance spies by altering behavior; i.e. can change the return value of an method.  
```javascript
sinon.stub(kue, 'createQueue').yields({
  create() { return this; },
  save(cb) { return cb(null); },
});
```


## gotchas

**resolved promise expected to reject will still pass**  
node-tap will pass a test that returns either a Promise without a resolved/rejected handler. If you are only testing for a rejection and the promise resolves, you will get a false positive.

```javascript
test('resolved promise that is expected to reject will still pass test', t =>
  Promise.resolve('no then listener will pass')
    .catch(reason => {
      t.true(reason === 'Reason for rejection'); // We have
    }));
```

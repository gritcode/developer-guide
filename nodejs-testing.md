[🏠](README.md) › [Node.js](nodejs.md) › Testing


# Testing

- [Conventions](#conventions)  
  - [Tests](#tests)
    - [Fixtures](#fixture)
    - [Test Doubles](#tests-doubles)
  - [Modules](#modules)
    - [Imperative Shell, Functional Core](#imperative-shell-functional-core)
- [Usage](#usage)  
- [Test framework](#test-framework)
- [Test Types](#test-types)
  - [Unit](#unit)
  - [Integration](#integration)
- [Common test cases](#common-test-cases)
- [Gotchas](#gotchas)


## Conventions
The purpose of adhering to conventions is to ensure testable _modules_ & relevant, maintainable _tests_. Modules live within `/lib`; tests in the `/test` directory.

### Tests
- Test first or test later - as long as we have tests before submitting code for review.  
- There is a hard separation between tests and the modules they are testing; no code within a module should exist for testing purposes. Tests are for modules, not modules for tests.
- Prefer a single assertion per test, not many assertions for a single test. This will make it easier to read failing tests. If there is many assertions, each assertion should either test the same variable, properties of the same variable object, or different variables that are in some way related.  
- Keep the name of the test as a descriptive summary of the assertions being performed and ensure you only describe what is actually being tested.  

**Directory structure**  
```bash
└── test  
    ├── fixture (optional)  
    ├── integration  
    ├── stub (optional)  
    └── unit  
```

#### Fixtures
- Avoid complex fixtures, prefer plain javascript objects and simple values.
- Fixtures normally serve as input values to modules being tested.  
- `tap@optizmo.grit.ninja` is a special fixture value that can be used for integration tests when inspecting ses. See the [Common Test Cases](#common-test-cases) section for more information on inspecting ses.  

#### Test doubles
Test doubles are useful for isolating units from their dependencies but an over reliance on them can invite complexity & fragility to tests.  

> Mocking is a code smell. If you have to do a lot of mocking to create a proper unit test, maybe that code doesn’t need unit tests at all.  
<cite>_[Eric Elliott](https://medium.com/javascript-scene/5-common-misconceptions-about-tdd-unit-tests-863d5beb3ce9)_</cite>

Simple test doubles such as shallow stubs ensure mocking is kept in check.

**Shallow stubs**  

> Stubs provide canned answers to calls made during the test, usually not responding at all to anything outside what's programmed in for the test.  
<cite>_[Martin Fowler](https://martinfowler.com/articles/mocksArentStubs.html)_</cite>

Deliberately keep stubs shallow, we are representing the state of a dependency with a simple, hardcoded response.  
  - Input is ignored (as it requires implementation to process)
  - Output is a minimal representation, usually just the correct type with an empty value (i.e. resolve an empty string instead of random, computed or hardcoded string).  
  - No or minimal implementation; it is inherently brittle and frankly impossible to attempt to reliably substitute implementation of another module; it requires a familiarity of the internals of the other module and must stay up to date with version changes.  

**example**  
```javascript
// shallow stub for business module, resolve empty object
const businessModule = () => Promise.resolve({});
```

### Modules
Each module should follow the Imperative Shell, Functional Core paradigm;  isolating business rules in a functional manner with dependency injection.

#### Imperative Shell, Functional Core

A standard module down is composed of:

  1. `index.js` **Module Context** The scope that the module belongs to, i.e. if the module functionality belongs to, or is a hapi plugin, this file will contain Hapi-centric implementation such as route configuration. This file requires the the shell, not the core directly.

  2. `shell.js` **Dependencies** Provide dependencies to the core.js and returns the core api. It is a wrapper for the core with resolved dependencies and any options or settings passed in.  

  3. `core.js` **Business logic** Isolated units of business logic. Dependencies are injected so there should be no require statements or `new` keywords to instantiate objects.

**AWS endpoints**  

AWS Modules endpoints implement the imperative shell, functional core with a slightly modified module file structure. The transport layer is considered part of the business logic to be tested as it has its own dependencies and paths that need to be tested.  

  1. `index.js` **AWS Context** Exports an object of handlers, which are lazy evaluated functions injected from the shell. Lazy evaluation means handlers don't instantiate and require modules that belong to other handlers in the shell. It also pushes the aws specific arguments of `event, context, cb` to the edge of the module by accepting them in the `index.js`.  

  2. `shell.js` **Dependencies** Exports the handler functions to be invoked by the `index.js`

  3. `transport.js` **Business logic** The default transport file (normally the api consumed by the corresponding web application). An example of an additional transport would be an api consumed by the public. With many transports, we could then move them inside a transport folder: `/transport/index.js` and `transport/public.js`.

  4. `business.js` **Business logic** The AWS business.js is no different to a standard module.

**Additional Reading**  
- [Unit testing: How to get your team started - FunFunFunction #2](https://www.youtube.com/watch?v=TWBDa5dqrl8)


## Usage
- Automated tests are run in two environments: by the Gitlab CI server and should be run locally while developing.
- Both use command `npm test`, while `npm run watch` can be used in local development for automatically running unit tests on file changes.
- `npm test` uses sandbox account specific env vars found in the version controlled .env.default file.
- NOTE: `TESTING=true` env variable ensures replyInternalError prints to the console, useful for debugging build trace or in local terminal.  

package.json
```json
{
  "test": "TESTING=true node node_modules/.bin/dotenv -e .env.default tap --coverage-report=text-summary --timeout=120 --no-bail",
  "watch": "nodemon --exec npm run test test/unit --silent"
}
```

### Continuous Integration

- Each project should run its automated tests on the gitlab CI server.  
- Each project README.md will have [badges](https://docs.gitlab.com/ce/user/project/pipelines/settings.html#badges) to display the status.  
- Both unit and integration have a corresponding stage and job. Each uses its npm run script.  

**Example**  
```yml
# NOTE: missing node_modules cache etc
unit:
  stage: unit
  script: [npm test test/unit]
integration:
  stage: integration
  script: [npm test test/integration]
```


## Test Types

- Unit tests are scoped to a single unit and use simple test doubles (shallow stubs) for dependencies.
- Integration tests are scoped to combinations of units and their dependencies.

### Unit

> The primary goal of unit testing is to take the smallest piece of testable software in the application, isolate it from the remainder of the code, and determine whether it behaves exactly as you expect.  
<cite>_[Microsoft](https://msdn.microsoft.com/en-us/library/aa292197)_</cite>

- In order to isolate, unit tests cannot test dependency implementation.
- Use [shallow stubs](#shallow-stubs) to replicate dependency state.  
- If most of the implementation is invoking dependencies, the test itself will be reduced to the control flow of the promise chain and its handlers, for example, unit tests for AWS lambda transport layers generally follow a similar pattern of `event › mockValidateEvent › mockBusinessLayer › mockReply|mockReplyInternalError`. As most of the implementation is invoking dependencies (which we are not testing at the unit layer), unit tests for transport layers will generally only test that control flow works given expected dependency behaviour.

### Integration

> Integration tests are like unit tests without using test doubles for a certain subset of dependencies, essentially testing the interactions between the software its dependencies.  
<cite>_[Denis Sokolov](https://code.tutsplus.com/articles/tdd-terminology-simplified--net-30626)_</cite>

- Integration tests interact with AWS services on the sandbox account. This approach assumes:
  - Unlike the reliability of the state of the source code on the sandbox account the underlying services (infrastructure) should be solid.
  - Each environment (sandbox, develop & master) have the same services provisioned (Should be handled by deployer).
- See [Common test cases](#ses) for examples how to test aws services in integration tests.  
- Avoid using test doubles; prefer real working implementations so we can inspect AWS services are working as expected where possible.

**Additional Reading**
[Write tests. Not too many. Mostly integration.](https://blog.kentcdodds.com/write-tests-not-too-many-mostly-integration-5e8c7fff591c)


## Test framework  
- Use [node-tap](http://www.node-tap.org) as a testing framework. The benefits are:
  - code coverage out of the box
  - A minimal API surface.
  - Maintained by NPM CEO.
  - Outputs a common, widely used protocol in TAP.
  - Compatible with node debugging tools like `devtool`, `denode` & nodejs native debugger module so you can set breakpoints while testing (impossible in AVA).
- Stick to the single assertion `t.strictSame` where possible so no special knowledge of an assertion library is required.
- Stick to `t.end`, don't need to use `t.plan`.
- Keep tests lightweight, let tests fail through timing out no need to explicitly call `t.fail`. The only exception is for tests that return Promises, as they pass if rejected without a catch handler.

### Bad  
```javascript
test('getNumber resolves a number', t =>
  getNumber().then(value => t.strictSame(isNaN(value), false))
);
```

### Best
```javascript
test('getNumber resolves a number', t =>
  getNumber()
    .then(value => t.strictSame(isNaN(value), false))
    .catch(reason => t.fail())
);
```

**Additional Reading**  
[Why I use Tape Instead of Mocha & So Should You](https://medium.com/javascript-scene/why-i-use-tape-instead-of-mocha-so-should-you-6aa105d8eaf4#.ubnmyaiub)


## Common test cases

```javascript
// Passes test
test('pass this', t => {
  t.pass();
  t.end();
});

// throws error
test('Throws error', t => {
  try {
    fnThrowsError();
  } catch (e) {
    t.pass();
    t.end();
  }
});

// returns promise
test('returns promise', t => {
  const isPromise = fnReturnsPromse();
  t.strictSame(typeof isPromise.then, 'function');
  t.end();
});
```

### Integration

#### SES
- To test a modules integration with the AWS SES, have your module send emails to `tap@optizmo.grit.run`. Modifying such a value at runtime could be done either as input (i.e. http request payload) or as an environment variable.  
- Behind the scenes the sandbox account has an associated ses rule, s3 bucket, sns topic & sqs queue already provisioned for capturing emails sent to the tap recipient.
- The functionality to read mail headers is encapsulated behind `tests.createPollMail` & `tests.createMailHeader`.

```javascript
const { tests } = require('utils');

// return pollMail from test, run assertions from within the then callback.
test('sends job started email to principal', t =>
  // createPollMail long polls sqs for incoming mail to tap@optizmo.grit.ninja
  tests.createPollMail(aws).then(value => {
    // createMailHeader parses raw mail to easily obtain mail header values
    const mailHeader = tests.createMailHeader(value);
    t.strictSame('hello@world.com', mailHeader('From'));
    t.strictSame('Hello world', mailHeader('Subject'));
    t.end();
  });
);
```


## Gotchas

**resolved promise expected to reject will still pass**  
node-tap will pass a test that returns either a Promise without a resolved/rejected handler. If you are only testing for a rejection and the promise resolves, you will get a false positive.

```javascript
test('resolved promise that is expected to reject will still pass test', t =>
  Promise.resolve('no then listener will pass')
    .catch(reason => {
      t.true(reason === 'Reason for rejection'); // We have
    }));
```

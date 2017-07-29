[🏠](README.md) › React Design patterns & best practices


# React

- Prefer idiomatic React conventions over bleeding edge javascript.  

- [Conventions](#conventions)
  - [es6 classes](#es6-classes)
  - [Presentational and container components](#presentational-and-container-components)
  - [Higher Order Components](#Higher-order-components)
  - [Performance considerations](#performance-considerations)
- [Development practices](#development-practices)
  - [Development Mode](#development-mode)
  - [Testing](#testing)
- [Libraries](#libraries)
- [Projects](#projects)
  - [Structure](#structure)

## Conventions

### es6 classes

- es6 classes are the idiomatic way for creating stateful (container) components as recommended by the React core team.  
- Avoid deep hierarchies; only `extend` the `React.component` class. Use [higher order components](#Higher-order-components) for building complex components from simpler, smaller components.  

#### caveats  
- Components using es6 classes have a few caveats, consider this component:

```javascript
class extends React.Component {
  constructor() {
    super();
    this.state = { name: 'Ben' };
  }
  toggleName() { ... }
  render() {
    return <p toggleName={this.toggleName.bind(this)}>{this.state.name}</p>
  }
}
```

- `super()` must be called before `this.state` as their is on instance and thus no `this` before the React.Component constructor has been invoked.
- Do not use getIntitialState lifecycle method, instead declare state as an plain object within the constructor
- class methods are not bound to the component instance (consistent with es2015 classes). To avoid appending `bind(this)` to each method use arrow functions `onClick={() => this.toggleName}` or bind them in the constructor `this.toggleName = this.toggleName.bind(this)`.
- propTypes and defaultProps are defined as properties on the constructor instead of in the class body.

```javascript
const MyComponent = class extends React.Component {}
MyComponent.defaultProps = { foo: 'foo' }
MyComponent.propTypes = { foo: React.PropTypes.string };
```


### Presentational and container components

**Simplified component separation**

   | Presentational |  Container
---|---|--
State | N |  Y
Lifecycle methods   | N | Y
this keyword | N | Y
Renders | Y | N
Fetches data | N | Y
Naming convention | component_name | container_component_name

Separating presentational from container components enables:  
  1. Reusability  
  1. Type checking of data (By setting React.propTypes on presentational component)  


#### Presentational components  
Presentational components are stateless, pure components that are a function of their props. They take a single props argument and return a React element to render.

const ListItem = props => <li>{props.text}</li>;
ListItem.propTypes = {
  text: React.PropTypes.string,
}


#### Container components  
A container does data fetching and then renders its corresponding sub-component. That’s it.

StockWidgetContainer => StockWidget




have performance optimizations
- No internal state
- No backing instances (React.component just a React.Element)
- No component lifecycle methods
- propTypes + defaultProps can be set as properties of the function

**further reading**  
- [Reusable Components](https://facebook.github.io/react/docs/reusable-components.html)
- [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)
- [](https://medium.com/@learnreact/container-components-c0e67432e005#.grklqyxu6)


### Higher Order Components
Idiomatic convention for working with presentational and container components.  


```javascript
const Container = Presenter => class extends React.Component {
  constructor() {
    super();
    this.state = { name: 'Ben' };
  }
  toggleName() {
    const name = (this.state.name === 'Ben') ? 'Court' : 'Ben';
    this.setState({ name });
  }
  render() {
    return <Presenter {...this.state} toggleName={this.toggleName.bind(this)} />
  }
}

const Presenter = props => <p onClick={props.toggleName}>{props.name}</p>;
Presenter.propTypes = {
  name: React.PropTypes.number,
}
Presenter.defaultProps = {
  name: 'Mason',
}
const App = Container(Presenter);
```

### Performance Considerations  

**Further reading**  
(https://medium.com/modus-create-front-end-development/component-rendering-performance-in-react-df859b474adc#.h7bg24bfy)

## Development practices


### Development Mode

- Logs warnings to the console when propType validation fails.  
- Enabled by default with the uncompressed version of react and disabled in minified version.  
- Use `propTypes` to get runtime errors logged to the console in development mode.  


- Inspect React within the browser using [devtools]()


### testing

- React documentation singles out [Enzyme by Airbnb](http://airbnb.io/enzyme/) as a unit testing framework.  


### Webpack

- Provides a single abstraction for most tooling
- Bundle source code into a single file or many "chunked" files
- Perform transformations (css precompilers,  optimizations (minification) ) on source code
- Can receive HTML, JS, CSS and small images as input
-


## Libraries


- React router, Helmet


## Projects

### Structure

- [Group components by feature not type](https://vimeo.com/168648012)
-






## TODO: group into sections

managing state
- Use redux for component agnostic (app) state
- state is a source of truth, [declare it in a single location](https://facebook.github.io/react/tips/props-in-getInitialState-as-anti-pattern.html).
- Component specific state should be declared in the constructor, after `super()`.  
- Avoid directly accessing state by passing state to child components as props.


boilerplate tooling

https://www.npmjs.com/package/sitegen

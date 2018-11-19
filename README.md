# with-immutable-props-to-js

[![Builds](https://img.shields.io/circleci/project/github/tophat/with-immutable-props-to-js.svg)](https://circleci.com/gh/tophat/with-immutable-props-to-js) [![codecov](https://codecov.io/gh/tophat/with-immutable-props-to-js/branch/master/graph/badge.svg)](https://codecov.io/gh/tophat/with-immutable-props-to-js) [![Greenkeeper badge](https://badges.greenkeeper.io/tophat/with-immutable-props-to-js.svg)](https://greenkeeper.io/)

A higher-order component for keeping Immutable objects outside your presentational components

## Installation

```
yarn add with-immutable-props-to-js
```

or

```
npm install with-immutable-props-to-js
```

This library also lists `react` and `immutable` as peer dependencies, so make sure they are installed in your project as well.

## Usage

```javascript
import withImmutablePropsToJS from 'with-immutable-props-to-js'
```

If you're not using ECMAScript modules:

```javascript
const withImmutablePropsToJS = require('with-immutable-props-to-js').default
```

Example:

```javascript
import React from 'react'
import { connect } from 'react-redux'
import { withImmutablePropsToJS } from 'with-immutable-props-to-js'

const MyDumbComponent = props => {
   // ...
   // props.objectProp is turned into a plain JavaScript object
   // props.arrayProp is turn into a plain JavaScript array
}

MyDumbComponent.propTypes = {
   objectProp: PropTypes.object,
   arrayProp: PropTypes.array,
}

const mapStateToProps = state => ({
   objectProp: mySelectorThatReturnsImmutableMap(state),
   arrayProp: mySelectorThatReturnsImmutableList(state),
})

export default connect(mapStateToProps)(withImmutablePropsToJS(MyDumbComponent))
```

## Motivation

The need for this higher-order component stems from the simultaneous use of several common libraries/patterns in React applications:

- [Redux](https://redux.js.org/) is a great way to manage application state.
- [Selectors](https://redux.js.org/introduction/learningresources#selectors) are a great way to encapsulate state access.
- [Immutable.js](https://facebook.github.io/immutable-js/) is a great way to make immutable updates in your [reducers](https://redux.js.org/basics/reducers).

When using these technologies together, there are some important rules of thumb you should follow:

- *Don't us Immutable.js in your presentational (dumb) components*: this makes your components more reusable, as their interfaces don't mandate the use of Immutable.js
- *Don't call .toJS() on Immutable objects in selectors*: .toJS() will return a brand new object every time, and cause your React components to re-render even if your app state doesn't change; return Immutable objects from all your (non-primitive type) selectors for consistency
- *Don't call .toJS() on Immutable objects in mapStateToProps*: for the same reason as above, doing this will causes unnecessary re-renders

At this point, the keen observer will have realized: _there's no way left to get data from selectors into my presentational components_.

Solution: use `with-immutable-props-to-js`. The higher order component takes its props, calls .toJS() on any Immutable objects, and passes all props along with the converted props to the wrapped component.
Your dumb components remain portable, all calls to .toJS() are isolated within the higher-order component, and the higher-order component controls re-rendering, so it only rerenders the wrapped dumb component when `connect` determines that the Immutable values from the selectors have changed. Sweet!

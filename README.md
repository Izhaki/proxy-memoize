# proxy-memoize

[![CI](https://img.shields.io/github/workflow/status/dai-shi/proxy-memoize/CI)](https://github.com/dai-shi/proxy-memoize/actions?query=workflow%3ACI)
[![npm](https://img.shields.io/npm/v/proxy-memoize)](https://www.npmjs.com/package/proxy-memoize)
[![size](https://img.shields.io/bundlephobia/minzip/proxy-memoize)](https://bundlephobia.com/result?p=proxy-memoize)

Intuitive magical memoization library with Proxy and WeakMap

## Introduction

In frontend framework like React, object immutability is important.
JavaScript itself doesn't support forcing immutability.
Several libraries help encouraging immutable coding style,
like [immer](https://github.com/immerjs/immer).
While immer helps updating an object,
this library helps creating a derived value from an object, a.k.a. selector.

This library utilizes Proxy and WeakMap, and provides memoization.
The memoized function will re-evaluate the original function
only if the used part of argument (object) is changed.
It's intuitive in a sense and magical in another sense.

## Install

```bash
npm install proxy-memoize
```

## How it behaves

```js
import memoize from 'proxy-memoize';

const fn = memoize(x => ({ sum: x.a + x.b, diff: x.a - x.b }));

fn({ a: 2, b: 1, c: 1 }); // ---> { sum: 3, diff: 1 }
fn({ a: 3, b: 1, c: 1 }); // ---> { sum: 4, diff: 2 }
fn({ a: 3, b: 1, c: 9 }); // ---> { sum: 4, diff: 2 } (returning a cached value)
fn({ a: 4, b: 1, c: 9 }); // ---> { sum: 5, diff: 3 }

fn({ a: 1, b: 2 }) === fn({ a: 1, b: 2 }); // ---> true
```

## Usage with React Context

Instead of bare useMemo.

```js
const Component = (props) => {
  const [state, dispatch] = useContext(MyContext);
  const render = useCallback(memoize(([props, state]) => (
    <div>
      {/* render with props and state */}
    </div>
  )), [dispatch]);
  return render([props, state]);
};

const App = ({ children }) => (
  <MyContext.Provider value={useReducer(reducer, initialState)}>
    {children}
  </MyContext.Provider>
);
```

- [CodeSandbox](https://codesandbox.io/s/proxy-memoize-demo-vrnze)

## Usage with React Redux

Instead of [reselect](https://github.com/reduxjs/reselect).

```js
import { useSelector } from 'react-redux';

const getScore = memoize(state => ({
  score: heavyComputation(state.a + state.b),
  createdAt: Date.now(),
}));

const Component = ({ id }) => {
  const { score, title } = useSelector(useCallback(memoize(state => ({
    score: getScore(state),
    title: state.titles[id],
  })), [id]));
  return <div>{score.score} {score.createdAt} {title}</div>;
};
```

- [CodeSandbox 1](https://codesandbox.io/s/proxy-memoize-demo-c1021)
- [CodeSandbox 2](https://codesandbox.io/s/proxy-memoize-demo-fi5ni)

## Usage with Zustand

For derived values.

```js
import create from 'zustand';

const useStore = create(set => ({
  valueA,
  valueB,
  // ...
}));

const getDerivedValueA = memoize(state => heavyComputation(state.valueA))
const getDerivedValueB = memoize(state => heavyComputation(state.valueB))
const getTotal = state => getDerivedValueA(state) + getDerivedValueB(state)

const Component = () => {
  const total = useStore(getTotal)
  return <div>{total}</div>;
};
```

- [CodeSandbox](https://codesandbox.io/s/proxy-memoize-demo-yo00p)

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

### memoize

create a memoized function

#### Parameters

-   `fn` **function (obj: Obj): Result** 
-   `options` **{size: [number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)?}?** 

#### Examples

```javascript
import memoize from 'proxy-memoize';

const fn = memoize(obj => ({ sum: obj.a + obj.b, diff: obj.a - obj.b }));
```

Returns **function (obj: Obj): Result** 

## Related projects

-   [react-tracked](https://github.com/dai-shi/react-tracked)
-   [reactive-react-redux](https://github.com/dai-shi/reactive-react-redux)
-   [svelte3-redux](https://github.com/dai-shi/svelte3-redux)
-   [memoize-state](https://github.com/theKashey/memoize-state)
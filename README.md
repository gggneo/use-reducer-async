# use-reducer-async

[![Build Status](https://travis-ci.com/dai-shi/use-reducer-async.svg?branch=master)](https://travis-ci.com/dai-shi/use-reducer-async)
[![npm version](https://badge.fury.io/js/use-reducer-async.svg)](https://badge.fury.io/js/use-reducer-async)
[![bundle size](https://badgen.net/bundlephobia/minzip/use-reducer-async)](https://bundlephobia.com/result?p=use-reducer-async)

React useReducer with async actions

## Introduction

React useReducer doesn't support async actions natively.
Unlike Redux, there's no middleware interface, but hooks are composable.

This is a tiny library to extend useReducer's dispatch
so that dispathing async actions invoke async functions.

## Install

```bash
npm install use-reducer-async
```

## Usage

```javascript
const initialState = {
  sleeping: false,
};

const reducer = (state, action) => {
  switch (action.type) {
    case 'START_SLEEP': return { ...state, sleeping: true };
    case 'END_SLEEP': return { ...state, sleeping: false };
    default: throw new Error('no such action type');
  }
};

const asyncActionHandlers = {
  SLEEP: ({ dispatch }) => async (action) => {
    dispatch({ type: 'START_SLEEP' });
    await new Promise(r => setTimeout(r, action.ms));
    dispatch({ type: 'END_SLEEP' });
  },
};

const Component = () => {
  const [state, dispatch] = useReducerAsync(reducer, initialState, asyncActionHandlers);
  return (
    <div>
      <span>{state.sleeping ? 'Sleeping' : 'Idle'}</span>
      <button type="button" onClick={() => dispatch({ type: 'SLEEP', ms: 1000 })}>Click</button>
    </div>
  );
};
```

### Notes for abortability

If an async action handler dispatches an action after the component is unmounted,
React in the development mode warns about possible memory leaks.

It is the responsibility of the async action handler to deal with such cases.

All async action handlers receive `signal` in the argument.
Refer [`examples/04_abort/src`](./examples/04_abort/src) for the usage.

Note: The implementation depends on [AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) in the DOM spec.
If you are using an environment that doesn't have AbortController (for example IE11), you need a polyfill:
[1](https://github.com/mo/abortcontroller-polyfill)
[2](https://github.com/mysticatea/abort-controller)

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

### useReducerAsync

useReducer with async actions

#### Parameters

-   `reducer` **R** 
-   `initialState` **ReducerState&lt;R>** 
-   `asyncActionHandlers` **AsyncActionHandlers&lt;R, AsyncAction>** 

#### Examples

```javascript
import { useReducerAsync } from 'use-reducer-async';

const asyncActionHandlers = {
  SLEEP: ({ dispatch, getState, signal }) => async (action) => {
    dispatch({ type: 'START_SLEEP' });
    await new Promise(r => setTimeout(r, action.ms));
    dispatch({ type: 'END_SLEEP' });
  },
  FETCH: ({ dispatch, getState, signal }) => async (action) => {
    dispatch({ type: 'START_FETCH' });
    try {
      const response = await fetch(action.url);
      const data = await response.json();
      dispatch({ type: 'FINISH_FETCH', data });
    } catch (error) {
      dispatch({ type: 'ERROR_FETCH', error });
    }
  },
};
const [state, dispatch] = useReducerAsync(reducer, initialState, asyncActionHandlers);
```

Returns **\[ReducerState&lt;R>, Dispatch&lt;ExportAction>]** 

## Examples

The [examples](examples) folder contains working examples.
You can run one of them with

```bash
PORT=8080 npm run examples:01_minimal
```

and open <http://localhost:8080> in your web browser.

You can also try them in codesandbox.io:
[01](https://codesandbox.io/s/github/dai-shi/use-reducer-async/tree/master/examples/01_minimal)
[02](https://codesandbox.io/s/github/dai-shi/use-reducer-async/tree/master/examples/02_typescript)
[03](https://codesandbox.io/s/github/dai-shi/use-reducer-async/tree/master/examples/03_getstate)
[04](https://codesandbox.io/s/github/dai-shi/use-reducer-async/tree/master/examples/04_abort)

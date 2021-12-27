
A question of `useRef` vs `useState` is an interesting on when discussing
React.  Although the use case differs a lot, the implementation not so much.
Best, let us look at the code.

Here are some snippets from `packages/react-reconciler/src/ReactFiberHooks.js`. We start with the implementation of `useRef` hook:

```js
function mountRef(initialValue) {
  ...

  const ref = { current: initialValue };
  if (__DEV__) {
    Object.seal(ref);
  }
  hook.memoizedState = ref;
  return ref;
}
```

Surprisingly, this is not a lot of code. An object with a property *current* is
created and persisted, that is all. Additionally the `Object.seal` method is
called in development build to prevent adding new properties.  There is no
re-render mechanism, but the hook is just a heap for data. From the
implementation point it also bears no connection to DOM, which is just a common
use case.

 Now we will see that the `useState` hook does not look very different:

```js
function mountState(initialState) {
  ...

  if (typeof initialState === 'function') {
    initialState = initialState();
  }
  hook.memoizedState = hook.baseState = initialState;

  const queue = (hook.queue = {
    last: null,
    dispatch: null,
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: (initialState: any),
  });

  const dispatch = ...
    ...

  return [hook.memoizedState, dispatch];
}
```

The first line allows to lazy-initialize the state, but then the state object
is persisted in the same way as in *useRef*. The fundamental difference lays in
the way in which the data is updated, a call to update the state involves
internal processing that leads to component re-render. Furthermore what is
interesting is that *useState* is just a special case of reducer under the
hood:

```js
function updateState(initialState) {
  return updateReducer(basicStateReducer, initialState);
}
```

Interestingly many other hooks share the same internals, here is the implementation of `useCallback`, which might be seen as just a wrapper:

```js
function mountCallback(callback, deps): {
  ...

  const nextDeps = deps === undefined ? null : deps;
  hook.memoizedState = [callback, nextDeps];
  return callback;
}
```

What makes a small difference, that is deps comparison, is implemented in the
update method:

```js
function updateCallback(callback, deps) {
  ...

  const nextDeps = deps === undefined ? null : deps;
  const prevState = hook.memoizedState;
  if (prevState !== null) {
    if (nextDeps !== null) {
      const prevDeps = prevState[1];
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        return prevState[0];
      }
    }
  }
  hook.memoizedState = [callback, nextDeps];
  return callback;
}
```

`useMemo` ? Anyone interested ?

```js
function mountMemo(nextCreate, deps) {
  ...

  const nextDeps = deps === undefined ? null : deps;
  const nextValue = nextCreate();
  hook.memoizedState = [nextValue, nextDeps];
  return nextValue;
}


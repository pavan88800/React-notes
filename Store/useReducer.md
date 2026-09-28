# React `useReducer` Hook

## How It Works

### Syntax

```js
const [state, dispatch] = useReducer(reducer, initialState);
```

## Reducer Function

A reducer is a pure function that takes the current state and an action, then returns the next state.

```js
function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };

    default:
      return state;
  }
}
```

## Dispatch Function

dispatch() is used to send an action to trigger a state update.

```js
dispatch({ type: "increment" });
```

## An action can also contain a payload when additional data is needed:

```js
dispatch({
  type: "setName",
  payload: "Pavan"
});
```

## When to Use useReducer

1. Complex State Logic

Use useReducer when the state contains multiple related values or has multiple state transitions.

## 2. Next State Depends on Previous State

It is useful when state transitions depend heavily on the previous state.

## 3. Centralized Update Logic

It allows you to keep state-changing logic inside one reducer instead of scattering it across multiple event handlers.

# Redux – Centralized Store & Unidirectional Data Flow

## 1. What is Redux?

Redux is a **state management library** that provides centralized state management and a predictable state update mechanism using unidirectional data flow.

- Centralized state management a predictable state update mechanism
- Unidirectional data flow
- A single centralized store in a typical Redux application

The Basic idea is:

> **One centralized store + predictable state updates + unidirectional data flow.**

---

## 2. Centralized Store

Redux typically uses **one store** for the entire application.

For example:

```text
                Redux Store
        ┌───────────┼───────────┐
        ↓           ↓           ↓
       User        Cart       Products
       State       State        State
```

Different parts of the application can be logically separated using **slices**:

```js
configureStore({
  reducer: {
    user: userReducer,
    cart: cartReducer,
    products: productsReducer
  }
});
```

So although we have multiple slices, they are part of the **same centralized store**.

---

## 3. Can Redux technically have multiple stores?

Yes.

Technically, we can create multiple Redux store instances.

For example:

```text
User Store
Cart Store
Product Store
```

However, this is generally **not recommended for a normal Redux application**.

### Why?

The main problem is **coordination between independent stores**.

Suppose a product is removed from the cart:

```text
Cart Store
     ↓
Product Store needs an update
     ↓
User Store needs an update
     ↓
UI needs to reflect all changes
```

Now we need additional mechanisms to coordinate those state updates.

As the application becomes larger:

- Stores can become dependent on each other.
- State synchronization becomes harder.
- It becomes harder to understand the complete data flow.
- There is a higher risk of inconsistent or out-of-sync state.
- Debugging becomes more complicated.

Therefore, instead of creating multiple stores, Redux generally uses:

```text
ONE STORE
   ↓
Multiple Slices
   ↓
Different logical domains
```

---

# 4. Advantages of a Single Store

### Centralized state

There is one central source of truth:

```text
                Redux Store
                     ↓
        ┌────────────┼────────────┐
        ↓            ↓            ↓
      User          Cart        Products
```

A component can select the specific state it needs.

```js
const user = useSelector((state) => state.user);
```

---

### Easier state coordination

Suppose:

```text
Product removed from cart
```

A single action can be processed by multiple reducers/slices if required.

```text
                Action
                  ↓
              Redux Store
              ↙    ↓    ↘
          Cart   Product  User
          Slice   Slice   Slice
```

This keeps related state changes within the same predictable Redux flow.

---

### Easier debugging

Redux DevTools can show:

```text
Previous State
      ↓
    Action
      ↓
  Next State
```

This makes it easier to understand how the application state changed.

---

# 5. Redux Unidirectional Data Flow

Redux follows **one-way / unidirectional data flow**.

The basic flow is:

```text
User Interaction
       ↓
      View
       ↓
   dispatch()
       ↓
     Action
       ↓
    Reducer
       ↓
  Redux Store
       ↓
  Updated State
       ↓
      View
```

Example:

```js
dispatch(addToCart(product));
```

The component does **not directly modify the Redux state**.

Instead, it dispatches an action.

---

# 6. What is an Action?

An action describes **what happened**.

Example:

```js
{
  type: "cart/addToCart",
  payload: product
}
```

The action itself doesn't directly change the state.

It tells Redux:

> "This event happened."

---

# 7. What is Dispatch?

`dispatch()` sends an action to Redux.

Example:

```js
dispatch(addToCart(product));
```

The flow is:

```text
Component
    ↓
dispatch(action)
    ↓
Redux
```

---

# 8. What is a Reducer?

A reducer determines **how the state should change** based on the action.

Conceptually:

```text
Current State + Action
          ↓
       Reducer
          ↓
      Next State
```

Example:

```js
const cartSlice = createSlice({
  name: "cart",

  initialState: {
    items: []
  },

  reducers: {
    addToCart(state, action) {
      state.items.push(action.payload);
    }
  }
});
```

---

# 9. Components and Redux State

Components should not directly modify Redux state.

❌ Avoid thinking:

```js
store.cart.push(product);
```

Instead:

```js
dispatch(addToCart(product));
```

To read state:

```js
const cart = useSelector((state) => state.cart);
```

So:

```text
Component
   │
   ├── Read → useSelector()
   │
   └── Update → dispatch()
```

---

# 10. Why Unidirectional Data Flow?

The main benefit is **predictability**.

Instead of components changing state in arbitrary directions:

```text
Component A ↔ Store ↔ Component B
       ↕          ↕
Component C ↔ Store ↔ Component D
```

Redux gives us a predictable flow:

```text
Component
    ↓
 Action
    ↓
Reducer
    ↓
 Store
    ↓
Updated State
    ↓
Components
```

This makes the application easier to:

- Understand
- Debug
- Test
- Maintain
- Reason about

---

# 11. Important Interview Point

Don't say:

> "Redux cannot have multiple stores."

That's technically incorrect.

Say:

> **"Redux can technically have multiple store instances, but a typical Redux application is designed around a single store. Multiple stores make coordination and synchronization between different state domains more difficult. Redux instead uses one centralized store with multiple slices to keep the state logically separated while maintaining a predictable state flow."**

---

# 12. Final Interview Answer

If the interviewer asks:

### "What is Redux?"

You can answer:

> **"Redux is a state management library that provides centralized state management and predictable unidirectional data flow. Typically, we have one centralized store containing different slices such as user, cart, and products. Components read the state they need using selectors and dispatch actions when something happens. Reducers process those actions and produce the next state. This centralized approach makes state changes predictable and easier to debug and maintain."**

### Easy mental model

```text
                 REDUX
                   │
        ┌──────────┴──────────┐
        │                     │
 Centralized Store     Unidirectional Flow
        │                     │
   Multiple Slices     View → Action
        │                     ↓
        │                  Reducer
        │                     ↓
        └─────────────── Store
                              ↓
                             View
```

**Remember:**

> **One Store → Multiple Slices → Actions → Reducers → Updated State → UI**

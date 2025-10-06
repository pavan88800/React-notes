## 📘 React Rendering Notes (Parent → Child + Memo)

## 🔹 1. Normal Behavior

When a parent component re-renders, React will also re-render all of its children by default,
even if:

The child has no props, OR The child’s props did not change.

👉 Reason: React re-runs the whole subtree under the parent.

---

## 🔹 2. With React.memo

React.memo wraps a child and tells React:
👉 “Only re-render me if my props have changed.”

Effect:

If parent re-renders but child’s props are same → child will not re-render.

If parent re-renders and child’s props changed → child will re-render.

---

## 🔹 3. Important Nuances

React.memo checks props using shallow comparison (objects/arrays compared by reference).

If props are complex objects/arrays that are new every render, child will still re-render.

Solution → use useMemo / useCallback to stabilize props.

---

## 🔹 4. When React.memo Won’t Help

If the child uses Context (useContext) → it will re-render whenever the context value changes (memo can’t stop this).

If the child has its own state change → it re-renders regardless of memo.

React.memo only optimizes props from parent → child.

---

## 🔹 5. Quick Examples

## 🟢 Without React.memo

```js
function Parent({ count }) {
  return (
    <>
      <Child />
      <p>Count: {count}</p>
    </>
  );
}

function Child() {
  console.log("Child render");
  return <h1>Child</h1>;
}

## Every time parent re-renders → Child logs again.
```

---

## 🟢 With React.memo

```js
const Child = React.memo(() => {
  console.log("Child render");
  return <h1>Child</h1>;
});

## Now Child only re-renders if props to it actually change.
```

---

## ✅ Golden Rules

Parent re-render → all children re-render (default).

Wrap child in React.memo → only re-renders when props change.

React.memo is for performance optimization, not correctness.

Combine React.memo + useCallback / useMemo for stable props.

Don’t overuse memoization → use only when performance issues appear.

---

## 🔹 Why React.memo, useMemo, useCallback shouldn’t be overused

## 1. Extra Work for React

Memoization is not free — React has to store the previous reference and compare props/values each render.

If the component is small and cheap to render, this extra check can actually make performance worse.

---

## 2. Memory References

useMemo / useCallback keep references in memory.

If you overuse them everywhere, you’re basically filling React with a bunch of “cached” references that
don’t really save much time but still consume memory.

---

## 3. Complexity Overhead

Code becomes harder to read when everything is wrapped in memo, useMemo, useCallback.

You should keep them only where performance benefit > complexity cost.

---

## 🔹 When you SHOULD use memoization

When you have expensive calculations → useMemo.

When passing functions as props to React.memo children → useCallback.

When child re-renders unnecessarily due to unchanged props → React.memo.

---

## 🔹 When you should NOT use memoization

For tiny components (like a <Button> or <span>).

If re-render cost is cheaper than comparison cost.

If state/props change too often (then caching doesn’t help much).

---

## Note to taken:-

React.memo only prevents unnecessary re-renders when the props from parent → child have not changed (by shallow comparison).

That means:

✅ If the parent re-renders but the child’s props didn’t change → React.memo will stop the child from re-rendering.

❌ If the child uses Context API (useContext) → whenever the context value changes, all consumers will re-render, even if wrapped in React.memo.

Because context bypasses props and directly injects values.

Optimization here: split context into smaller ones or use selectors (like use-context-selector).

❌ If the child uses Redux (useSelector) → component re-renders whenever the selected slice changes.

React.memo won’t stop it, because redux updates come through hooks, not props.

Optimization here:

Select only the exact state needed (useSelector(state => state.user.name)).

Use memoized selectors (reselect).

Sometimes wrap sub-components with React.memo if you pass props down.

---

1. Re-render

Happens when props or state change inside a component.

The component function executes again, so local variables inside the function are redeclared.

BUT:

useState preserves its value across re-renders.

useRef preserves its .current value across re-renders.

Effects (useEffect) may run again depending on dependency array.

Cleanup function of previous effect only runs if dependencies change.

---

2. Unmount

Happens when the component is removed from the DOM.

React runs the cleanup functions of all useEffects for that component.

All state (useState) and refs (useRef) are lost because the component is destroyed.

```js
useEffect(() => {
  console.log("Effect runs, count:", count);

  return () => {
    console.log("Cleanup runs, count:", count);
  };
}, []);
```

Key points:

The return inside useEffect is a cleanup function.

It does NOT unmount the component by itself.

React calls this function when the component is about to unmount or before re-running the effect (if dependencies change).

## Unmounting only happens when React removes the component from the UI, e.g., via conditional rendering:

```js
{
  showChild && <Child />;
}
```

showChild = false → Child is unmounted → React calls the cleanup function.

So in your example:

If the component stays mounted, the cleanup function will not run, even though it’s returned from useEffect.

## The cleanup runs automatically on unmount or before re-running the effect if dependencies change.

## NOTE:-

Unmount only destroys the component.

React removes it from the UI.

Cleanup functions run.

State and refs are lost.

It does NOT automatically create a new component.

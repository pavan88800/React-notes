# JS Execution & UI Rendering: React vs Vanilla JS

This note explains why UI elements like modals may appear delayed, even if their code executes first, in both **React** and **vanilla JavaScript**.

---

## 1. Core Principle

> **JavaScript is single-threaded. The browser cannot paint updates until the current JS execution finishes.**

- Any **heavy synchronous JS** blocks the main thread.
- Browser waits until JS finishes to **paint DOM changes**.

---

## 2. Vanilla JS Example

```html
<script>
  document.getElementById("modal").style.display = "block"; // runs first
  heavyLoop(); // blocks JS thread
</script>
```

Step 1: Modal DOM update executes ✅

Step 2: heavyLoop() executes ❌ → blocks browser

Result: Modal appears after heavy JS finishes, even though it ran first.

Reason: JS thread is busy; browser paint is blocked.

---

## 3. React Example

```js
{
  isOpen && <Model />;
}
<VerySlowComponent />;
```

Step 1: Model() executes first → returns JSX (Virtual DOM) ✅

Step 2: VerySlowComponent() executes → heavy JS ❌ blocks main thread

Step 3: React commits Virtual DOM → updates real DOM

Step 4: Browser paints → modal appears

Even though Model() runs first, browser cannot paint until render phase completes.

---

# React Re-rendering & Performance Notes 🚀

## 1. When Does a Component Re-render?

A component re-renders in 3 cases:

1. **Its own state changes**
2. **Its parent re-renders**
3. **Its props change**

---

## 2. JSX Creates New Objects Every Render

Every time a component renders, it creates **brand new JavaScript objects** for its JSX:

```jsx
// This is literally just:
<VerySlowComponent />
// → React.createElement(VerySlowComponent, null)
// → { type: VerySlowComponent, props: {} }  ← new object every render!
```

- Same reference → React **skips** re-render
- New object → React **re-renders**

---

## 3. Passing Content as Prop (Clever Pattern!)

```jsx
export default function App() {
  return (
    <ScrollableWithMovingBlock
      content={
        <>
          <VerySlowComponent />
          <BunchOfstuff />
          <OtherStuffAlsoComplicated />
        </>
      }
    />
  );
}
```

### Why children DON'T re-render here:

- `App` has **no state** → never re-renders
- `content` JSX is created **once**, same reference forever
- When `ScrollableWithMovingBlock` updates its own state → `content` prop reference unchanged → children **skip re-render** ✅

### When they WILL re-render:

- If components are placed **inline** inside stateful component → new JSX objects every render → re-renders!

---

## 4. Custom Hook Instances are ISOLATED per Component

```jsx
const useCounter = () => {
  const [count, setCount] = useState(0);
  return { count, setCount };
};

const A = () => {
  const { count } = useCounter(); // A's OWN private instance
};

const B = () => {
  const { count } = useCounter(); // B's OWN private instance
};
```

### Key Points:

- Each component gets its **own private copy** of the hook's state
- Changing A's state **never affects** B
- Hook instances are **completely isolated**

Think of it like a **house blueprint** — same blueprint, but every family builds their own separate house!

---

## 5. Hook State Re-renders Its HOST Component

```jsx
const useModel = () => {
  const [internalState, setInternalState] = useState(false);
  const toggle = () => setInternalState(!internalState);
  return { toggle };
};

const A = () => {
  useModel(); // just calling is enough!
  return <div>Hello</div>;
};
```

### Rules:

| Scenario                              | Re-render?                 |
| ------------------------------------- | -------------------------- |
| Hook called, setState never triggered | ❌ No                      |
| Hook called, setState triggered       | ✅ Yes                     |
| Return value used or not              | Doesn't matter!            |
| B also uses same hook                 | ❌ B's instance unaffected |

> **Hook doesn't re-render itself — it makes its HOST component re-render!**

Hook is NOT independent. It **attaches itself** to whichever component calls it.

---

## 6. State Colocation — Push State Down!

```jsx
// ❌ BAD — state in App, everything re-renders on button click!
export default function App() {
  const [isOpen, setIsOpen] = useState(false);
  return (
    <div>
      <button onClick={() => setIsOpen(prev => !prev)}>Open</button>
      {isOpen && <Model />}
      <VerySlowComponent />     {/* re-renders on every click! */}
      <BunchOfstuff />          {/* re-renders on every click! */}
    </div>
  );
}

// ✅ GOOD — state moved down, only ModelButton re-renders!
const ModelButton = () => {
  const [isOpen, setIsOpen] = useState(false);
  return (
    <>
      <button onClick={() => setIsOpen(prev => !prev)}>Open</button>
      {isOpen && <Model />}
    </>
  );
};

export default function App() {
  return (
    <div>
      <ModelButton />            {/* only this re-renders! ✅ */}
      <VerySlowComponent />      {/* never re-renders! ✅ */}
      <BunchOfstuff />           {/* never re-renders! ✅ */}
    </div>
  );
}
```

> **Always push state as LOW as possible in the tree — only components that NEED the state should own it!**

---

## 7. How to STOP Unnecessary Re-renders

### Option 1 — State Colocation (push state down)

Move state to the component that actually needs it.

### Option 2 — Pass content as prop

Keep stateful logic separate from heavy children by passing them as props.

### Option 3 — React.memo

```jsx
const Child = React.memo(() => {
  return <div>I won't re-render unless my props change!</div>;
});
```

### Option 4 — useRef instead of useState

```jsx
const useModel = () => {
  const internalState = useRef(false); // NO re-render!
  const toggle = () => {
    internalState.current = !internalState.current;
  };
  return { toggle };
};
```

---

## 8. Shared State — When hooks ARE shared

Custom hooks are isolated. To share state across components use:

```jsx
// Context API
const CountContext = createContext();

// or External stores
const store = create(() => ({ count: 0 })); // Zustand
const store = createStore({ count: 0 }); // Redux
```

Only **Context consumers** (`useContext`) re-render when context value changes.

---

## 9. Critical Rendering Path (CRP)

Steps browser takes to convert code into pixels on screen:

```
1. Parse HTML      → build DOM tree
2. Fetch CSS       → build CSSOM (CSS Object Model)
3. DOM + CSSOM     → create Render Tree
4. Layout          → calculate positions, width, height
5. Paint           → draw pixels, colors, text, images
6. Composite       → layer management, animations, transforms
```

### React + CRP connection:

- React re-renders happen in **JavaScript phase**
- After React updates DOM → browser runs **CRP again**
- Fewer React re-renders = fewer CRP cycles = **faster UI!** 🚀

---

## Golden Rules Summary 🎯

| Rule                              | Description                                     |
| --------------------------------- | ----------------------------------------------- |
| Hook instances are isolated       | Each component gets its own private copy        |
| Hook re-renders its host          | setState in hook = host component re-renders    |
| Push state down                   | Only components that need state should own it   |
| Same JSX reference = no re-render | Pass content as prop from stateless parent      |
| useRef = no re-render             | Use when you don't need visual updates          |
| Context/Redux = shared state      | Only way to truly share state across components |

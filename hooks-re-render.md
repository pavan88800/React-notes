# When Do Custom Hooks Cause Re-renders? 🎯

## Quick Answer

> **A component re-renders when its hook's `setState` is TRIGGERED — not just because the hook is called!**

---

## ✅ WILL Cause Re-render

### 1. Hook's setState is triggered

```jsx
const useModel = () => {
  const [isOpen, setIsOpen] = useState(false);
  const toggle = () => setIsOpen(!isOpen); // setState triggered!
  return { toggle };
};

const A = () => {
  const { toggle } = useModel();
  return <button onClick={toggle}>Click</button>; // triggers re-render
};
```

### 2. Hook called and its internal state updates

```jsx
const useCounter = () => {
  const [count, setCount] = useState(0);
  useEffect(() => {
    setCount(1); // triggers on mount → A re-renders!
  }, []);
  return { count };
};

const A = () => {
  useCounter(); // even without using return value → A re-renders!
  return <div>Hello</div>;
};
```

### 3. Hook's state updated via callback passed outside

```jsx
const useModel = () => {
  const [state, setState] = useState(false);
  const toggle = () => setState(!state);
  return { toggle };
};

const A = () => {
  const { toggle } = useModel();
  // toggle belongs to A's instance
  // calling it → A re-renders!
  return <button onClick={toggle}>Open</button>;
};
```

### 4. Parent re-renders (hook doesn't matter here)

```jsx
const App = () => {
  const [count, setCount] = useState(0); // App state changes
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Click</button>
      <A /> {/* A re-renders because App re-rendered! */}
    </div>
  );
};

const A = () => {
  useModel(); // hook doesn't save A here!
  return <div>Hello</div>;
};
```

### 5. Hook uses Context internally

```jsx
const useTheme = () => {
  const theme = useContext(ThemeContext); // consuming context!
  return theme;
};

const A = () => {
  useTheme(); // when ThemeContext value changes → A re-renders!
  return <div>Hello</div>;
};
```

---

## ❌ Will NOT Cause Re-render

### 1. Hook called but setState never triggered

```jsx
const useModel = () => {
  const [isOpen, setIsOpen] = useState(false);
  const toggle = () => setIsOpen(!isOpen);
  return { toggle };
};

const A = () => {
  useModel(); // called but toggle never triggered → NO re-render!
  return <div>Hello</div>;
};
```

### 2. Hook uses useRef instead of useState

```jsx
const useModel = () => {
  const isOpen = useRef(false); // useRef → no re-render!
  const toggle = () => {
    isOpen.current = !isOpen.current;
  };
  return { toggle };
};

const A = () => {
  const { toggle } = useModel();
  return <button onClick={toggle}>Click</button>; // NO re-render!
};
```

### 3. Another component's hook setState triggers

```jsx
const A = () => {
  const { toggle } = useModel(); // A's instance
  return <button onClick={toggle}>Click</button>;
};

const B = () => {
  useModel(); // B's instance → completely isolated!
  // A's toggle triggers → B NEVER re-renders! ❌
  return <div>Hello</div>;
};
```

### 4. Hook return value not used (but setState never called)

```jsx
const useCounter = () => {
  const [count, setCount] = useState(0);
  return { count, setCount };
};

const A = () => {
  useCounter(); // return value ignored AND setCount never called
  return <div>Hello</div>; // NO re-render! ❌
};
```

---

## Summary Table

| Scenario                                        | Re-renders?              |
| ----------------------------------------------- | ------------------------ |
| Hook's setState triggered                       | ✅ Yes                   |
| Hook called, setState never triggered           | ❌ No                    |
| Return value used or not (setState triggered)   | ✅ Yes — doesn't matter! |
| Return value not used, setState not triggered   | ❌ No                    |
| Another component triggers same hook's setState | ❌ No — isolated!        |
| Hook uses useRef instead of useState            | ❌ No                    |
| Hook uses useContext, context value changes     | ✅ Yes                   |
| Parent re-renders                               | ✅ Yes — always!         |

---

## Golden Rules 🏆

```
1. Hook instances are ISOLATED per component
   → A's hook state never affects B's hook state

2. Hook re-renders its HOST component
   → Hook has no life of its own
   → It attaches to whichever component calls it

3. Just calling a hook is NOT enough to cause re-render
   → setState must actually be TRIGGERED

4. useRef = no re-render
   → Use when you don't need visual updates

5. useContext inside hook = re-renders on context change
   → Context is SHARED, unlike useState

6. Return value doesn't matter
   → Even if you don't use it, setState triggers re-render
```

---

## Quick Decision Chart

```
Hook called in component
        ↓
Is setState triggered?
   ↙         ↘
 YES           NO
  ↓             ↓
Re-render ✅   No re-render ❌
```

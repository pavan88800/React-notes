export default function App() {
const [isOpen, setIsOpen] = useState(false); // 1️⃣ hooks run first

return ( // 2️⃣ THEN JSX returns

<div>
<ScrollableWithMovingBlock />
<button onClick={() => setIsOpen((prev) => !prev)}>Model</button>
{isOpen && <Model />}
<VerySlowComponent />
<BunchOfstuff />
<OtherStuffAlsoComplicated />
</div>
);
}

```

---

## Execution Order
```

1. App function called
2. useState runs → gives isOpen, setIsOpen
3. return statement hits
4. React starts creating JSX objects top to bottom:
   → ScrollableWithMovingBlock JSX created
   → button JSX created
   → isOpen && Model JSX created
   → VerySlowComponent JSX created
   → BunchOfstuff JSX created
   → OtherStuffAlsoComplicated JSX created
5. Returns complete JSX tree to React
6. React renders it to DOM

```

---

## Key Point 🎯
```

JSX is NOT rendered one by one!
→ App function runs completely first
→ creates ALL JSX objects
→ returns the WHOLE tree at once
→ then React updates the DOM

That why its causing delay to load model( means html to render on DOM)

---

This is literally just:

{
type: "div",
props: {
children: [
{ type: VerySlowComponent, props: {} }, // object!
{ type: BunchOfstuff, props: {} }, // object!
{ type: OtherStuffAlsoComplicated, props: {} } // object!
]
}
}

```

---

## So when parent re-renders:
```

Parent re-renders
→ creates BRAND NEW objects for all children
→ React sees new references
→ re-renders all children! ❌

```

## When parent NEVER re-renders:
```

Parent never re-renders
→ same objects in memory forever
→ React sees same references
→ skips re-rendering children! ✅

---

Golden Rule 🎯

Every JSX element = JavaScript object. New render = new object = re-render. Same object reference = skip re-render!

## Full flow every time state changes:

```js
setState called
→ component re-renders
→ entire function runs again
→ NEW JSX objects created
→ React compares with previous JSX (reconciliation)
→ updates only what changed in real DOM
```

const App = () => {
const [count, setCount] = useState(0);

// Every time count changes
// this entire function runs again
// ALL of these are brand new objects!
return (

<div>
<VerySlowComponent /> {/_ new object every render! _/}
<BunchOfstuff /> {/_ new object every render! _/}
<p>{count}</p> {/_ new object every render! _/}
</div>
);
};

```

---

## But React is smart! 🧠

Even though new JSX objects are created — React doesn't blindly re-render everything. It **compares** old vs new:
```

same type + same props → skip re-render ✅
same type + different props → re-render ✅
different type → unmount old, mount new ✅

```

---

## Golden Rule 🎯
```

State changes → re-render → new JSX objects created
BUT
new JSX objects ≠ always visual update
React reconciliation decides what actually changes!

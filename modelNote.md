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

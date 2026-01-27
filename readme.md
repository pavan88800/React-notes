## Keep in mind when design a component you need to think about there state, props, and what does the component do

## 🔑 Core rule (memorize this)

Child components should NOT decide state or business logic.
They should only notify the parent of events.

Parent = decides
Child = emits

---

## In React, state and business logic should live in the parent. Child components should be stateless or only manage local UI state and communicate via callbacks.

## example only:-

## This is exactly what you already did 👍

```js
<PaginationButton value={value} onclick={handleClick} />
```

Child:

```js
onClick={() => onclick(value)}
```

Child does NOT know about setPageCount

Child does NOT decide page logic

Child just sends value

---

One nuance (don’t get confused)

## Child can have local UI state:

```js
const Button = () => {
  const [hovered, setHovered] = useState(false); // ✅ OK
};
```

Local UI state is fine.
Business / shared state is NOT.

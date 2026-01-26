## Keep in mind when design a component you need to think about there state, props, and what does the component do

## 🔑 Core rule (memorize this)

Child components should NOT decide state or business logic.
They should only notify the parent of events.

Parent = decides
Child = emits

---

One nuance (don’t get confused)

## Child can have local UI state:

```js
const Button = () => {
  const [hovered, setHovered] = useState(false); // ✅ OK
};
```

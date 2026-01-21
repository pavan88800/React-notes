## React: Passing Values & Emitting Events (Parent ↔ Child)

### Core Principle

> **If a component receives a value, it should be the one that uses that value to emit events.**

---

### Responsibility Split

**Parent Component**

- Owns **state** and **business logic**
- Decides _what happens_ when an event occurs
- Passes **data** and **function references**

**Child Component**

- Owns its **identity** (e.g., `value`)
- Handles **UI & interactions**
- Emits **intent upward** by calling callbacks with its value

---

### Recommended Pattern (Best Practice)

**Parent**

```jsx
<Rating value={value} isFilled={value <= filled} onSelect={handleSelect} />
```

**Child**

```jsx
const Rating = ({ value, onSelect }) => (
  <button onClick={() => onSelect(value)} />
);
```

**Why this works well**

- Clear ownership (state vs interaction)
- Child is reusable
- Parent stays clean
- Scales well

---

### What to Avoid (Design Smell)

```jsx
<Rating value={value} onClick={() => handleSelect(value)} />
```

**Issues**

- Parent pre-binds identity
- Child becomes passive
- Duplicate responsibility

> ⚠️ Works, but not clean design

---

### When Pre-Binding Is OK

```jsx
<Button onClick={saveForm} />
```

Use this when:

- No identity is needed
- Action is generic

---

### Golden Rule

> **State flows down, events flow up.**

```
Props (data)   ↓
Parent  ───────▶  Child

Events (intent) ↑
Parent  ◀───────  Child
```

---

### Interview One-Liner

> "I keep state in the parent, pass function references down, and let the child emit events using its own identity."

---

### Why This Matters

- Cleaner architecture
- Better reusability
- Easier testing
- Senior-level React thinking

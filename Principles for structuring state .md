## 1️⃣ Group related state

What it means: If two or more pieces of state are logically connected and usually change together, put them in a single state object.

Why: Keeps updates consistent and reduces bugs.

Example:

```js
// ❌ separate states
const [firstName, setFirstName] = useState("");
const [lastName, setLastName] = useState("");

// ✅ grouped
const [user, setUser] = useState({ firstName: "", lastName: "" });
```

---

## 2️⃣ Avoid contradictions in state

What it means: Your state should never disagree with itself. Avoid storing conflicting information.

Why: Contradictory state can break your UI or logic.

Example:

```js
// ❌ contradictory
const [isLoggedIn, setIsLoggedIn] = useState(true);
const [user, setUser] = useState(null);

// ✅ consistent
const [user, setUser] = useState(null);
const isLoggedIn = user !== null; // derived
```

---

## 3️⃣ Avoid redundant state

What it means: If you can calculate some information from the component’s props or its existing state variables during rendering, you should not put that information into that component’s state..

## Avoid redundant state means:

👉 If some information already exists in state or props, don’t store it again in another state variable.

Why: Redundant state is unnecessary and can cause bugs.

Example:

```js
// ❌ redundant
const [input, setInput] = useState("");
const [selectedText, setSelectedText] = useState(input.slice(0, cursor));

// ✅ derived
const selectedText = input.slice(0, cursor); // no need for extra state
```

---

## 4️⃣ Avoid duplication in state

What it means: Don’t store the same data in multiple state variables or nested objects.

Why: Hard to keep in sync; leads to bugs.

Example:

```js
// ❌ duplicate
const [username, setUsername] = useState("");
const [formData, setFormData] = useState({ username: "" });

// ✅ single source of truth
const [formData, setFormData] = useState({ username: "" });
```

---

## 5️⃣ Avoid deeply nested state

What it means: Don’t make your state too hierarchical or deeply nested.

Why: Updating nested objects is harder and can lead to mistakes. Prefer flat structures.

Example:

```js
// ❌ deep nesting
const [data, setData] = useState({ user: { profile: { name: "", age: 0 } } });

// ✅ flatter structure
const [profile, setProfile] = useState({ name: "", age: 0 });
```

---

## 💡 Summary:

Group related state → keep connected data together.

Avoid contradictions → don’t let state disagree with itself.

Avoid redundant state → compute what you can instead of storing it.

Avoid duplication → don’t store the same data multiple times.

Avoid deep nesting → keep state flat for easier updates.

---

## NOTE:

## What is Derived:-

Derived values are values that can be calculated from other state or props instead of being stored separately.

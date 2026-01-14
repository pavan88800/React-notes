-- React Accordion Component - README
Overview
This is an accessible accordion component built with React that supports both single and multiple panel expansion modes, along with keyboard navigation for improved accessibility.
Features

Single/Multiple Mode: Toggle between allowing one panel open at a time or multiple panels
Keyboard Navigation: Navigate between accordion items using arrow keys
Accessible: Uses proper ref management for focus control
Reusable: Configurable via props

---

Mental Model & Design Decisions
State Management Approach
Current Approach (Object-based state):

---

const [accordion, setAccordion] = useState({});
// State structure: { 0: true, 2: true } - indices of open panels

```

**Why this is better than separate state for keyboard navigation:**

#### 1. **Single Source of Truth**
- One state (`accordion`) manages which panels are open
- Refs (`buttonRef`) handle only DOM references for focus management
- Clear separation of concerns: state for data, refs for DOM manipulation

#### 2. **Simplicity & Predictability**
- You don't need to sync multiple state variables
- No risk of state getting out of sync (e.g., keyboard state saying panel 2 is focused but accordion state showing panel 1 is open)
- Easier to debug - just check one state object

#### 3. **Better Performance**
- Using refs for keyboard navigation doesn't trigger re-renders
- State updates only happen when panels open/close, not on every key press
- React's ref system is designed exactly for this use case: accessing DOM elements without causing re-renders

#### 4. **React Best Practices**
- **Refs are for**: Accessing DOM nodes, managing focus, integrating with third-party libraries
- **State is for**: Data that affects what gets rendered
- This approach follows React's philosophy perfectly

### How It Works
```

User presses ArrowDown
↓
handleKeyDown fires
↓
Accesses buttonRef.current[nextIndex]
↓
Calls .focus() on next button
↓
Browser moves focus (NO re-render needed!)

```

Compare this to a hypothetical state-based approach:
```

User presses ArrowDown
↓
handleKeyDown fires
↓
setFocusedIndex(nextIndex) ← State update
↓
Component re-renders ← Unnecessary!
↓
useEffect runs to move focus
↓
Browser moves focus

---

Index-Based State Structure

---

// Why object with numeric keys instead of array?
const accordion = { 0: true, 2: true } // ✅ Good
const accordion = [true, false, true] // ❌ Less flexible

// Benefits:
// 1. Easy to check if specific index is open: accordion[index]
// 2. Easy to toggle: { ...prev, [index]: !active }
// 3. Sparse data structure - only stores open panels
// 4. Works seamlessly with both single/multiple modes

```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `items` | Array | `accordionData` | Array of accordion items with `id`, `title`, and `content` |
| `type` | String | `"multiple"` | `"single"` or `"multiple"` - controls panel expansion behavior |

## Keyboard Navigation

| Key | Action |
|-----|--------|
| `ArrowUp` | Move focus to previous accordion button |
| `ArrowDown` | Move focus to next accordion button |
| `Enter/Space` | Toggle the focused accordion panel |

## Code Structure
```

App Component (Container)
├── State: accordion object tracking open panels
├── Ref: buttonRef array for DOM access
├── handleClick: Manages panel open/close logic
└── handleKeyDown: Manages keyboard navigation

AccordionItem Component (Presentation)
├── Receives all logic via props
├── Renders button and content panel
└── Attaches event handlers and refs

---

## Usage Example

import App from './App';

// Multiple panels can be open
<App type="multiple" />

// Only one panel can be open at a time
<App type="single" />

// Custom items
const customItems = [
{ id: 1, title: "Custom Title", content: "Custom content..." }
];
<App items={customItems} type="multiple" />

---

## Key Takeaways

Use refs for DOM manipulation - They're perfect for managing focus without causing re-renders

Use state for data - Only for information that affects what renders on screen

Separate concerns - State handles business logic, refs handle DOM interactions

Object-based indexing - Flexible and efficient for managing open/closed states

Accessibility first - Keyboard navigation makes the component usable for everyone

---

Common Pitfalls to Avoid

## ❌ Don't do this:

```js
// Storing focus state unnecessarily
const [focusedIndex, setFocusedIndex] = useState(0);

useEffect(() => {
  buttonRef.current[focusedIndex]?.focus();
}, [focusedIndex]);
```

## ✅ Do this instead:

```js
// Direct DOM manipulation via refs
function handleKeyDown(e, index) {
  if (e.key === "ArrowDown") {
    buttonRef.current[index + 1]?.focus();
  }
}
```

## Remember: The best code is often the simplest code that solves the problem effectively. This accordion uses the right tool for each job - state for data, refs for DOM access.

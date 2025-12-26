# Stop Prop Explosion — Start Composing Components (React)

## 🚫 Problem: Prop Explosion

In React, a common beginner mistake is creating **one large component with too many props**.

As features grow, the number of props keeps increasing, making the component:

- Hard to read
- Hard to scale
- Hard to maintain
- Difficult to reason about

This pattern leads to **prop explosion**, where a component becomes tightly coupled to too many responsibilities.

---

## ✅ Better Approach: Component Composition

Instead of passing everything via props, use **composition** to build components.

Composition allows you to:

- Clearly express UI structure using JSX
- Build flexible and reusable components
- Extend functionality without breaking existing APIs

Rather than _telling_ a component what to render through props,
you _show_ the structure directly in JSX.

---

## 🌍 Real-World Examples

This approach is widely used in popular UI libraries:

- Ant Design
- Radix UI
- shadcn/ui
- Headless UI

These libraries favor **composable APIs** over large prop-based configurations.

---

## 🧠 Junior vs Senior Thinking

### ❌ Junior Way

- One component with many props
- Logic and layout tightly coupled
- Every new feature adds more props

### ✅ Senior Way

- Smaller composable components
- Clear and readable JSX
- Layout is controlled by composition
- Easier to scale and maintain over time

---

## 💡 Rule of Thumb

> If your component needs too many props, it’s probably time to compose.

Clean code isn’t about writing less code —  
it’s about writing **readable, scalable, and maintainable code**.

---

## ⚠️ When to Use Composition

**Best suited for:**

- Cards
- Modals
- Tabs
- Accordions
- Dropdowns
- Layout-heavy components

**Avoid over-composition for:**

- Simple buttons
- Small presentational components
- Minor UI variations

---

## 📝 Key Takeaway

Composition helps you design components that grow **with features**,  
not against them.

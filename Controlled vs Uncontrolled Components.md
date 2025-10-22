## Controlled vs Uncontrolled Components in React

1. Controlled Components

In a controlled component, the state is controlled by the parent.

The child component just receives the value via props and notifies the parent on changes.

## In short:

Controlled Component = child input controlled by parent state

---

2. Uncontrolled Components

In an uncontrolled component, the component manages its own state internally.

The parent does not control the value; you can access it via a ref when needed.

In short:
Uncontrolled Component = child input manages its own state

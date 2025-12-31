## 1️⃣ items vs data — what’s the real difference?

Short answer: both work, but they communicate different intent.

Let’s break it down properly.

## Option A: items ✅ (recommended for UI components)

```js
<Tab items={items} />
```

## What items communicates:-

“This component expects a list of UI items it knows how to render.”

## items implies:

A collection

Structured specifically for this component

UI-oriented (title, description, icon, etc.)

Example:

```js
const items = [
  { id: 1, title: "HTML", description: "..." },
  { id: 2, title: "CSS", description: "..." }
];
```

This reads naturally:

“Tabs renders these items.”

---

## Why seniors prefer items

Clear intent

Self-documenting

Matches many UI libraries (Menu items, List items, Dropdown items)

Easier for other devs to understand without reading docs

---

## Option B: data ⚠️ (generic, less expressive)

```js
<Tab data={items} />
```

## What data communicates

“Here is some raw data. Do whatever you want with it.”

## This is:

Very generic

Lacks UI meaning

Often used in low-level or data-driven components

---

## Where data does make sense

```js
<Table data={rows} />
<Chart data={series} />
<VirtualList data={records} />

```

These components:

Don’t care about what the data represents

Only care about how much and how to iterate

---

Senior rule of thumb (important)

If the prop represents domain/UI concepts, use a descriptive name (items, tabs, options).

If the prop represents raw, generic input, use data.

For Tabs:

✅ items
❌ data

---

## 2️⃣ Why value (even when it’s an index)

This is separate but related.

What value means in React

value = the current state of a controlled component

It does not mean:

string only

input only

text only

It can be:

number (index)

string (id)

object

enum

---

Why not activeIndex?

```js
<Tab activeIndex={index} />
```

This:

Leaks internal implementation

---

## But this:

```js
<Tab value={index} />
```

Says:

“This is the current value of the Tabs component.”

---

Tabs doesn’t care what the value represents — only that it can compare it.

---

## Industry proof (real libraries)

Radix UI

```js
<Tabs value="account" onValueChange={setValue} />
```

Value is a string, not an index — still called value.

---

## React itself

```js
<input value={text} onChange={...} />

```

Same mental model.

---

## 3️⃣ The big design principle (this is gold)

## ❌ Bad API (leaks internals)

```js
<Tab activeIndex={index} onActiveIndexChange={setIndex} />
```

---

## ✅ Good API (abstract & flexible)

```js
<Tab items={items} value={index} onChange={setIndex} />
```

Now tomorrow you can switch to:

```js
value={item.id}
```

And the API still makes sense. That’s future-proof design.

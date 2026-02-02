# 🧠 How Browser Decides Which Rendering Steps to Skip

## Browser Rendering Engines

Different browsers use different rendering engines that automatically optimize rendering:

- **Chrome/Edge:** Blink engine
- **Firefox:** Gecko engine
- **Safari:** WebKit engine

---

## 🔄 What Happens When Elements/Styles Change

### When Element Changes (DOM Changes)

**Examples:**

- Adding/removing elements
- Changing text content
- Modifying attributes
- React `setState()` that changes DOM structure

**What Happens:**

```
1. JavaScript/React executes change
   ↓
2. DOM tree is updated (immediate)
   ↓
3. Browser recalculates Render Tree (batched, after JS finishes)
   ↓
4. Layout/Reflow (if element affects geometry)
   ↓
5. Paint (if element is visible)
   ↓
6. Composite (always)
```

**Example:**

```javascript
// React
setState({ items: [...items, newItem] });

// Vanilla JS
const div = document.createElement("div");
document.body.appendChild(div);
```

**Result:** Full rendering pipeline executes

---

### When Style Changes (CSS/CSSOM Changes)

**Examples:**

- Changing inline styles
- Adding/removing CSS classes
- Modifying CSS properties
- React `setState()` that changes styles

**What Happens:**

```
1. JavaScript/React executes change
   ↓
2. CSSOM is updated (immediate)
   ↓
3. Browser recalculates Render Tree (batched, after JS finishes)
   ↓
4. Layout/Reflow (depends on property - see below)
   ↓
5. Paint (depends on property - see below)
   ↓
6. Composite (always)
```

**Steps 4-6 depend on WHICH CSS property changed** (see next section)

**Example:**

```javascript
// React
setState({ color: "red" });

// Vanilla JS
element.style.color = "red";
element.classList.add("active");
```

**Result:** Depends on which CSS properties changed

---

### Key Differences

| Change Type        | What Updates | Rendering Pipeline                                 |
| ------------------ | ------------ | -------------------------------------------------- |
| **Element change** | DOM tree     | Usually full pipeline (Layout + Paint + Composite) |
| **Style change**   | CSSOM tree   | Depends on CSS property (may skip steps)           |

---

### Important: Browser Batching

The browser is smart and **batches multiple changes**:

```javascript
// All these happen immediately:
element.style.width = "100px"; // CSSOM update
element.style.height = "200px"; // CSSOM update
element.style.color = "red"; // CSSOM update

// Browser waits for JavaScript to finish, then does:
// ONE Render Tree recalculation
// ONE Layout (if needed)
// ONE Paint
// ONE Composite

// NOT 3 separate render cycles!
```

---

## 🔍 How Browser Decides

The browser analyzes what changed and decides which steps to execute:

```
Browser analyzes the change:
  ↓
Is it a GEOMETRY property? (width, height, margin, etc.)
  → YES: Do Layout + Paint + Composite
  → NO: Check next...
  ↓
Is it a VISUAL property? (color, background, etc.)
  → YES: Skip Layout, do Paint + Composite
  → NO: Check next...
  ↓
Is it a COMPOSITE property? (opacity, transform, etc.)
  → YES: Skip Layout + Paint, only Composite
```

---

## 📊 Browser's Internal Rules

The browser has **built-in knowledge** of which CSS properties affect what:

### Properties that trigger LAYOUT

```
width, height, margin, padding, border
top, left, right, bottom, position
display, float, flex properties
font-size (affects text layout)
```

**→ Browser does:** Layout + Paint + Composite

---

### Properties that trigger PAINT only

```
color, background, background-color
border-color, box-shadow, outline
visibility, text-decoration
```

**→ Browser does:** Paint + Composite (skips Layout)

---

### Properties that trigger COMPOSITE only

```
opacity, transform, filter
```

**→ Browser does:** Composite only (skips Layout + Paint)

---

## 🔍 Examples - How Browser Decides

### Example 1: Geometry Change

```javascript
element.style.width = "500px";

// Browser thinks:
// "width changed → affects geometry → need Layout + Paint + Composite"
```

**Result:** All 3 phases execute

---

### Example 2: Visual Change

```javascript
element.style.color = "red";

// Browser thinks:
// "color changed → only visual → skip Layout, do Paint + Composite"
```

**Result:** Only Paint + Composite

---

### Example 3: Composite Change

```javascript
element.style.transform = "translateX(100px)";

// Browser thinks:
// "transform changed → GPU can handle → skip Layout + Paint, only Composite"
```

**Result:** Only Composite (fastest!)

---

## ⚙️ Behind The Scenes

When you change a style, the browser:

```
1. Marks the element as "dirty" (needs update)
2. Checks which properties changed
3. Looks up property in internal table
4. Determines which phases are needed
5. Schedules appropriate rendering phases
6. Executes only necessary phases
```

---

## 🎯 Key Point

**YOU (developer) DON'T control the skipping**

**THE BROWSER automatically optimizes based on:**

- Which CSS properties you changed
- Its internal knowledge of what each property affects
- Current state of the render tree

---

## 💡 What You Can Do

You can **write code** that helps the browser skip more steps:

### ❌ BAD (forces expensive operations):

```javascript
element.style.width = "500px"; // Layout + Paint + Composite
```

### ✅ GOOD (cheaper operation):

```javascript
element.style.transform = "scaleX(1.5)"; // Only Composite
```

---

## 🔗 Useful Resource

**Check what each CSS property triggers:**

👉 **https://csstriggers.com/**

---

## 📊 Summary Table

| Who decides         | What they decide                                       |
| ------------------- | ------------------------------------------------------ |
| **Browser**         | Which steps to skip based on CSS properties changed    |
| **You (developer)** | Which CSS properties to use (to help browser optimize) |

---

## 🚀 Performance Tips

1. **Use `transform` and `opacity` for animations** (only triggers Composite)
2. **Avoid changing geometry properties frequently** (width, height, margin)
3. **Batch DOM/style changes together** (browser optimizes batched changes)
4. **Use CSS classes instead of inline styles** when possible
5. **Check csstriggers.com** before optimizing animations

---

**Remember:** The browser is smart and automatically optimizes rendering. Your job is to write code that makes optimization easier!

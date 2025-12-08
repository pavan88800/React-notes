// PARENT: Manages state and coordination

```js
function Parent() {
  const [state, setState] = useState();

  const handleChildAction = (childIndex, data) => {
    // Parent logic here
  };

  return items.map((item, index) => (
    <Child
      key={index}
      value={item} // ✅ Only this child's data
      isActive={index === activeIndex} // ✅ Derived boolean
      onAction={(data) => handleChildAction(index, data)} // ✅ Ready callback
    />
  ));
}

// CHILD: Handles its own UI and events
function Child({ value, isActive, onAction }) {
  const handleEvent = (e) => {
    // Process event
    onAction(processedData); // Tell parent
  };

  return <div onClick={handleEvent}>{value}</div>;
}
```

---

## 🎯 Summary - The 3 Rules

### Rule 1: **Single Responsibility**

Each component does ONE thing well

### Rule 2: **Minimal Props**

Child only gets what it NEEDS

### Rule 3: **Communication Through Callbacks**

Child tells parent, parent decides

---

## 💡 Your OTP Component - Quick Rules

```

PARENT OTP:
✅ Store: otp array, focusedIndex
✅ Handle: completion, validation, paste
✅ Provide: callbacks that are ready to use

CHILD OtpInput:
✅ Handle: single digit display
✅ Handle: keyboard events for THIS box
✅ Tell parent: when value changes, when navigation needed
❌ Don't know: other boxes, index, total OTP
```

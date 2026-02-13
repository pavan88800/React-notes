##🔹 useMemo

If dependencies change →
👉 executes the function during render
👉 returns new computed value

With []:

Executes once on initial render

Never executes again (unless unmounted/remounted)

---

## 🔹 useCallback

If dependencies change →
👉 creates a new function reference
👉 function body does NOT execute

Function body executes only when you call it (click, hover, etc.)

With []:

Same function reference forever

Executes only when invoked

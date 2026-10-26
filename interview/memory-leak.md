# Memory Leaks in React — Interview Answer

> **Question:** "What is a memory leak in React, and how would you prevent it?"

## 1. Answer to say (~60 seconds)

"A memory leak is memory that can't be garbage-collected because something still references it, even though the app no longer needs it.

In React, the usual cause is a **side effect that outlives its component**. Examples are a timer, a `window` listener, a subscription, a socket or an observer. Each of these holds a callback. That callback is a closure over the component's props, state and DOM nodes, so after unmount none of that memory can be freed.

I prevent leaks by giving **every effect a matching cleanup**. I clear timers, remove listeners, unsubscribe, close sockets, disconnect observers, abort requests with `AbortController`, and destroy third-party instances like charts or maps.

Strict Mode helps in development because it mounts, unmounts and remounts components, which surfaces missing cleanups early. To confirm a leak, I use Chrome DevTools heap snapshots: I mount and unmount a screen several times and check whether memory or detached DOM nodes keep growing."

## 2. Mental model

**Garbage collector rule:** memory is freed only when _nothing_ can reach it.

```
window ──► resize listener ──► closure ──► component state, props, DOM nodes
   ▲
   └── window lives forever, so everything on the right lives forever too
```

The cleanup function cuts that arrow.

**Analogy:** cleanup is checking out of a hotel. If you leave without returning the key (an unremoved listener), the room stays booked under your name, even though you're gone.

## 3. Common causes

| Cause                                                          | Long-lived holder            | Cleanup                                         |
| -------------------------------------------------------------- | ---------------------------- | ----------------------------------------------- |
| `setInterval` / `setTimeout`                                   | Browser timer queue          | `clearInterval` / `clearTimeout`                |
| Event listeners on `window` / `document`                       | Global object                | `removeEventListener` (same function reference) |
| Subscriptions (store, event emitter, RxJS)                     | The store / emitter          | `unsubscribe()`                                 |
| WebSocket / `EventSource`                                      | Open connection              | `socket.close()`                                |
| `ResizeObserver` / `IntersectionObserver` / `MutationObserver` | Browser observer registry    | `observer.disconnect()`                         |
| In-flight fetch                                                | Pending promise + callbacks  | `AbortController.abort()`                       |
| Third-party widgets (Chart.js, Leaflet, Mapbox, editors)       | Library internals + DOM      | `instance.destroy()` / `.remove()`              |
| Module-level caches (`const cache = new Map()`)                | Module scope (lives forever) | Cap size, LRU eviction, or `WeakMap`            |
| Refs holding large data or DOM nodes in a long-lived parent    | Parent component             | Set `ref.current = null` when done              |

## 4. Prevention patterns with code

### Event listener

```tsx
useEffect(() => {
  const handleResize = () => setWidth(window.innerWidth);
  window.addEventListener("resize", handleResize);
  return () => window.removeEventListener("resize", handleResize);
}, []);
```

The cleanup must pass the **same function reference**. Removing a new inline arrow function does nothing:

```js
window.removeEventListener("resize", () => {}); // no-op: different function
```

### Timer

```tsx
useEffect(() => {
  const id = setInterval(() => setSeconds((s) => s + 1), 1000);
  return () => clearInterval(id);
}, []);
```

The functional update `s => s + 1` also avoids a stale closure.

### Fetch with AbortController

```tsx
useEffect(() => {
  const controller = new AbortController();

  fetch(`/api/flights/${id}`, { signal: controller.signal })
    .then((res) => res.json())
    .then(setFlight)
    .catch((err) => {
      if (err.name !== "AbortError") setError(err.message);
    });

  return () => controller.abort(); // unmount or id changed → cancel
}, [id]);
```

This also prevents **race conditions**: a slow old response can't overwrite a newer one.

### Subscription / WebSocket

```tsx
useEffect(() => {
  const socket = new WebSocket(url);
  socket.onmessage = (e) => setMessages((m) => [...m, JSON.parse(e.data)]);
  return () => socket.close();
}, [url]);
```

### Observer

```tsx
function useElementWidth(ref: React.RefObject<HTMLElement>) {
  const [width, setWidth] = useState(0);

  useEffect(() => {
    if (!ref.current) return;
    const observer = new ResizeObserver(([entry]) =>
      setWidth(entry.contentRect.width)
    );
    observer.observe(ref.current);
    return () => observer.disconnect();
  }, [ref]);

  return width;
}
```

### Third-party library instance

```tsx
useEffect(() => {
  const chart = new Chart(canvasRef.current!, config);
  return () => chart.destroy();
}, [config]);
```

## 5. Common misconception (senior signal)

Older React showed this warning: _"Can't perform a React state update on an unmounted component… indicates a memory leak."_

React 18 **removed** that warning. Calling `setState` after unmount is a harmless no-op, and the old `isMounted` flag workaround doesn't fix anything. The real leak is the long-lived source, such as `window`, an interval or a socket, that still holds the callback. The fix is to clean up or cancel that source.

## 6. How to detect a leak

1. Chrome DevTools → **Memory** tab → take a heap snapshot.
2. Mount and unmount the suspect screen 5–10 times (navigate in and out).
3. Click the trash icon to force garbage collection, then take a second snapshot.
4. Switch to **Comparison** view and look for objects whose count keeps growing.
5. Filter by **"Detached"** to find DOM nodes removed from the page but still referenced.
6. For a quick signal, open DevTools → **Performance monitor** and watch whether JS heap size keeps rising and never drops.

Other aids:

- **Strict Mode**, because the dev-only double mount exposes missing cleanups.
- The ESLint rule `react-hooks/exhaustive-deps` catches wrong dependencies that cause stale or duplicate subscriptions.

## 7. Likely follow-up questions

**When does the cleanup function run?**
It runs before the effect re-runs (when dependencies change) and when the component unmounts.

**Why does Strict Mode run effects twice?**
To verify that mount → cleanup → mount leaves the app in the same state. If something breaks or duplicates, the cleanup is missing.

**Do closures cause leaks by themselves?**
No. A closure only keeps memory alive while something long-lived references the closure. Remove that reference and the garbage collector frees it.

**`WeakMap` vs `Map` for caches?**
A `WeakMap` doesn't stop its keys from being garbage-collected, so entries disappear when the key object is no longer used. A `Map` keeps everything until you delete it.

**Can a memory leak happen without `useEffect`?**
Yes. Module-level caches, global stores that grow forever, and objects attached to `window` all leak regardless of component lifecycle.

## 8. One-line summary

> Every effect that starts something long-lived must stop it in its cleanup. Then verify with heap snapshots, not guesses.

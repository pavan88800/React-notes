# Synechron React.js Interview — Round-wise Answers

Each question below has a speakable answer (first person, 30–90 seconds) plus code where the interviewer would expect it. The pattern they reward: **what I used → why → a real example → the trade-off**. Examples lean on AirOps (React, TypeScript, Redux) — swap in your own specifics.

## Round 1 — React.js concepts

### 1. What React features have you used in real projects?

"On AirOps — an aviation ops platform for helicopter operators — I've used function components with hooks throughout, Redux Toolkit for shared server-ish state like flights, crew and aircraft, and Context for app-wide config like auth and theme. I've built custom hooks for data fetching, forms and permissions, used `React.lazy` + `Suspense` for route-level code splitting, error boundaries around risky widgets, portals for modals and toasts, and `useMemo`/`React.memo` on heavy tables like schedules. With TypeScript, every component's props and every Redux slice are typed."

**Tip:** name 5–6 features, then pick _one_ and go deep with a story (problem → what you did → result).

### 2. Which hooks do you use and when?

| Hook                                 | When I reach for it                                                                                   |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------- |
| `useState`                           | Local UI state: toggles, inputs, open/closed                                                          |
| `useReducer`                         | Several related state fields or transitions (multi-step forms, wizards)                               |
| `useEffect`                          | Syncing with something _outside_ React: subscriptions, timers, DOM APIs, fetching without a library   |
| `useRef`                             | Mutable value that shouldn't re-render (timer ids, previous value, DOM node)                          |
| `useMemo`                            | Expensive derived data (filter/sort a 2,000-row table) or stable object identity for a memoized child |
| `useCallback`                        | Stable function identity when passed to a `React.memo` child or a hook dependency array               |
| `useContext`                         | Reading app-wide, rarely-changing values (auth user, theme, locale)                                   |
| `useLayoutEffect`                    | Measuring DOM before paint (tooltip positioning) to avoid flicker                                     |
| `useTransition` / `useDeferredValue` | Keeping typing responsive while a big list re-filters                                                 |
| `useId`                              | Accessible label/input ids that match between server and client                                       |

One-liner to say: "If it's not syncing with an external system, it probably doesn't need `useEffect` — I derive it during render instead."

### 3. Have you created custom hooks? Explain one.

"Yes. A custom hook is just a function starting with `use` that composes other hooks so components share _logic_, not state. One I wrote is `useDebouncedValue`, used in our flight search so we don't fire an API call on every keystroke."

```tsx
import { useEffect, useState } from "react";

export function useDebouncedValue<T>(value: T, delay = 300): T {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id); // cancel if value changes before delay
  }, [value, delay]);

  return debounced;
}

// usage
const debouncedQuery = useDebouncedValue(query, 400);
useEffect(() => {
  if (debouncedQuery) search(debouncedQuery);
}, [debouncedQuery]);
```

Points to mention: each component calling it gets its _own_ state; the cleanup is what makes the debounce work; it's generic and typed.

### 4. Which React version are you currently using?

Say the exact version from `package.json` and why. Template: "We're on React 18.x. We've evaluated 19 — the blocker is <a dependency / testing setup>, and our upgrade plan is <plan>." If you're on 19: "React 19 — we upgraded in <month>; the biggest win was <ref as prop / Actions / compiler>." Never guess — interviewers follow up.

### 5. Major changes in newer React versions

**React 18**

- Concurrent rendering via `createRoot` (opt-in by using concurrent features).
- Automatic batching — state updates inside promises, timeouts and native handlers are batched too, not just React events.
- `useTransition`, `useDeferredValue` — mark updates as non-urgent.
- Streaming SSR with `Suspense` and selective hydration.
- New hooks: `useId`, `useSyncExternalStore` (for libraries like Redux), `useInsertionEffect`.
- Strict Mode double-invokes effects in dev to surface missing cleanups.

**React 19**

- **Actions**: async functions in transitions; `useActionState`, `useFormStatus`, `useOptimistic` for forms and optimistic UI.
- `use()` — read a promise or context during render (works with Suspense, can be called conditionally).
- `ref` is a regular prop — `forwardRef` no longer needed.
- `<Context>` can be used directly as a provider.
- Native support for `<title>`, `<meta>`, stylesheets and preloading APIs.
- Server Components and Server Actions are stable (used through frameworks like Next.js).
- Better hydration error messages; removed legacy APIs (`propTypes` on functions, string refs, `ReactDOM.render`).

**React Compiler** (shipped separately as a build plugin) auto-memoizes components and values, so much manual `useMemo`/`useCallback` becomes unnecessary. Mention it as "the direction React is going."

### 6. How do you identify and fix unnecessary re-renders?

**Identify**

1. React DevTools Profiler → record an interaction → flame graph shows which components rendered and _why_ ("props changed", "parent rendered", "hooks changed").
2. DevTools setting "Highlight updates when components render".
3. Chrome Performance tab for long tasks; `why-did-you-render` in dev if needed.

**Common causes → fixes**

| Cause                                                      | Fix                                                                   |
| ---------------------------------------------------------- | --------------------------------------------------------------------- |
| Parent re-renders, child props unchanged                   | `React.memo` on the child                                             |
| New object/array/function created every render breaks memo | `useMemo` / `useCallback`, or move constants outside the component    |
| State lives too high in the tree                           | Move state down to where it's used, or pass children as props         |
| One big Context value changes → every consumer re-renders  | Split contexts (state vs dispatch), memoize the value                 |
| Redux selector returns a new reference each time           | Select primitives, use `createSelector` (reselect), or `shallowEqual` |
| Huge lists                                                 | Virtualize (`react-window` / TanStack Virtual), stable `key`s         |
| Expensive filtering on every keystroke                     | `useDeferredValue` / `useTransition`, debounce                        |

Story to tell: "Our crew schedule grid re-rendered every row on any cell edit. Profiler showed the row props were new objects each time. I memoized rows, made selectors return per-row data, and virtualized the list — interaction went from noticeably laggy to instant." (Use your real numbers if you have them.)

And always add: "I measure first. `memo` everywhere has its own cost."

### 7. Redux vs Context API — when would you use each?

Mental model: **Context is a pipe, Redux is a store.** Context only _delivers_ a value down the tree; it has no update logic, no selectors, and every consumer re-renders when the value changes. Redux is a state-management system: central store, reducers, selectors that subscribe to slices, middleware, DevTools time-travel.

|             | Context API                                                             | Redux Toolkit                                                   |
| ----------- | ----------------------------------------------------------------------- | --------------------------------------------------------------- |
| Best for    | Low-frequency, app-wide values: auth user, theme, locale, feature flags | Large, frequently-updated shared state touched by many features |
| Re-renders  | All consumers on any change                                             | Only components whose selected slice changed                    |
| Tooling     | None                                                                    | DevTools, middleware, RTK Query for caching API data            |
| Boilerplate | Minimal                                                                 | More, but RTK reduces it a lot                                  |

Answer to say: "I use Context for things that rarely change, and Redux Toolkit when many screens read and write the same data — like flights and crew in AirOps. For _server_ data I prefer RTK Query or React Query over hand-written reducers, because caching, refetching and loading states come for free. For small apps, Zustand is a lighter middle ground."

## Round 2 — Live coding

The task builds step by step, so write the hook to be _extendable_ from the start. Talk while you type: say what each piece handles before writing it. Typical API: `https://jsonplaceholder.typicode.com/posts`.

### Step 1–3: Custom `useFetch` hook with error handling

```tsx
import { useCallback, useEffect, useState } from "react";

type FetchState<T> = {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => void;
};

export function useFetch<T>(url: string): FetchState<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [reloadKey, setReloadKey] = useState(0);

  useEffect(() => {
    const controller = new AbortController();

    async function load() {
      setLoading(true);
      setError(null);
      try {
        const res = await fetch(url, { signal: controller.signal });
        // fetch only rejects on network failure — 404/500 must be checked manually
        if (!res.ok) throw new Error(`Request failed: ${res.status}`);
        const json: T = await res.json();
        setData(json);
      } catch (err) {
        if ((err as Error).name === "AbortError") return; // unmounted / url changed
        setError((err as Error).message || "Something went wrong");
      } finally {
        if (!controller.signal.aborted) setLoading(false);
      }
    }

    load();
    return () => controller.abort(); // cleanup: cancel stale request
  }, [url, reloadKey]);

  const refetch = useCallback(() => setReloadKey((k) => k + 1), []);

  return { data, loading, error, refetch };
}
```

Things to say out loud (this is what they're grading):

- **`res.ok` check** — `fetch` doesn't throw on HTTP 404/500, a classic trap.
- **`AbortController` in cleanup** — prevents setting state after unmount and race conditions when `url` changes fast.
- **Three states** — loading, error, data — so the UI can render each.
- **`refetch`** — gives a retry button something to call.
- **Generic `<T>`** — the hook doesn't know about posts; the caller decides the type.

### Step 4: Display all titles

```tsx
type Post = { id: number; title: string; body: string };

export function PostTitles() {
  const { data, loading, error, refetch } = useFetch<Post[]>(
    "https://jsonplaceholder.typicode.com/posts"
  );

  if (loading) return <p>Loading…</p>;
  if (error)
    return (
      <div role="alert">
        <p>Error: {error}</p>
        <button onClick={refetch}>Retry</button>
      </div>
    );
  if (!data?.length) return <p>No posts found.</p>;

  return (
    <ul>
      {data.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

Mention: `key={post.id}`, not the array index; handle the empty state too.

### Step 5: Modify the hook to return only the first item

Ask a clarifying question first: "Should the hook _always_ return the first item, or should callers choose?" That question alone scores points. Then offer the clean version — a `select` option, so the hook stays reusable:

```tsx
export function useFetch<T, R = T>(url: string, select?: (data: T) => R) {
  // ...same as before, but on success:
  // const json: T = await res.json();
  // setData(select ? select(json) : (json as unknown as R));
}

// caller decides the shape
const { data: firstPost } = useFetch<Post[], Post | undefined>(
  url,
  (posts) => posts[0]
);
```

If they literally want the hook changed, the minimal edit is `setData(Array.isArray(json) ? json[0] : json)` — but say why the `select` version is better: one hook, many shapes, no duplicated fetch logic. (Wrap `select` in `useCallback` or keep it out of the dependency array, otherwise an inline function re-triggers the effect.)

**Bonus if time allows:** "In production I'd use React Query or RTK Query — they add caching, deduping, retries and stale-while-revalidate. Writing it by hand shows I understand what those libraries solve."

## Round 3 — Testing

### 1. Experience with Jest & React Testing Library?

"Yes — Jest is the test runner and assertion library; RTL renders components and queries them the way a user would. RTL's philosophy is _test behavior, not implementation_: I don't assert on state or internal methods, I assert on what's on screen after a user action. For API calls I mock at the network layer with MSW (Mock Service Worker) so components use their real fetching code."

Be honest about depth. If coverage at work is thin, say: "We focus tests on critical flows like <flight booking / crew assignment>, plus reducers and utils."

### 2. What types of test cases have you written?

| Type                 | What it covers                             | Example                                                    |
| -------------------- | ------------------------------------------ | ---------------------------------------------------------- |
| Unit                 | Pure functions, reducers, selectors, utils | Duty-hour calculator returns correct total across midnight |
| Component            | Render + interaction of one component      | Form shows validation error when required field is empty   |
| Hook                 | Custom hook logic via `renderHook`         | `useFetch` returns error state on 500                      |
| Integration          | Several components + store + mocked API    | Filter a list → API called with params → results rendered  |
| Snapshot (sparingly) | Guarding stable, small UI                  | Rarely — they break on harmless changes                    |
| E2E (outside Jest)   | Full user flow in a real browser           | Playwright / Cypress login → create flight                 |

Also mention the **cases inside each test**: happy path, loading state, error state, empty state, edge inputs, and accessibility (queries by role ensure labels exist).

### 3. How do you test React components?

The flow: **render → find like a user → act like a user → assert what the user sees.**

Query priority: `getByRole` → `getByLabelText` → `getByText` → `getByTestId` (last resort). Use `getBy` when it must exist, `queryBy` to assert absence, `findBy` for async.

Testing the `PostTitles` component from Round 2:

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { PostTitles } from "./PostTitles";

beforeEach(() => {
  global.fetch = jest.fn();
});

it("shows titles after loading", async () => {
  (fetch as jest.Mock).mockResolvedValueOnce({
    ok: true,
    json: async () => [
      { id: 1, title: "First post" },
      { id: 2, title: "Second post" }
    ]
  });

  render(<PostTitles />);

  expect(screen.getByText(/loading/i)).toBeInTheDocument();
  expect(await screen.findByText("First post")).toBeInTheDocument();
  expect(screen.getAllByRole("listitem")).toHaveLength(2);
});

it("shows error and retries", async () => {
  (fetch as jest.Mock)
    .mockResolvedValueOnce({ ok: false, status: 500 })
    .mockResolvedValueOnce({
      ok: true,
      json: async () => [{ id: 1, title: "Recovered" }]
    });

  render(<PostTitles />);

  expect(await screen.findByRole("alert")).toHaveTextContent("500");
  await userEvent.click(screen.getByRole("button", { name: /retry/i }));
  expect(await screen.findByText("Recovered")).toBeInTheDocument();
});
```

Testing the hook directly:

```tsx
import { renderHook, waitFor } from "@testing-library/react";

it("returns error on failed request", async () => {
  (fetch as jest.Mock).mockResolvedValueOnce({ ok: false, status: 404 });
  const { result } = renderHook(() => useFetch("/api/x"));
  await waitFor(() => expect(result.current.loading).toBe(false));
  expect(result.current.error).toMatch(/404/);
});
```

For components using Redux, say: "I write a `renderWithProviders` helper that wraps the component in a fresh store with preloaded state, so each test is isolated."

Phrases that signal seniority: `userEvent` over `fireEvent` (it simulates real typing/clicking), `findBy` instead of manual waits, MSW over mocking `fetch` in larger suites, and "I avoid testing implementation details so refactors don't break tests."

## Round 4 — JavaScript

### 1. Rate yourself in JavaScript out of 5

Say **4**, then justify it immediately: "4 — I'm comfortable with closures, the event loop, promises/async, prototypes, `this`, and ES6+ features, and I use them daily. I'd keep the last point for spec-level depth, like engine internals or every coercion edge case." A 5 invites a trap question; a 3 undersells 4–5 years of experience.

Expect the follow-up to test the rating — be ready for the event-loop output puzzle below.

### 2. Explain closures with an example

**Definition:** a closure is a function that remembers the variables from the scope where it was _created_, even after that outer function has finished running.

**Analogy:** a function carries a backpack. When it's created, it packs references to the variables around it, and it can open that backpack whenever it runs later.

```js
function createCounter() {
  let count = 0; // private — nothing outside can touch it
  return {
    increment: () => ++count,
    get: () => count
  };
}

const counter = createCounter();
counter.increment();
counter.increment();
counter.get(); // 2 — createCounter finished long ago, but count lives on
```

**Practical uses to mention:** data privacy (module pattern), function factories, `debounce`/`throttle`, memoization caches, and event handlers. In React, every hook callback is a closure over that render's props and state.

**The classic trap:**

```js
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 3 3 3
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 0 1 2
```

`var` is function-scoped, so all callbacks close over one `i`; `let` creates a new binding per iteration.

**React tie-in (senior signal) — stale closures:**

```jsx
useEffect(() => {
  const id = setInterval(() => setCount(count + 1), 1000); // always reads count = 0
  return () => clearInterval(id);
}, []);
// fix: setCount(c => c + 1)  — functional update doesn't depend on the closure
```

### 3. Practical JavaScript concepts used in your projects

Pick 4–5 and give one real usage each:

| Concept                                        | How I've used it                                                                                                             |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Closures                                       | `debounce` for search inputs, hook factories                                                                                 |
| Promises / async-await                         | API layer; `Promise.all` to load flight, crew and aircraft data in parallel; `Promise.allSettled` when partial failure is OK |
| Event loop                                     | Explaining why state isn't updated right after `setState`, and ordering of microtasks vs timers                              |
| Destructuring, spread, optional chaining, `??` | Immutable Redux updates, safe access on nested API data                                                                      |
| Array methods (`map`, `filter`, `reduce`)      | Transforming API responses into table rows, grouping flights by date                                                         |
| Debounce / throttle                            | Search fields, scroll and resize handlers                                                                                    |
| Shallow vs deep copy                           | Avoiding accidental mutation; `structuredClone` when a deep copy is needed                                                   |
| `this` / arrow functions                       | Arrow functions in callbacks to avoid lost `this`                                                                            |
| Modules + dynamic `import()`                   | Code splitting heavy screens                                                                                                 |
| Date / time zones                              | Aviation scheduling across time zones — store UTC, display local                                                             |

**Event-loop warm-up they may throw at you:**

```js
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
// Output: 1 4 3 2 — sync first, then microtasks (promises), then macrotasks (timers)
```

## Round 5 — Node.js & AI tools

### 1. What is your experience with Node.js? / To what extent?

They're checking whether you're frontend-only or can work across the stack. Be honest and specific — scope, not adjectives:

"My core is frontend, but I'm comfortable in Node. Day to day I use it for tooling — build scripts, Vite/webpack config, codegen, mock servers. I've built small Express APIs and BFF-style endpoints, and with Next.js I write API routes and server-side data fetching. I understand the Node event loop, non-blocking I/O, streams at a basic level, and middleware patterns. I haven't owned a large production Node service, but I can contribute to one."

Adjust the last line to your real exposure. Concepts they may probe:

- **Why Node is good for I/O-heavy work:** single-threaded event loop + non-blocking I/O handles many concurrent connections; CPU-heavy work blocks it (use worker threads or another service).
- **Express middleware:** functions with `(req, res, next)` run in order — auth, logging, validation, error handler last.
- **CommonJS vs ES modules:** `require`/`module.exports` vs `import`/`export`; `"type": "module"` in `package.json`.
- **Env config & secrets:** `process.env`, never ship secrets to the client bundle.

### 2. Have you used GitHub Copilot or other AI tools?

They want to hear you use AI **with judgment**, not blindly. Structure: tools → where they help → how you verify.

"Yes — I use Copilot in the editor and chat assistants like Claude/ChatGPT. They're great for boilerplate (typed API clients, test scaffolding, Redux slices), converting JS to TS, writing regex or date logic, and explaining unfamiliar code. I treat output like a junior's PR: I read every line, run the tests, and check edge cases and security. I don't paste proprietary code into tools the company hasn't approved. Net effect: I spend less time on repetitive code and more on design and review."

If you've built anything AI-related (a RAG chatbot, an LLM feature in a UI), mention it in one line — it's a differentiator right now. Be ready for: "Give an example where AI was wrong and how you caught it."

## Final checklist

- [ ] Write `useFetch` from memory in under 10 minutes — with `res.ok`, `AbortController`, loading/error/data, `refetch`
- [ ] Extend it with a `select` option without breaking existing callers
- [ ] Write two RTL tests for it (success + error/retry)
- [ ] Know your exact React version and one reason you are (or aren't) on 19
- [ ] One re-render story with a before/after from AirOps
- [ ] One custom hook story from real work
- [ ] Explain closures + stale closures in under 90 seconds
- [ ] Predict output of an event-loop puzzle
- [ ] Redux vs Context answer in two sentences
- [ ] Honest Node.js scope statement
- [ ] AI-tools answer that includes how you verify output

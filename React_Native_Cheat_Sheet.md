# React Native Senior Engineer — Cheat Sheet

> Single-page quick reference across all 3 volumes. Use this for last-minute review before interviews.

---

## Vol 1 — JS / TypeScript / React Hooks

### Scope & Hoisting

| Keyword | Scope | Hoisted | TDZ |
|---|---|---|---|
| `var` | Function | Yes (as `undefined`) | No |
| `let` | Block | Yes | Yes |
| `const` | Block | Yes | Yes |

### `==` vs `===`

```txt
== → allows type coercion   (0 == false → true)
=== → value AND type check  (0 === false → false)
```

**Rule**: Always use `===`.

---

### Event Loop Priority

```txt
1. Call Stack (synchronous)
2. Microtasks (Promise .then, async/await)
3. Macrotasks (setTimeout, setInterval)
```

---

### Promises & Async

| Pattern | When to use |
|---|---|
| Sequential `await` | Each call depends on previous result |
| `Promise.all([...])` | Independent parallel calls |
| `.catch()` | Centralized error handling |

---

### Immutability Cheat Codes

```tsx
// Update object
setUser(prev => ({ ...prev, name: "New" }));

// Nested update
setUser(prev => ({ ...prev, address: { ...prev.address, city: "Toronto" } }));

// Update item in array
setItems(prev => prev.map(item => item.id === id ? { ...item, isSelected: true } : item));

// Remove item
setItems(prev => prev.filter(item => item.id !== id));

// Add item
setItems(prev => [...prev, newItem]);
```

---

### TypeScript Utility Types

| Type | What it does |
|---|---|
| `Partial<T>` | All fields optional |
| `Pick<T, "a" \| "b">` | Select specific fields |
| `Omit<T, "a">` | Remove specific fields |
| `Record<K, V>` | Dictionary type |
| `ReturnType<typeof fn>` | Type of function's return value |

---

### Discriminated Unions (Prefer over flags)

```tsx
// ❌ Don't
type State = { isLoading: boolean; error?: string; data?: User };

// ✅ Do
type State =
  | { status: "loading" }
  | { status: "success"; data: User }
  | { status: "error"; message: string };
```

---

### Hooks Quick Reference

| Hook | Purpose | Triggers re-render? |
|---|---|---|
| `useState` | Local UI state | Yes |
| `useEffect` | Side effects after render | No (side effect only) |
| `useMemo` | Cache expensive value | No |
| `useCallback` | Cache function reference | No |
| `useRef` | Mutable value, DOM ref, stale fix | No |
| `React.memo` | Memoize component | — |

---

### When to Use useMemo / useCallback

```txt
Use them when:
  ✅ Expensive computation (filter, sort on large list)
  ✅ Stable reference passed to React.memo child

Skip them when:
  ❌ Simple value (user.name)
  ❌ Child is not memoized anyway
```

---

### Stale Closure Fixes

```tsx
// Fix 1: use functional updater (no captured value)
setCount(prev => prev + 1);

// Fix 2: add the value to dep array
useEffect(() => { ... }, [count]);

// Fix 3: useRef to always read latest without adding deps
const latestFn = useRef(fn);
useEffect(() => { latestFn.current = fn; }, [fn]);
```

---

### Class → Hooks Mapping

| Class | Hooks |
|---|---|
| `this.state` | `useState` |
| `componentDidMount` | `useEffect(() => {}, [])` |
| `componentDidUpdate` | `useEffect(() => {}, [dep])` |
| `componentWillUnmount` | cleanup return in `useEffect` |
| instance variable | `useRef` |
| reusable lifecycle logic | custom hook |

---

### JSX Gotcha — Falsy `0` Renders

```tsx
// ❌ Renders "0" on screen when count is 0
{count && <Text>{count}</Text>}

// ✅ Explicit boolean check
{count > 0 && <Text>{count}</Text>}
{!!count && <Text>{count}</Text>}
```

---

## Vol 2 — React Native Core

### Architecture: Old vs New

| Feature | Old (Bridge) | New (JSI) |
|---|---|---|
| JS→Native comm | Async JSON serialization | Direct C++ JSI call |
| Module loading | All at startup | Lazy (TurboModules) |
| Renderer | Shadow Thread | Fabric (sync layout) |
| Performance | Bridge bottleneck | Near-native speed |

---

### Metro Bundler Reset (Common Fix)

```bash
watchman watch-del-all
rm -rf node_modules
rm -rf /tmp/metro-*
npm install
npx react-native start --reset-cache
```

---

### Hermes Benefits

```txt
✅ Faster startup (precompiled bytecode)
✅ Lower memory
✅ Better low-end Android performance
```

Check: `!!global.HermesInternal`

---

### Core Components Quick Ref

| Component | Use for |
|---|---|
| `View` | Layout container |
| `Text` | All text |
| `Pressable` | Touchable areas (prefer over TouchableOpacity) |
| `TextInput` | Controlled text input |
| `FlatList` | Large scrollable lists (virtualized) |
| `ScrollView` | Small, fully-rendered lists |

---

### Layout — Yoga Engine

```txt
Default flex direction: column
```

```tsx
// ❌ Avoid
<View style={{ height: "100%" }}>

// ✅ Use
<View style={{ flex: 1 }}>
```

---

### Screen Architecture Pattern

```txt
Screen → Container Hook → Redux / Query / API
Screen → Reusable UI Components
```

---

### Redux Data Flow

```txt
Action → Reducer → New State → UI Update
```

### Redux Saga Flow

```txt
UI dispatch → WatcherSaga → WorkerSaga → API → put(success/failure)
```

### takeLatest vs takeEvery

| Effect | Use when |
|---|---|
| `takeLatest` | Only last result matters (search, refresh) |
| `takeEvery` | All events matter (analytics, logging) |

---

### FlatList Performance Checklist

- [ ] `keyExtractor` returns stable unique ID
- [ ] Row component wrapped in `React.memo`
- [ ] `renderItem` wrapped in `useCallback`
- [ ] `getItemLayout` set if rows are fixed height
- [ ] `initialNumToRender` and `maxToRenderPerBatch` tuned
- [ ] `removeClippedSubviews` enabled
- [ ] No inline functions/objects as props to rows

---

### Pagination

| Type | Endpoint | Problem |
|---|---|---|
| Offset | `?page=1&limit=20` | Items shift when new records inserted |
| Cursor | `?cursor=abc&limit=20` | Stable, better for mobile feeds |

---

### Auth Token Refresh Flow

```txt
Request → 401 → Pause requests → Refresh token → Retry all queued requests
If refresh fails → Logout
```

---

### Environment Config Pattern

```ts
export const Config = {
  apiUrl: process.env.API_URL ?? "https://api.example.com",
};
```

Never hardcode URLs in code.

---

## Vol 3 — Performance, Testing, Debugging

### Re-render Causes

```txt
1. Local state change
2. Parent component re-rendered
3. Context value changed
4. Redux selector returned new reference
5. Inline object/function prop
```

---

### RN Threading Model

| Thread | Responsible for |
|---|---|
| JS Thread | React render, Redux, Saga, business logic, JSON |
| UI Thread | Native views, gestures, animations |
| Native Modules Thread | Camera, Bluetooth, file system etc. |

**If JS thread blocks** → button presses delayed, animations stutter, list scroll feels bad.

---

### Animation Performance

```tsx
// ❌ JS-driven (can stutter)
useNativeDriver: false

// ✅ Native-driven (smooth 60fps)
useNativeDriver: true
```

Use **Reanimated** for complex gestures + animations.

---

### Memory Leak Checklist

- [ ] `setInterval` / `setTimeout` cleared in cleanup
- [ ] API response guarded with `mounted` flag or `AbortController`
- [ ] Event listeners removed on unmount
- [ ] Redux subscriptions unsubscribed

---

### Image Performance Checklist

- [ ] Use thumbnails not original images
- [ ] Provide fixed `width` and `height`
- [ ] Use `resizeMode="cover"`
- [ ] Request WebP from backend when possible
- [ ] Avoid base64 images
- [ ] Cache images (FastImage or built-in cache)

---

### Testing Strategy

| What | Test type |
|---|---|
| Reducer | Unit (Jest) |
| Selector | Unit (Jest) |
| Custom hook | Unit (Jest) |
| Saga | Unit (redux-saga-test-plan) |
| Component | RNTL |
| Screen | RNTL integration |
| Login / critical flows | Detox E2E |

**Senior rule**: Test user-visible behavior, not implementation details.

---

### Crash Debugging Flow

```txt
Crash report → Stack trace → Device/OS/version →
Reproduce locally → Root cause → Fix → Release → Monitor
```

Tools: Firebase Crashlytics, Sentry, Bugsnag

---

### Production Logger Rule

```ts
// ❌ Never log secrets
console.log("token", token);

// ✅ Structured logging
logger.info("Payment started", { screen: "PaymentScreen" });
```

---

### Optimistic UI Pattern

```txt
1. Dispatch optimistic action (update UI immediately)
2. Call API
3. On success → confirm
4. On failure → dispatch rollback action + show toast
```

---

### Feature Flags Rule

```ts
// Always define safe defaults
const defaultFlags: FeatureFlags = {
  newHomeScreen: false,
};
```

Remote config may fail or return stale — defaults prevent crashes.

---

### Backend Mobile-Friendly Checklist

- [ ] Paginated responses (cursor-based preferred)
- [ ] Small payloads (no 5000-record dumps)
- [ ] Image resizing API (`?w=300&h=300&format=webp`)
- [ ] Compression (gzip)
- [ ] Cache headers
- [ ] Stable IDs
- [ ] Idempotency keys for payment/submit APIs

---

## Senior Interview One-Liners

| Question | Answer |
|---|---|
| What is a closure? | A function that remembers variables from its outer lexical scope |
| What is a stale closure? | A function capturing old state/props from a previous render |
| Why immutability? | React/Redux use reference comparison to detect changes |
| useMemo vs useCallback? | useMemo = cached value. useCallback = cached function reference |
| Why use keys? | Keys preserve component identity during reconciliation, not just to silence warnings |
| ScrollView vs FlatList? | ScrollView renders all children. FlatList virtualizes — only renders visible rows |
| Why Redux Saga? | Complex async: cancellation, retries, sequencing, race conditions, parallel calls |
| What blocks the JS thread? | Heavy synchronous work — parsing large JSON, complex loops, layout calculations |
| Optimistic UI tradeoff? | Better perceived performance, but you need rollback logic for failure cases |
| JSI vs Bridge? | Bridge serializes JSON async. JSI calls native C++ directly — faster, synchronous capable |

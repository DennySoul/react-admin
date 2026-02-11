---
name: debugging
description: Structured debugging methodology for React applications and TypeScript code with systematic 5-step process (Reproduce, Isolate, Analyze, Hypothesize, Test). Use when diagnosing bugs, troubleshooting errors, investigating React re-render issues (too many re-renders), analyzing async/await problems, debugging useEffect dependencies, resolving "Cannot read property of undefined" errors, fixing stale closures, or any React/TypeScript runtime errors.
---

# Debugging Methodology

## Structured Debugging Process

Follow this systematic approach:

### 1. Reproduce
- Confirm the exact steps to reproduce
- Note the environment (browser, Node version, OS)
- Identify if it's consistent or intermittent

### 2. Isolate
- Find the smallest code path that triggers the issue
- Comment out unrelated code to narrow scope
- Check if issue exists in isolation (new component/file)

### 3. Analyze
- Read the full error message and stack trace
- Identify the exact line where error originates
- Trace data flow backward from the error

### 4. Hypothesize
- Form a theory about the root cause
- List possible causes in order of likelihood
- Consider recent changes that might be related

### 5. Test & Fix
- Test hypothesis with minimal change
- Verify fix doesn't break other functionality
- Add test to prevent regression

## React-Specific Debugging

### Common React Errors

#### "Cannot read property 'X' of undefined"
```tsx
// Cause: Accessing property before data loads
const name = user.name; // user is undefined

// Fix: Optional chaining + fallback
const name = user?.name ?? 'Unknown';

// Or: Early return / loading state
if (!user) return <Loading />;
```

#### "Too many re-renders"
```tsx
// Cause: State update in render
const [count, setCount] = useState(0);
setCount(count + 1); // ❌ Called during render

// Fix: Move to useEffect or event handler
useEffect(() => {
  setCount(c => c + 1);
}, [someDependency]);
```

#### "Can't perform state update on unmounted component"
```tsx
// Cause: Async operation completes after unmount
useEffect(() => {
  fetchData().then(setData);
}, []);

// Fix: Cleanup with abort or flag
useEffect(() => {
  let cancelled = false;
  fetchData().then(data => {
    if (!cancelled) setData(data);
  });
  return () => { cancelled = true; };
}, []);
```

#### "Objects are not valid as a React child"
```tsx
// Cause: Rendering object directly
return <div>{user}</div>; // user is an object

// Fix: Access specific property or stringify
return <div>{user.name}</div>;
return <div>{JSON.stringify(user)}</div>;
```

### useEffect Dependency Issues

```tsx
// Symptom: Effect runs infinitely
useEffect(() => {
  setItems(data.filter(d => d.active));
}, [data]); // ❌ If data is new array each render

// Diagnosis: Check if dependency is stable
console.log('data reference:', data);

// Fix: Memoize upstream or use functional update
const activeItems = useMemo(
  () => data.filter(d => d.active),
  [data]
);
```

## Debugging Tools

### Console Methods
```tsx
console.log('value:', value);           // Basic
console.table(arrayOfObjects);          // Tabular data
console.trace('how did we get here');   // Stack trace
console.time('operation');              // Performance
// ... operation
console.timeEnd('operation');
console.group('Section');               // Grouped logs
console.log('nested');
console.groupEnd();
```

### React DevTools
- Components tab: Inspect props and state
- Profiler tab: Find performance bottlenecks
- "Highlight updates" to see re-renders

### Network Tab
- Check request/response payloads
- Look for failed requests (red)
- Verify correct headers sent

## Debugging Prompts Template

When asking for debugging help, include:

```
**Error:** [exact error message]

**Stack trace:**
\`\`\`
[paste full stack trace]
\`\`\`

**Reproduction steps:**
1. [step 1]
2. [step 2]

**Expected:** [what should happen]

**Actual:** [what happens instead]

**Relevant code:**
\`\`\`tsx
[paste relevant code]
\`\`\`

**What I've tried:**
- [attempt 1]
- [attempt 2]
```

## Common Pitfalls

### Async/Await
```tsx
// ❌ Forgetting await
const data = fetchData(); // Returns Promise, not data

// ❌ Await in loop (sequential, slow)
for (const id of ids) {
  await fetchItem(id);
}

// ✅ Parallel execution
await Promise.all(ids.map(fetchItem));
```

### Stale Closures
```tsx
// ❌ Stale value in callback
const [count, setCount] = useState(0);
const handleClick = () => {
  setTimeout(() => {
    console.log(count); // Always logs initial value
  }, 1000);
};

// ✅ Use ref for latest value
const countRef = useRef(count);
countRef.current = count;
const handleClick = () => {
  setTimeout(() => {
    console.log(countRef.current);
  }, 1000);
};
```

### TypeScript Narrowing
```tsx
// ❌ Type guard doesn't persist
if (user !== null) {
  setTimeout(() => {
    console.log(user.name); // TS error: user might be null
  }, 0);
}

// ✅ Capture narrowed value
if (user !== null) {
  const validUser = user;
  setTimeout(() => {
    console.log(validUser.name); // OK
  }, 0);
}
```

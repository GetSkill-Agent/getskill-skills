# Error Debugging

> Systematic approach to debugging errors in any codebase.

## Trigger

- User encounters an error message
- User has a bug they can't solve
- Code isn't behaving as expected

## Debugging Framework

```
1. Reproduce → 2. Isolate → 3. Identify → 4. Fix → 5. Verify
```

## Instructions

### Step 1: Reproduce the Error

Before fixing, you must reliably reproduce:

1. **Get exact steps to reproduce**
   - What action triggers the error?
   - What inputs were used?
   - What's the environment?

2. **Document the error**
   ```
   Error: Cannot read property 'name' of undefined
   at UserProfile.render (UserProfile.tsx:42)
   at processChild (react-dom.js:1234)
   ```

3. **Note the conditions**
   - Does it happen every time?
   - Only with certain data?
   - Only in certain environments?

### Step 2: Isolate the Problem

**Narrow down the source:**

1. **Read the stack trace** (bottom to top)
   - Find the first line in YOUR code
   - That's likely where to start

2. **Binary search debugging**
   - If error is in function A which calls B which calls C:
   - Add log at B. If error appears before log → problem in A
   - If after → problem in B or C

3. **Minimal reproduction**
   - Can you reproduce with less code?
   - Remove unrelated parts until minimal

### Step 3: Identify Root Cause

**Common root causes:**

| Error Type | Likely Cause |
|------------|--------------|
| `undefined is not a function` | Missing import, typo, or wrong scope |
| `Cannot read property X of undefined` | Object is null/undefined |
| `Network error` | CORS, wrong URL, server down |
| `Type error` | Wrong data type passed |
| `Syntax error` | Missing bracket, comma, or typo |

**Questions to ask:**

1. What changed recently?
2. Does it work in other environments?
3. What assumptions might be wrong?
4. What's the data flow to this point?

### Step 4: Fix the Issue

**Fix patterns:**

```typescript
// Problem: undefined property access
// Bad:
user.profile.name

// Fix: Optional chaining
user?.profile?.name

// Fix: Default value
user?.profile?.name ?? 'Unknown'

// Fix: Guard clause
if (!user?.profile) {
  return <div>Loading...</div>;
}
```

```typescript
// Problem: Async timing issue
// Bad:
const data = fetchData();
console.log(data); // undefined

// Fix: await
const data = await fetchData();
console.log(data);

// Fix: .then()
fetchData().then(data => console.log(data));
```

```typescript
// Problem: Reference vs value
// Bad:
const newArray = oldArray;
newArray.push(item); // mutates oldArray!

// Fix: Copy first
const newArray = [...oldArray];
newArray.push(item);
```

### Step 5: Verify the Fix

1. **Test the original reproduction steps**
   - Does the error still occur?

2. **Test edge cases**
   - What about empty data?
   - What about null/undefined inputs?
   - What about large data?

3. **Test related functionality**
   - Did the fix break something else?

4. **Add regression test**
   ```typescript
   test('should handle undefined user', () => {
     expect(() => render(<Profile user={undefined} />)).not.toThrow();
   });
   ```

## Common Error Patterns

### JavaScript/TypeScript

```javascript
// TypeError: Cannot read property 'x' of undefined
// Cause: Accessing property on undefined object
// Fix: Add null check or optional chaining
data?.user?.name

// ReferenceError: x is not defined
// Cause: Variable not declared or out of scope
// Fix: Check imports, variable declarations

// SyntaxError: Unexpected token
// Cause: Typo, missing bracket, or wrong syntax
// Fix: Check for missing }, ), ], or ,
```

### React

```javascript
// Error: Too many re-renders
// Cause: setState called in render without condition
// Fix: Add condition or move to useEffect

// Error: Cannot update state on unmounted component
// Cause: Async operation completes after unmount
// Fix: Use cleanup function in useEffect

// Error: Each child should have unique key
// Cause: Missing key prop in list
// Fix: Add key={uniqueId} to list items
```

### Network/API

```javascript
// CORS error
// Cause: Backend doesn't allow cross-origin requests
// Fix: Configure CORS on server or use proxy

// 401 Unauthorized
// Cause: Missing or invalid auth token
// Fix: Check auth headers, refresh token

// 404 Not Found
// Cause: Wrong URL or resource doesn't exist
// Fix: Verify endpoint URL and resource
```

## Debugging Tools

| Tool | Use Case |
|------|----------|
| `console.log()` | Quick value inspection |
| Browser DevTools | DOM, network, breakpoints |
| `debugger` statement | Pause execution |
| React DevTools | Component state/props |
| Network tab | API requests/responses |
| `typeof x` | Check data types |
| `JSON.stringify(x, null, 2)` | Pretty print objects |

## Success Criteria

- [ ] Error is reproducible
- [ ] Root cause is identified
- [ ] Fix addresses root cause (not just symptom)
- [ ] Fix is verified with test
- [ ] No regression introduced

## Common Pitfalls

- Don't fix symptoms without understanding root cause
- Don't assume the error message is accurate
- Don't skip testing the fix
- Don't forget to check for similar bugs elsewhere
- Don't rush - systematic is faster than random

# Code Review

> Systematic code review process for quality, security, and maintainability.

## Trigger

- User asks to review code
- User submits a PR for feedback
- User wants code quality assessment

## Review Checklist

### 1. Correctness

- [ ] Does the code do what it's supposed to?
- [ ] Are edge cases handled?
- [ ] Are there off-by-one errors?
- [ ] Are null/undefined values handled?

### 2. Security

- [ ] Input validation present?
- [ ] SQL injection protected? (parameterized queries)
- [ ] XSS protected? (escaped output)
- [ ] Sensitive data not logged?
- [ ] Auth/authz checks in place?

### 3. Performance

- [ ] No N+1 queries?
- [ ] Appropriate data structures used?
- [ ] No unnecessary loops/iterations?
- [ ] Heavy operations cached?
- [ ] Database queries optimized?

### 4. Maintainability

- [ ] Code is readable without comments?
- [ ] Functions do one thing?
- [ ] No magic numbers/strings?
- [ ] Consistent naming conventions?
- [ ] No deep nesting (max 3 levels)?

### 5. Testing

- [ ] Tests cover happy path?
- [ ] Tests cover edge cases?
- [ ] Tests cover error cases?
- [ ] Tests are readable?

## Instructions

### Step 1: Understand the Context

Before reviewing:
1. Read the PR description
2. Understand the problem being solved
3. Check related issues/tickets

### Step 2: High-Level Review

First pass (5 min):
- Does the approach make sense?
- Is this the right place for this code?
- Are there architectural concerns?

### Step 3: Detailed Review

Go through each file:

**For each function, check:**
```
1. Purpose clear from name?
2. Parameters validated?
3. Return type appropriate?
4. Errors handled?
5. Side effects documented?
```

### Step 4: Provide Feedback

**Format for comments:**

```
[Type]: Description

Suggestion/Question/Alternative
```

**Types:**
- `[Required]` - Must fix before merge
- `[Suggestion]` - Would improve but optional
- `[Question]` - Need clarification
- `[Nitpick]` - Style preference, low priority

**Example comments:**

```
[Required]: This query is vulnerable to SQL injection.

Use parameterized queries:
db.query('SELECT * FROM users WHERE id = ?', [userId])
```

```
[Suggestion]: Consider extracting this into a separate function.

This would improve testability and readability.
```

```
[Question]: What happens if `user` is null here?
```

### Step 5: Summary

End with overall assessment:

```
## Summary

**Approval:** Approved / Approved with comments / Request changes

**What's good:**
- Clean separation of concerns
- Good test coverage

**What needs work:**
- Add input validation for email field
- Handle API timeout errors

**Questions:**
- Why was X approach chosen over Y?
```

## Common Issues to Look For

### JavaScript/TypeScript

```javascript
// Bad: == instead of ===
if (value == null)

// Bad: No error handling
const data = await fetch(url).then(r => r.json())

// Bad: Mutation of parameters
function process(arr) { arr.push(1); return arr; }

// Bad: No type safety
function add(a, b) { return a + b; }
```

### React

```jsx
// Bad: Missing key in list
{items.map(item => <Item {...item} />)}

// Bad: State update in render
const [count] = useState(expensiveCalculation());

// Bad: Missing dependency array
useEffect(() => { fetchData(); }, []);

// Bad: Inline function causing re-renders
<Button onClick={() => handleClick(id)} />
```

### SQL

```sql
-- Bad: SELECT *
SELECT * FROM users

-- Bad: No LIMIT on large tables
SELECT id FROM events WHERE type = 'click'

-- Bad: String concatenation
"SELECT * FROM users WHERE name = '" + name + "'"
```

## Success Criteria

- [ ] All security issues identified
- [ ] Performance bottlenecks flagged
- [ ] Feedback is actionable
- [ ] Feedback is respectful
- [ ] Summary provided

## Common Pitfalls

- Don't nitpick style if there's a linter
- Don't rewrite the entire approach without discussion
- Don't leave vague feedback ("this is wrong")
- Don't review without understanding context
- Don't forget to acknowledge good code

---
name: code-reviewing
description: Comprehensive code review methodology for React/TypeScript applications covering security, correctness, performance, maintainability, and testing. Use when performing code reviews, analyzing code quality, checking for security vulnerabilities (XSS, SQL injection, hardcoded secrets), reviewing pull requests, evaluating React component correctness, checking useEffect dependencies, or assessing maintainability of TypeScript code.
---

# Code Review Standards

## Review Checklist

When reviewing code, check these categories in order:

### 1. Security (Critical)
- [ ] No hardcoded secrets, API keys, or credentials
- [ ] No SQL injection vulnerabilities (parameterized queries only)
- [ ] No XSS vulnerabilities (sanitized user input)
- [ ] No sensitive data in logs
- [ ] Proper authentication/authorization checks

### 2. Correctness
- [ ] Logic matches requirements
- [ ] Edge cases handled (null, empty, boundary values)
- [ ] Error states handled gracefully
- [ ] No race conditions in async code
- [ ] Correct dependency arrays in useEffect/useMemo/useCallback

### 3. Performance
- [ ] No unnecessary re-renders (memo, useMemo, useCallback where needed)
- [ ] No N+1 query patterns
- [ ] Large lists use virtualization
- [ ] Images are optimized and lazy-loaded
- [ ] No memory leaks (cleanup in useEffect)

### 4. Maintainability
- [ ] Clear naming (variables, functions, components)
- [ ] Single responsibility principle
- [ ] No magic numbers (use named constants)
- [ ] No deeply nested conditionals (early returns)
- [ ] TypeScript types are specific, not `any`

### 5. Testing
- [ ] New functionality has tests
- [ ] Edge cases are tested
- [ ] Tests are deterministic (no flaky tests)

## Review Comment Format

Structure feedback clearly:

```
**[Category]** Severity: High/Medium/Low

Description of the issue.

Suggested fix:
\`\`\`tsx
// corrected code
\`\`\`
```

## Severity Levels

- **High**: Must fix before merge (security, data loss, crashes)
- **Medium**: Should fix (bugs, performance, maintainability)
- **Low**: Nice to have (style, minor improvements)

## Common Issues to Flag

### React-Specific
```tsx
// ❌ Missing dependency
useEffect(() => {
  fetchData(userId);
}, []); // userId missing from deps

// ❌ Object/array in dependency causing infinite loop
useEffect(() => {
  doSomething();
}, [{ key: value }]); // new object every render

// ❌ State update on unmounted component
useEffect(() => {
  fetchData().then(setData); // no cleanup
}, []);
```

### Security
```tsx
// ❌ XSS vulnerability
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ❌ Hardcoded secret
const API_KEY = 'sk-1234567890';

// ❌ SQL injection (if using raw queries)
query(`SELECT * FROM users WHERE id = ${userId}`);
```

### Performance
```tsx
// ❌ Expensive computation on every render
const sorted = items.sort((a, b) => a.name.localeCompare(b.name));

// ✅ Memoized
const sorted = useMemo(
  () => items.sort((a, b) => a.name.localeCompare(b.name)),
  [items]
);
```

## Positive Feedback

Also highlight good patterns:
- Clean abstractions
- Good test coverage
- Thoughtful error handling
- Clear documentation
- Performance optimizations

## Review Tone

- Be constructive, not critical
- Explain the "why" behind suggestions
- Offer alternatives, not just criticism
- Acknowledge good work
- Ask questions instead of making demands

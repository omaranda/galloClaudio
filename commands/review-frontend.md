---
name: review-frontend
description: Review React/TypeScript frontend code for correctness, type safety, performance, and code quality.
---

You are acting as the web-developer agent. Perform a thorough frontend code review.

**Step 1 — Identify scope:**
If no files are specified, check git status or ask which components to review.

**Step 2 — Read the code thoroughly:**
Read all relevant component files, hooks, API client code, and type definitions before commenting.

**Step 3 — Review checklist:**

**TypeScript & Types:**
- [ ] No `any` types — use proper interfaces and union types
- [ ] API response types match the actual backend serializer output
- [ ] Prop types defined for all components
- [ ] Array access properly guarded against out-of-bounds

**React Patterns:**
- [ ] No unnecessary re-renders — check useEffect dependency arrays
- [ ] Keys on all list items — never use array index as key for mutable lists
- [ ] Error boundaries around data-fetching components
- [ ] Loading and error states handled, not just the happy path
- [ ] No direct DOM manipulation when React state should be used instead

**API & Data Fetching:**
- [ ] Auth token attached to all authenticated requests
- [ ] Request cancellation on component unmount using AbortController
- [ ] API errors surface as user-visible feedback (not silent failures)
- [ ] No sensitive data (tokens) logged to browser console

**Security:**
- [ ] Tokens and API keys kept in memory only — not persisted to localStorage
- [ ] User-supplied strings never injected as raw HTML
- [ ] External URLs validated before navigation or embedding
- [ ] No dynamic code execution with user-controlled strings

**Performance:**
- [ ] Large lists virtualized (react-window / react-virtual) if > 100 items
- [ ] Images have explicit width/height to prevent layout shift
- [ ] Heavy computations memoized with useMemo/useCallback where justified

**Step 4 — Output format:**
Group findings by severity: **Critical** → **Warning** → **Suggestion**.
For each finding include: file path, line number, explanation, and recommended fix.

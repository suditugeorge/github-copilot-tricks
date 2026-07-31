---
name: react19-test-patterns
description: >
  React 19 test authoring patterns for the noviapet-web frontend (Vitest + jsdom + React Testing
  Library, React Compiler disabled during tests). Use this skill when writing or fixing tests for
  React 19 components — covers act() imports, fireEvent vs Simulate, StrictMode call-count changes,
  async action testing (useActionState/useOptimistic), Suspense/use() testing, and the
  renderWithI18n helper. Trigger for prompts like "test", "vitest", "testing library", "act",
  "StrictMode", "test fails", "coverage", or when writing *.test.tsx files in noviapet-web.
---

# React 19 Test Patterns — noviapet-web

noviapet-web tests with **Vitest + jsdom + React Testing Library**. Globals are enabled; setup is
in `src/test/setup.ts`. The **React Compiler is disabled during tests** (see
`noviapet-web/docs/REACT_COMPILER.md`) so V8 coverage maps cleanly to source lines.

## Priority order for test issues

Fix in this order — each layer depends on the previous:

1. **`act` import** — fix first, unblocks everything
2. **`Simulate` → `fireEvent`** — fix immediately after act
3. **`react-dom/test-utils` cleanup** — remove remaining imports
4. **StrictMode call counts** — measure actual, don't guess
5. **Async action wrapping** — for "not wrapped in act" warnings
6. **Custom render helper** — verify once per codebase (noviapet-web uses `renderWithI18n`)

---

## 1. `act()` import

```tsx
// ❌ REMOVED in React 19:
import { act } from 'react-dom/test-utils';

// ✅ React 19:
import { act } from 'react';
```

If mixed with other test-utils imports, split them:

```tsx
// ❌ Before:
import { act, Simulate, renderIntoDocument } from 'react-dom/test-utils';

// ✅ After:
import { act } from 'react';
import { fireEvent, render } from '@testing-library/react';
```

> noviapet-web uses `@testing-library/user-event` for interactions, which wraps events in `act`
> automatically. You rarely need to import `act` directly — prefer `userEvent` (see section 5).

---

## 2. `Simulate` → `fireEvent` / `userEvent`

```tsx
// ❌ Simulate REMOVED in React 19:
import { Simulate } from 'react-dom/test-utils';
Simulate.click(element);
Simulate.change(input, { target: { value: 'hello' } });
Simulate.submit(form);

// ✅ React 19 — prefer userEvent (noviapet-web convention):
import userEvent from '@testing-library/user-event';
await user.click(element);
await user.type(input, 'hello');
await user.click(submitButton); // or user.upload for files

// ✅ Or fireEvent for lower-level control:
import { fireEvent } from '@testing-library/react';
fireEvent.click(element);
fireEvent.change(input, { target: { value: 'hello' } });
fireEvent.submit(form);
```

> **noviapet-web convention:** use `@testing-library/user-event` for interactions (per
> `.copilot-instructions.md`). Reserve `fireEvent` for cases where `userEvent` does not support
> the event type.

---

## 3. `react-dom/test-utils` full API map

| Old (`react-dom/test-utils`) | New location |
|---|---|
| `act` | `import { act } from 'react'` |
| `Simulate` | `fireEvent` from `@testing-library/react` or `userEvent` |
| `renderIntoDocument` | `render` from `@testing-library/react` |
| `findRenderedDOMComponentWithTag` | `getByRole`, `getByTestId` from RTL |
| `findRenderedDOMComponentWithClass` | `getByRole` or `container.querySelector` |
| `scryRenderedDOMComponentsWithTag` | `getAllByRole` from RTL |
| `isElement`, `isCompositeComponent` | remove — not needed with RTL |
| `isDOMComponent` | remove |

---

## 4. StrictMode call-count fixes

React 19 StrictMode no longer double-invokes `useEffect` in development. Spy assertions counting
effect calls must be updated.

**Strategy — always measure, never guess:**

```bash
# Run the failing test, read the actual count from the error:
npm run test -- --run src/path/to/file.test.tsx 2>&1 | grep -E "Expected|Received"
```

```tsx
// ❌ React 18 StrictMode (effects ran twice):
expect(mockFn).toHaveBeenCalledTimes(2); // 1 call × 2 (strict double-invoke)

// ✅ React 19 StrictMode (effects run once):
expect(mockFn).toHaveBeenCalledTimes(1);
```

```tsx
// Render-phase calls (component body) — still double-invoked in React 19 StrictMode:
expect(renderSpy).toHaveBeenCalledTimes(2); // stays at 2 for render body calls
```

> noviapet-web's `src/main.tsx` wraps the app in `<React.StrictMode>`. Tests render without
> StrictMode by default (via `renderWithI18n`), so effect double-invoke usually does not apply
> unless a test explicitly wraps in StrictMode.

---

## 5. Async action testing (`useActionState` / `useOptimistic`)

`useActionState` and `useOptimistic` are async — wrap interactions in `await` and let
`userEvent` handle the `act` wrapping:

```tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { renderWithI18n } from '@/test/helpers';

vi.mock('@/lib/api-client', () => ({
  apiFetch: vi.fn(),
}));

describe('ChatInput', () => {
  it('sends a message and shows it optimistically', async () => {
    const user = userEvent.setup();
    const { apiFetch } = await import('@/lib/api-client');
    vi.mocked(apiFetch).mockResolvedValue({ id: '1', role: 'assistant', text: 'Got it' });

    renderWithI18n(<ChatInput />);

    await user.type(screen.getByRole('textbox'), 'My pet is lethargic');
    await user.click(screen.getByRole('button', { name: /send/i }));

    // Optimistic message appears immediately:
    expect(screen.getByText('My pet is lethargic')).toBeInTheDocument();
    // Await the async action to settle:
    expect(await screen.findByText('Got it')).toBeInTheDocument();
  });
});
```

**Key points:**

- `userEvent.setup()` returns a user whose actions are auto-wrapped in `act`
- Use `findBy*` (async) to wait for post-action UI updates
- Mock `apiFetch` with `vi.mock` — never hit real endpoints (per `.copilot-instructions.md`)
- The React Compiler is disabled in tests, so `useActionState`'s action function is recreated
  each render — this is fine, `useActionState` tracks it internally

---

## 6. Suspense / `use()` testing

When testing components that use `use()` with a promise, control the promise resolution:

```tsx
import { Suspense } from 'react';

it('shows fallback while loading, then content', async () => {
  let resolvePromise!: (value: AiTriage) => void;
  const triagePromise = new Promise<AiTriage>((resolve) => {
    resolvePromise = resolve;
  });

  vi.mocked(apiFetch).mockReturnValue(triagePromise);

  renderWithI18n(
    <Suspense fallback={<div>Loading triage…</div>}>
      <TriageSummary appointmentId="123" />
    </Suspense>,
  );

  // Fallback shown while pending:
  expect(screen.getByText('Loading triage…')).toBeInTheDocument();

  // Resolve the promise:
  resolvePromise({ urgency_level: 'urgent', /* ... */ });

  // Content appears after resolution:
  expect(await screen.findByText('urgent')).toBeInTheDocument();
});
```

> **Memoize promises in tests.** The React Compiler is disabled in tests, so a bare
> `use(apiFetch(...))` recreates the promise every render → infinite suspend loop. Either memoize
> with `useMemo` in the component, or inject a stable promise via props/mocks (as above).

---

## 7. Custom render helper — `renderWithI18n`

noviapet-web's custom render helper is `renderWithI18n` in `src/test/helpers.tsx`. It wraps
components with the i18next provider so `useTranslation()` works. Use it for any component that
calls `useTranslation`:

```tsx
import { renderWithI18n } from '@/test/helpers';

it('renders translated heading', () => {
  renderWithI18n(<IntakeChatPage />);
  expect(screen.getByRole('heading', { name: /intake/i })).toBeInTheDocument();
});
```

Tests typically reset to `ro` with `await i18n.changeLanguage('ro')` in `beforeEach`. Verify this
helper once per codebase — do not recreate it per test.

---

## Coverage target

noviapet-web targets **100% code coverage** (per `.copilot-instructions.md`). Verify with:

```bash
npm run test:coverage
```

HTML report at `coverage/index.html`. UI primitives under `src/components/ui/` are excluded from
coverage. Feature code under `src/features/` and `src/components/` (non-ui) is included.

---

## Common pitfalls

- **"not wrapped in act"** — you forgot to `await` an async interaction. Use `await user.click()`
  or `await screen.findBy*` instead of `screen.getBy*` for post-async assertions.
- **Infinite suspend loop** — a `use()` component re-creates its promise every render in tests
  (compiler disabled). Memoize the promise with `useMemo` or inject a stable promise.
- **Effect count mismatch** — React 19 StrictMode runs effects once, not twice. Measure the actual
  count from the test error output.
- **`useFormStatus` not triggering** — it only works inside `<form action={...}>`, not
  `<form onSubmit>`. Test the form with `userEvent.setup()` and `await user.click(submitButton)`.
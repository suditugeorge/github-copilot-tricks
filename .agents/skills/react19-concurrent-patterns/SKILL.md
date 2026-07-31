---
name: react19-concurrent-patterns
description: >
  React 19 concurrent and async patterns for the noviapet-web frontend (React 19.2 + React Compiler).
  Use this skill when writing or refactoring components that handle async data, form submissions,
  streaming LLM responses, optimistic UI, or Suspense boundaries — e.g. the AI pet-intake chat
  (R-04), ai-triage polling, or any feature that calls apiFetch asynchronously. Covers use(),
  useActionState, useFormStatus, useOptimistic, useTransition, useDeferredValue, and Suspense.
  Trigger for prompts like "chat input", "optimistic message", "streaming response", "form action",
  "suspense", "pending state", or any async UI work in noviapet-web.
---

# React 19 Concurrent Patterns — noviapet-web

noviapet-web runs **React 19.2** with the **React Compiler** enabled in production builds (see
`noviapet-web/docs/REACT_COMPILER.md`). This skill teaches the correct React 19 async/concurrent
patterns to use when building features — most notably the **AI Pet Intake Chat** (R-04) and the
**AI Triage review panel**, but applicable to any async UI.

## Stack constraints that shape these patterns

- **React Compiler auto-memoizes.** Do NOT add manual `useMemo`/`useCallback`/`React.memo()` unless
  a component calls `useReactTable()` (the compiler skips those — wrap them in `memo()` manually,
  as `ClientListDesktopTable` and `TeamMembersDesktopTable` already do).
- **All network calls go through `apiFetch`** from `@/lib/api-client` (CSRF, cookie auth,
  RFC 9457 error parsing, `Accept-Language` header). Never use raw `fetch` in feature code.
- **Client SPA only.** There are no Server Components / Server Actions. "Actions" here means the
  client-side `useActionState` pattern, not RSC server mutations.
- **i18next is synchronous** (`useSuspense: false`). Do not suspend the app on i18n.
- **Tests disable the React Compiler** so V8 coverage maps cleanly. Test the uncompiled source.

## Part 1 — Patterns that already work (do not "migrate" them)

noviapet-web is already on React 19, so these are **authoring standards**, not migrations:

### createRoot — already correct

`src/main.tsx` uses `createRoot` from `react-dom/client`. Keep this pattern for any new entry points.

### useTransition — unchanged, use for expensive non-urgent updates

```tsx
const [isPending, startTransition] = useTransition();

function handleSearch(query: string) {
  startTransition(() => {
    setFilteredResults(computeExpensiveFilter(query));
  });
}
```

### useDeferredValue — unchanged, use to defer heavy rendering

```tsx
const deferredQuery = useDeferredValue(query);
// render based on deferredQuery so typing stays responsive
```

### Suspense for code splitting — unchanged

```tsx
const LazyPage = lazy(() => import('@/pages/some-page'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <LazyPage />
    </Suspense>
  );
}
```

---

## Part 2 — React 19 new APIs to adopt

These are the idiomatic React 19 patterns for the AI intake chat and similar async features.
For full before/after code on each, read the bundled references:

- **`references/react19-use.md`** — the `use()` hook for promises and context
- **`references/react19-actions.md`** — `useActionState`, `useFormStatus`, `useOptimistic`
- **`references/react19-suspense.md`** — Suspense for data fetching

### When to use which API (R-04 mapping)

| R-04 need | React 19 API | Why |
|---|---|---|
| Chat input free-text + photo upload | `useActionState` + `<form action>` | FormData collected automatically; `isPending` drives the send button |
| Send button pending state | `useFormStatus` | No prop-drilling from form to button |
| Message appears instantly on send | `useOptimistic` | Show user message before backend confirms |
| LLM clarifier "thinking…" state | `useActionState` `isPending` | Async LLM call via `apiFetch` |
| ai_triage job still running → spinner | `use()` + `<Suspense>` or polling | Suspend while polling the triage job status |
| Streaming clarifying-question text | `use()` on a streaming promise | Token-by-token render under a Suspense boundary |

### useActionState — for the chat input and any async form

```tsx
import { useActionState } from 'react';
import { apiFetch } from '@/lib/api-client';

async function sendChatMessageAction(
  prevState: { error: string | null; data: ChatResponse | null },
  formData: FormData,
): Promise<typeof prevState> {
  const text = String(formData.get('message') ?? '');
  try {
    const data = await apiFetch<ChatResponse>('/api/intake/messages', {
      method: 'POST',
      body: JSON.stringify({ text }),
    });
    return { error: null, data };
  } catch (err) {
    return { error: err instanceof Error ? err.message : 'Unknown error', data: null };
  }
}

function ChatInput() {
  const [state, formAction, isPending] = useActionState(sendChatMessageAction, {
    error: null,
    data: null,
  });

  return (
    <form action={formAction}>
      <input name="message" />
      {isPending && <Loading />}
      {state.error && <p role="alert">{state.error}</p>}
      <SendButton />
    </form>
  );
}
```

### useFormStatus — child components read pending state from the parent form

```tsx
import { useFormStatus } from 'react';

function SendButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? t('common:sending') : t('common:send')}
    </button>
  );
}
```

> `useFormStatus` only works inside a `<form action={...}>`. A plain `<form onSubmit>` will NOT
> trigger it.

### useOptimistic — instant message rendering in the chat list

```tsx
import { useOptimistic } from 'react';

function ChatMessageList({ messages }: { messages: ChatMessage[] }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage: ChatMessage) => [...state, newMessage],
  );

  async function handleSend(formData: FormData) {
    const text = String(formData.get('message') ?? '');
    addOptimisticMessage({ id: crypto.randomUUID(), role: 'user', text });
    // then call the action that persists via apiFetch
  }

  return (
    <>
      <ul>
        {optimisticMessages.map((m) => (
          <li key={m.id}>{m.text}</li>
        ))}
      </ul>
      <form action={handleSend}>
        <input name="message" />
        <SendButton />
      </form>
    </>
  );
}
```

### use() — for streaming LLM responses and triage polling

```tsx
import { use, Suspense } from 'react';

function ClarifierResponse({ promise }: { promise: Promise<ClarifierResult> }) {
  const result = use(promise); // suspends until the LLM clarifier resolves
  return <p>{result.followUpQuestion ?? result.value}</p>;
}

function IntakeChatPage() {
  return (
    <Suspense fallback={<Loading label={t('intake:thinking')} />}>
      <ClarifierResponse promise={clarifierPromise} />
    </Suspense>
  );
}
```

**Caveat:** `use()` does not cache. For the ai_triage polling job, prefer a small polling hook
(`setInterval` + `apiFetch`) wrapped in `useTransition` rather than raw `use()` on a fetch promise,
unless you memoize the promise. See `references/react19-suspense.md` for the memoized-promise pattern.

---

## React Compiler interaction

- `useActionState`, `useOptimistic`, `useFormStatus`, `use()`, `useTransition`, `useDeferredValue`
  are all **compiler-safe**. Write them naturally — no manual memoization needed.
- If a component also calls `useReactTable()`, the compiler skips it. Wrap that component in
  `React.memo()` manually (existing pattern in `client-list-table.tsx`).
- Do not wrap action functions in `useCallback` — the compiler handles reference stability.

## i18n reminder

All user-facing strings (`t('intake:thinking')`, button labels, error messages shown to the pet
parent) must be added to **both** `src/i18n/locales/en/intake.json` and `src/i18n/locales/ro/intake.json`.
Backend-supplied error strings from `ApiError` are rendered verbatim, not through `t()`.

## Testing reminder

Tests run with the React Compiler **disabled**. `useActionState`/`useOptimistic`/`use()` behave the
same at runtime; just ensure async actions are awaited inside `await act(...)` or with
`@testing-library/user-event`'s async helpers. See the `react19-test-patterns` skill for details.
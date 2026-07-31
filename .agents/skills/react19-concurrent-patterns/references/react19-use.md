---
title: React 19 use() Hook Pattern Reference
---

# React 19 `use()` Hook — noviapet-web Reference

`use()` is React 19's hook for unwrapping promises and context inside components. It enables
cleaner async patterns directly in the component body, avoiding the `useEffect + state` dance.

## What `use()` does

- Accepts a **promise** or a **context** object
- Returns the resolved value or context value
- Handles **Suspense automatically** for promises
- Can be called **conditionally** inside components (unlike other hooks)
- Throws errors, which an error boundary can catch

## `use()` with promises

### The old `useEffect + state` pattern (avoid for simple cases)

```tsx
function PatientHeader({ patientId }: { patientId: string }) {
  const [patient, setPatient] = useState<Patient | null>(null);

  useEffect(() => {
    let cancelled = false;
    apiFetch<Patient>(`/api/patients/${patientId}`).then((p) => {
      if (!cancelled) setPatient(p);
    });
    return () => { cancelled = true; };
  }, [patientId]);

  if (!patient) return <Loading />;
  return <h1>{patient.name}</h1>;
}
```

### The React 19 `use()` pattern

```tsx
import { use, Suspense } from 'react';
import { apiFetch } from '@/lib/api-client';

function PatientHeader({ patientId }: { patientId: string }) {
  const patient = use(apiFetch<Patient>(`/api/patients/${patientId}`));
  return <h1>{patient.name}</h1>;
}

// Wrap in a Suspense boundary at an appropriate level:
function AppointmentPage({ patientId }: { patientId: string }) {
  return (
    <Suspense fallback={<Loading />}>
      <PatientHeader patientId={patientId} />
    </Suspense>
  );
}
```

**Key differences:**

- `use()` unwraps the promise directly in the component body
- A Suspense boundary is still required, but can be placed higher in the tree
- No `useState`/`useEffect` needed for simple async data
- Conditional wrapping is allowed inside components

## `use()` with conditional fetching

```tsx
function IntakeClarifier({ answer }: { answer: string | null }) {
  if (!answer) return null;
  // Only fetches when answer is truthy — allowed because use() can be conditional
  const result = use(clarifyAnswer(answer));
  return <p>{result.followUpQuestion}</p>;
}
```

## `use()` with context

`use()` can read context conditionally, unlike `useContext` which must be top-level:

```tsx
function ThemeAwareButton({ useSystemTheme }: { useSystemTheme: boolean }) {
  const theme = useSystemTheme ? use(ThemeContext) : defaultTheme;
  return <button className={theme}>…</button>;
}
```

## Memoizing the promise (important)

A bare `use(apiFetch(...))` recreates the promise on every render. Memoize it:

```tsx
function PatientHeader({ patientId }: { patientId: string }) {
  const patientPromise = useMemo(
    () => apiFetch<Patient>(`/api/patients/${patientId}`),
    [patientId],
  );
  const patient = use(patientPromise);
  return <h1>{patient.name}</h1>;
}
```

> **React Compiler note:** the compiler auto-memoizes the promise reference, so manual `useMemo`
> is optional in production builds. In **tests** the compiler is disabled, so `useMemo` is needed
> there to avoid re-creating the promise each render and causing infinite suspend loops.

## Error handling

`use()` throws on rejection — wrap with an error boundary:

```tsx
function Root() {
  return (
    <ErrorBoundary fallback={<ErrorScreen />}>
      <Suspense fallback={<Loading />}>
        <DataComponent />
      </Suspense>
    </ErrorBoundary>
  );
}
```

## When NOT to use `use()`

- **Polling / retry logic** — `use()` doesn't retry. For the ai_triage job-status poll, use a
  `setInterval` + `apiFetch` hook wrapped in `useTransition` instead.
- **Debounced updates** — `use()` refetches on every prop change. Use `useEffect` with cleanup.
- **Complex multi-promise orchestration** — stick with `useEffect` + `AbortController`.
- **Requests needing CSRF mutation handling** — `apiFetch` already handles CSRF for mutating
  methods, but if you need fine-grained abort/cancel, `useEffect` with an `AbortController` is
  clearer (see existing hooks like `use-clients.ts`).

## Streaming LLM responses (R-04)

For the chat clarifier's streaming response, wrap the streaming promise in `use()` under a
Suspense boundary so the UI shows a "thinking…" fallback while tokens arrive:

```tsx
function ClarifierBubble({ streamPromise }: { streamPromise: Promise<string> }) {
  const text = use(streamPromise);
  return <BotMessage text={text} />;
}

<Suspense fallback={<Loading label={t('intake:thinking')} />}>
  <ClarifierBubble streamPromise={clarifierStream} />
</Suspense>
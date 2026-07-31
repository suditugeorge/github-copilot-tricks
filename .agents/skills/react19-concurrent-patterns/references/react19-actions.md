---
title: React 19 Actions Pattern Reference
---

# React 19 Actions — noviapet-web Reference

React 19 **Actions** handle async operations (form submissions, chat sends, uploads) with built-in
loading state, error handling, and optimistic updates. This replaces the `useReducer + onSubmit`
boilerplate.

> **noviapet-web is a client SPA.** "Actions" here means the client-side `useActionState` pattern,
> NOT React Server Components / Server Actions. The async function still calls `apiFetch`.

## `useActionState()`

Replaces `useReducer + useEffect` for form handling.

### Old pattern (avoid)

```tsx
function ChatInput() {
  const [state, dispatch] = useReducer(chatReducer, { loading: false, error: null, data: null });

  async function handleSubmit(e: FormEvent) {
    e.preventDefault();
    dispatch({ type: 'loading' });
    try {
      const data = await apiFetch<ChatResponse>('/api/intake/messages', { method: 'POST', body: ... });
      dispatch({ type: 'success', data });
    } catch (err) {
      dispatch({ type: 'error', error: (err as Error).message });
    }
  }

  return <form onSubmit={handleSubmit}>…</form>;
}
```

### React 19 pattern

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

**Differences:**

- One hook instead of `useReducer` + manual dispatch logic
- `formAction` replaces `onSubmit`; the form automatically collects `FormData`
- `isPending` is a boolean — no dispatch calls
- The action receives `(prevState, formData)`

## `useFormStatus()`

A **child component** hook that reads pending state from the nearest `<form action>`. Eliminates
prop-drilling of `isPending`.

```tsx
import { useFormStatus } from 'react';
import { useTranslation } from 'react-i18next';

function SendButton() {
  const { pending } = useFormStatus();
  const { t } = useTranslation();
  return (
    <button type="submit" disabled={pending}>
      {pending ? t('intake:sending') : t('intake:send')}
    </button>
  );
}

function ChatInput() {
  const [state, formAction] = useActionState(sendChatMessageAction, { error: null, data: null });
  return (
    <form action={formAction}>
      <input name="message" />
      <SendButton /> {/* no isPending prop needed */}
    </form>
  );
}
```

> `useFormStatus` only works inside a `<form action={...}>`. A plain `<form onSubmit>` will NOT
> trigger it.

## `useOptimistic()`

Updates the UI immediately while an async operation is in-flight. On success the confirmed data
replaces the optimistic value; on failure the UI reverts automatically.

### Old pattern (avoid)

```tsx
function ChatMessageList({ messages }: { messages: ChatMessage[] }) {
  const [optimistic, setOptimistic] = useState(messages);

  async function handleSend(text: string) {
    const tempMessage = { id: crypto.randomUUID(), role: 'user' as const, text };
    setOptimistic((prev) => [...prev, tempMessage]);
    try {
      const confirmed = await apiFetch<ChatMessage>('/api/intake/messages', { method: 'POST', body: ... });
      setOptimistic((prev) => [...prev.filter((m) => m.id !== tempMessage.id), confirmed]);
    } catch {
      setOptimistic(messages); // revert
    }
  }
  // ...
}
```

### React 19 pattern

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

**Key points:**

- `useOptimistic(currentState, updateFunction)`
- `updateFunction` receives `(state, optimisticInput)` and returns the new state
- Call `addOptimistic(input)` to trigger the optimistic update
- The confirmed value replaces the optimistic state when the action resolves

## Full example: AI intake chat send (all three hooks)

```tsx
import { useActionState, useFormStatus, useOptimistic } from 'react';
import { apiFetch } from '@/lib/api-client';
import { useTranslation } from 'react-i18next';

async function sendIntakeMessageAction(
  prevState: { error: string | null; data: ChatResponse | null },
  formData: FormData,
): Promise<typeof prevState> {
  const text = String(formData.get('message') ?? '');
  if (!text.trim()) return { error: null, data: null };
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

function SendButton() {
  const { pending } = useFormStatus();
  const { t } = useTranslation();
  return (
    <button type="submit" disabled={pending}>
      {pending ? t('intake:sending') : t('intake:send')}
    </button>
  );
}

function IntakeChat({ initialMessages }: { initialMessages: ChatMessage[] }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    initialMessages,
    (state, newMessage: ChatMessage) => [...state, newMessage],
  );

  const [state, formAction] = useActionState(sendIntakeMessageAction, { error: null, data: null });

  async function handleSend(formData: FormData) {
    const text = String(formData.get('message') ?? '');
    addOptimisticMessage({ id: crypto.randomUUID(), role: 'user', text });
    await formAction(formData);
  }

  return (
    <>
      <ul>
        {optimisticMessages.map((m) => (
          <li key={m.id}>{m.text}</li>
        ))}
      </ul>
      {state.error && <p role="alert">{state.error}</p>}
      <form action={handleSend}>
        <input name="message" />
        <SendButton />
      </form>
    </>
  );
}
```

## Comparison table

| Feature | Old (useReducer + onSubmit) | React 19 |
|---|---|---|
| Form handling | `onSubmit` + `useReducer` | `action` + `useActionState` |
| Loading state | Manual dispatch | Automatic `isPending` |
| Child pending state | Prop drilling | `useFormStatus` |
| Optimistic updates | Manual state dance | `useOptimistic` |
| Error handling | Manual in dispatch | Return from action |
| Complexity | More boilerplate | Less boilerplate |

## React Compiler note

All three hooks are compiler-safe. Do not wrap the action function in `useCallback` — the compiler
handles reference stability. In tests (compiler disabled), the action function is recreated each
render, which is fine because `useActionState` tracks it internally.
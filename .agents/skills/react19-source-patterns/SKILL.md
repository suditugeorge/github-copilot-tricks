---
name: react19-source-patterns
description: >
  React 19 source-file authoring patterns for the noviapet-web frontend (React 19.2 + TypeScript
  strict + shadcn/ui). Use this skill when creating or refactoring components that involve refs,
  context providers, default props, or any API that changed in React 19. Covers ref-as-prop
  (forwardRef no longer needed), <Context> as provider, defaultProps removal, useRef(null),
  createRoot, and removed APIs (findDOMNode, string refs, legacy context). Trigger for prompts
  like "forwardRef", "ref prop", "context provider", "createContext", "useRef", "defaultProps",
  or when wiring refs into shadcn/ui primitives in noviapet-web.
---

# React 19 Source Patterns — noviapet-web

noviapet-web is already on **React 19.2**, so this skill is an **authoring reference** for writing
correct React 19 source code — not a migration guide. Use it whenever you create components that
touch refs, context, or any API whose shape changed in React 19.

## Quick reference table

| Pattern | React 19 correct form | Reference |
|---|---|---|
| `ReactDOM.render(...)` | `createRoot().render()` | `references/api-migrations.md` |
| `ReactDOM.hydrate(...)` | `hydrateRoot(...)` | `references/api-migrations.md` |
| `unmountComponentAtNode` | `root.unmount()` | `references/api-migrations.md` |
| `ReactDOM.findDOMNode` | direct ref | `references/api-migrations.md` |
| `forwardRef(...)` wrapper | accept `ref` as a direct prop | `references/api-migrations.md` |
| `Component.defaultProps = {}` | ES6 default params | `references/api-migrations.md` |
| `useRef()` no arg | `useRef(null)` | `references/api-migrations.md` |
| Legacy Context (`contextTypes`) | `createContext` | `references/api-migrations.md` |
| String refs `this.refs.x` | `createRef()` or callback ref | `references/api-migrations.md` |
| `<Context.Provider>` | `<Context>` (provider is optional) | inline below |
| `import React from 'react'` (unused) | remove | inline below |

## ref as a direct prop (no `forwardRef`)

React 19 passes `ref` as a regular prop. Do NOT wrap components in `forwardRef`.

```tsx
// ✅ React 19 — ref is a normal prop:
function ChatBubble({ ref, className, ...props }: ChatBubbleProps) {
  return <div ref={ref} className={cn('chat-bubble', className)} {...props} />;
}

// Usage:
const bubbleRef = useRef<HTMLDivElement>(null);
<ChatBubble ref={bubbleRef} />;
```

`useImperativeHandle` still works — only the `forwardRef` wrapper is removed:

```tsx
function ChatScrollContainer({ ref, ...props }: ChatScrollContainerProps) {
  const innerRef = useRef<HTMLDivElement>(null);
  useImperativeHandle(ref, () => ({
    scrollToBottom: () => {
      if (innerRef.current) innerRef.current.scrollTop = innerRef.current.scrollHeight;
    },
  }));
  return <div ref={innerRef} {...props} />;
}
```

> **shadcn/ui note:** shadcn primitives may still ship `forwardRef` internally for backwards
> compatibility. That is fine — `forwardRef` still works in React 19, it is just optional. When
> writing NEW noviapet-web components, prefer the direct-ref-prop form.

## `<Context>` as provider

React 19 lets you render `<Context>` directly as the provider — `<Context.Provider>` still works
but is no longer required:

```tsx
const IntakeContext = createContext<IntakeSession | null>(null);

// ✅ React 19 — Context itself is the provider:
function IntakeChatPage({ children }: { children: ReactNode }) {
  const session = useState<IntakeSession>(...);
  return <IntakeContext value={session}>{children}</IntakeContext>;
}

// <IntakeContext.Provider value={...}> still works too — both are valid.
```

## defaultProps — use ES6 default params

`defaultProps` is removed in React 19. Use default parameter values:

```tsx
// ✅ React 19:
function UrgencyBadge({ urgency = 'routine', className }: UrgencyBadgeProps) {
  return <span className={cn(urgencyColor(urgency), className)}>{urgency}</span>;
}

// ❌ Removed in React 19:
// UrgencyBadge.defaultProps = { urgency: 'routine' };
```

## `useRef(null)` — always pass an initial value

```tsx
// ✅ React 19:
const scrollRef = useRef<HTMLDivElement>(null);

// ❌ React 19 requires an argument:
// const scrollRef = useRef<HTMLDivElement>();
```

## Unused `React` import

noviapet-web uses the automatic JSX runtime, so `import React from 'react'` is only needed when
you reference `React.*` APIs. With named imports preferred:

```tsx
// ✅ Preferred — named imports:
import { useState, useRef, useActionState } from 'react';

// Only import the default if you use React.* :
import React, { useState } from 'react';
```

## PropTypes rule

noviapet-web uses **TypeScript strict mode** and does not use `PropTypes`. Do not introduce
`.propTypes` assignments. If you ever encounter them, React 19 no longer runs propTypes validation
at runtime — TypeScript interfaces are the source of truth here.

## Read the reference

For full before/after code for each API change (including `forwardRef` + `useImperativeHandle`
edge cases, `defaultProps` null-vs-undefined behavior, and legacy context cross-file migrations),
read **`references/api-migrations.md`**.
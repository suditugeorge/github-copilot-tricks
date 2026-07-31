---
title: React 19 API Migrations Reference
---

# React 19 API Reference — noviapet-web

Complete before/after patterns for every React 19 API change. noviapet-web is already on React 19.2,
so use this as the **correct authoring form** reference, not a migration checklist.

---

## ReactDOM Root API

React 19 requires `createRoot()` or `hydrateRoot()`. `src/main.tsx` already uses `createRoot`.

### `createRoot()` — CSR app

```tsx
// ✅ React 19:
import { createRoot } from 'react-dom/client';
const root = createRoot(document.getElementById('root')!);
root.render(<App />);
```

### `hydrateRoot()` — SSR/static (not used in noviapet-web SPA, but for reference)

```tsx
import { hydrateRoot } from 'react-dom/client';
hydrateRoot(document.getElementById('root')!, <App />);
```

### `unmountComponentAtNode()` removed

```tsx
// ❌ Removed:
ReactDOM.unmountComponentAtNode(container);

// ✅ React 19:
const root = createRoot(container);
root.unmount();
```

---

## `findDOMNode()` removed

Use direct refs:

```tsx
// ❌ Removed:
const domNode = findDOMNode(componentRef);

// ✅ React 19:
const domNode = componentRef.current;
```

---

## `forwardRef()` — optional modernization

React 19 passes `ref` as a regular prop. `forwardRef` still works but is no longer needed.

### Function component direct ref

```tsx
// ❌ Old (React 18):
const Input = forwardRef<HTMLInputElement, InputProps>((props, ref) => (
  <input ref={ref} {...props} />
));

// ✅ React 19:
function Input({ ref, ...props }: InputProps) {
  return <input ref={ref} {...props} />;
}
```

### `forwardRef` + `useImperativeHandle`

```tsx
// ✅ React 19 — useImperativeHandle still valid, only forwardRef wrapper removed:
function TextInput({ ref, ...props }: TextInputProps) {
  const inputRef = useRef<HTMLInputElement>(null);
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current?.focus(),
    clear: () => {
      if (inputRef.current) inputRef.current.value = '';
    },
  }));
  return <input ref={inputRef} {...props} />;
}
```

> **shadcn/ui caveat:** shadcn primitives may still use `forwardRef` internally for backwards
> compatibility. That is fine. Use the direct-ref-prop form for NEW noviapet-web components.

---

## `defaultProps` removed

Use ES6 default params:

```tsx
// ❌ Removed in React 19:
function Button({ label, disabled }: ButtonProps) { /* ... */ }
Button.defaultProps = { label: 'Click', disabled: false };

// ✅ React 19:
function Button({ label = 'Click', disabled = false }: ButtonProps) {
  return <button disabled={disabled}>{label}</button>;
}
```

> noviapet-web uses TypeScript interfaces with optional props (`label?: string`). Default param
> values handle the `undefined` case. Note: `defaultProps` treated `null` as "use default";
> ES6 default params only apply when the prop is `undefined`, not `null`. If a caller explicitly
> passes `null`, the default is NOT applied — handle that explicitly if needed.

---

## `useRef()` requires an initial value

```tsx
// ❌ React 19 requires an argument:
const ref = useRef<HTMLDivElement>();

// ✅ React 19:
const ref = useRef<HTMLDivElement>(null);
```

---

## Legacy Context API removed

The old `contextTypes` / `childContextTypes` / `getChildContext` pattern is completely removed.
Use `createContext`:

```tsx
// ✅ React 19:
const IntakeContext = createContext<IntakeSession | null>(null);

function useIntakeContext(): IntakeSession {
  const ctx = useContext(IntakeContext);
  if (!ctx) throw new Error('useIntakeContext must be used within <IntakeContext>');
  return ctx;
}

// Provider — <Context> itself works as the provider in React 19:
function IntakeProvider({ children }: { children: ReactNode }) {
  const [session, setSession] = useState<IntakeSession>(initialSession);
  return <IntakeContext value={session}>{children}</IntakeContext>;
}
```

> noviapet-web's `AuthContext` (`src/auth/auth-context.tsx`) already uses `createContext`. Follow
> that pattern for new contexts. Always export a `use<Name>Context` hook that throws if the
> context is null, so consumers get a clear error when used outside the provider.

---

## String refs removed

```tsx
// ❌ Removed:
class Component extends React.Component {
  render() {
    return <input ref="inputRef" />;
  }
}

// ✅ React 19 — createRef:
class Component extends React.Component {
  inputRef = createRef<HTMLInputElement>();
  render() {
    return <input ref={this.inputRef} />;
  }
}

// ✅ React 19 — callback ref (more flexible):
function Component() {
  const inputRef = useRef<HTMLInputElement>(null);
  return <input ref={inputRef} />;
}
```

> noviapet-web uses function components exclusively — prefer `useRef(null)`.

---

## Unused `React` import

```tsx
// ❌ Only needed if you reference React.* :
import React from 'react';
function Component() { return <div />; }

// ✅ With automatic JSX runtime (noviapet-web default):
function Component() { return <div />; }

// ✅ Keep the import only if you use React.* APIs:
import React, { useState } from 'react';
```

noviapet-web prefers **named imports** (`import { useState } from 'react'`).

---

## Complete audit commands (for verifying a codebase)

```bash
# 1. ReactDOM.render calls (should be createRoot):
grep -rn "ReactDOM.render" src/ --include="*.ts" --include="*.tsx"

# 2. forwardRef usages (optional to remove):
grep -rn "forwardRef" src/ --include="*.ts" --include="*.tsx"

# 3. defaultProps assignments (must remove):
grep -rn "\.defaultProps\s*=" src/ --include="*.ts" --include="*.tsx"

# 4. useRef() without initial value (must add null):
grep -rn "useRef()" src/ --include="*.ts" --include="*.tsx"

# 5. Legacy context (must migrate):
grep -rn "contextTypes\|childContextTypes\|getChildContext" src/ --include="*.ts" --include="*.tsx"

# 6. String refs (must migrate):
grep -rn 'ref="' src/ --include="*.ts" --include="*.tsx"

# 7. findDOMNode (must remove):
grep -rn "findDOMNode" src/ --include="*.ts" --include="*.tsx"
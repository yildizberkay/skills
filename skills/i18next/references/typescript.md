# i18next TypeScript Reference

## Table of Contents
1. [Setup](#setup)
2. [Declaration File](#declaration-file)
3. [Selector API (v25.4+)](#selector-api)
4. [CustomTypeOptions](#customtypeoptions)
5. [Common Patterns](#common-patterns)
6. [Troubleshooting](#troubleshooting)

---

## Setup

Requirements:
- TypeScript v5+ (v4 only supported with i18next@22.5.1)
- `strict: true` or `strictNullChecks: true` in tsconfig
- Optional: `skipLibCheck: true` to avoid `HTMLAttributes` type conflicts

Type-safe translations are optional and may affect compilation time on large projects.

---

## Declaration File

Create `@types/i18next.d.ts` (or `src/@types/i18next.d.ts`) to augment i18next types:

### Basic approach (import JSON directly)

```typescript
// @types/i18next.d.ts
import "i18next";
import ns1 from "locales/en/ns1.json";
import ns2 from "locales/en/ns2.json";

declare module "i18next" {
  interface CustomTypeOptions {
    defaultNS: "ns1";
    resources: {
      ns1: typeof ns1;
      ns2: typeof ns2;
    };
  }
}
```

### Recommended approach (via i18n config)

**i18n.ts:**
```typescript
import ns1 from './locales/en/ns1.json';
import ns2 from './locales/en/ns2.json';

export const defaultNS = "ns1";
export const resources = {
  en: { ns1, ns2 },
} as const;

i18next.init({
  lng: "en",
  ns: ["ns1", "ns2"],
  defaultNS,
  resources,
});
```

**@types/i18next.d.ts:**
```typescript
import { resources, defaultNS } from "./i18n";

declare module "i18next" {
  interface CustomTypeOptions {
    defaultNS: typeof defaultNS;
    resources: typeof resources["en"];
  }
}
```

Important: Only import the default language resources for type definitions. The `resources` type provides the key structure.

---

## Selector API

Since v25.4, i18next supports a selector-based API for type-safe key access. This will become the default in v26.

### Enable the selector API

```typescript
// @types/i18next.d.ts
declare module "i18next" {
  interface CustomTypeOptions {
    enableSelector: true; // or "optimize" for large projects
    // ... other options
  }
}
```

### Usage

```typescript
// Selector style (v25.4+)
i18next.t($ => $.key);
i18next.t($ => $.look.deep);
i18next.t($ => $.key, { what: 'i18next', how: 'great' });
i18next.t($ => $.friend, { context: 'male' });
i18next.t($ => $.item, { count: 5 });

// Access different namespace
i18next.t($ => $.button.save, { ns: 'common' });

// Fallback value
i18next.t($ => $.unknownKey, { defaultValue: t($ => $.fallbackKey) });
```

### "optimize" mode

Setting `enableSelector: "optimize"` handles arbitrarily large translation sets without IDE performance degradation. However, translation keys are not modified (no automatic plural/context suffix resolution), so you need to specify the correct key or use `@i18next-selector/vite-plugin`.

### Testing with selectors

Use `keyFromSelector` to extract the key string for mocking:

```typescript
import { keyFromSelector } from "i18next";

const mockT = (selector: ($: Record<string, any>) => any) => keyFromSelector(selector);
const key = mockT($ => $.abc.def);
console.log(key); // => "abc.def"
```

---

## CustomTypeOptions

All available options for type configuration:

| Option | Default | Description |
|--------|---------|-------------|
| `enableSelector` | `false` | Enable selector API. `true` or `"optimize"` |
| `defaultNS` | `'translation'` | Default namespace for `useTranslation()` type inference |
| `resources` | object | Key structure from primary language files |
| `fallbackNS` | `false` | Fallback namespace(s) |
| `keySeparator` | `'.'` | Key separator character |
| `nsSeparator` | `':'` | Namespace separator character |
| `pluralSeparator` | `'_'` | Plural separator character |
| `contextSeparator` | `'_'` | Context separator character |
| `returnNull` | `true` | Allow null as return type |
| `returnObjects` | `false` | Allow object return types |
| `compatibilityJSON` | `'v4'` | Only `'v4'` supported for types |
| `allowObjectInHTMLChildren` | `false` | For React: allow objects in HTML elements (Trans component) |
| `interpolationPrefix` | `'{{'` | Interpolation prefix |
| `interpolationSuffix` | `'}}'` | Interpolation suffix |
| `strictKeyChecks` | `false` | Enforce valid keys even with defaultValue |

### Disabling null return type

```typescript
// @types/i18next.d.ts
declare module "i18next" {
  interface CustomTypeOptions {
    returnNull: false;
  }
}
```

Also update runtime config to match:
```javascript
i18next.init({ returnNull: false });
```

---

## Common Patterns

### React with react-i18next (typed)

```typescript
// i18n.ts
import i18next from 'i18next';
import { initReactI18next } from 'react-i18next';
import common from './locales/en/common.json';
import app from './locales/en/app.json';

export const defaultNS = 'app';
export const resources = { en: { common, app } } as const;

i18next.use(initReactI18next).init({
  lng: 'en',
  defaultNS,
  resources
});

// @types/i18next.d.ts
import { resources, defaultNS } from '../i18n';
declare module 'i18next' {
  interface CustomTypeOptions {
    defaultNS: typeof defaultNS;
    resources: typeof resources['en'];
  }
}

// Component.tsx
import { useTranslation } from 'react-i18next';

function MyComponent() {
  const { t } = useTranslation(); // uses defaultNS 'app'
  return <h1>{t($ => $.title)}</h1>; // type-safe!
}
```

### Multiple i18next instances

For projects with multiple instances using different translation resources, create separate `tsconfig.json` and `i18next.d.ts` files per instance. See [i18next/i18next-monorepo](https://github.com/i18next/i18next-monorepo) for an example.

---

## Troubleshooting

### Intellisense not working
- Update TypeScript to v5+
- Ensure `strict: true` or `strictNullChecks: true` in tsconfig
- Verify declaration file is in scope

### Out of memory (OOM) errors

Best fix: use `enableSelector: "optimize"` — it handles large translation sets efficiently.

Other mitigations:
- Split typecheck and lint into separate tasks
- Use a monorepo structure
- Increase Node memory: `NODE_OPTIONS="--max_old_space_size=10240" tsc`

Migration tools for selector API:
- `@i18next-selector/codemod` — automatic code migration
- `@i18next-selector/vite-plugin` — build-time key resolution

### Template literal type errors (non-selector API only)

If using string keys with template literals:
```typescript
// Error: Argument of type 'string' is not assignable...
t(`${expression}.title`);

// Fix: assert as const
t(`${expression}.title` as const);
```

This is a TypeScript limitation that doesn't affect the selector API.

### "Excessively deep and possibly infinite" error

Usually indicates incorrect type declaration setup. Review your `i18next.d.ts` configuration. Does not occur with `enableSelector: "optimize"`.

### Interpolation values not inferred

Interpolation type inference requires translation files to be either:
- TypeScript files using `as const`
- Interfaces in `.d.ts` files

JSON files don't support `as const` natively. Generate types from JSON using build tools.

### `HTMLAttributes<T>` type error

Set `skipLibCheck: true` in tsconfig.

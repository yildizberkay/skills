# i18next Patterns, Plugins & Best Practices

## Table of Contents
1. [Best Practices](#best-practices)
2. [Namespace Organization](#namespace-organization)
3. [Plugin Ecosystem](#plugin-ecosystem)
4. [Creating Custom Plugins](#creating-custom-plugins)
5. [Caching Strategies](#caching-strategies)
6. [Common Pitfalls](#common-pitfalls)
7. [Migration Guide (v3 → v4 JSON format)](#migration-guide)
8. [Framework Integration Patterns](#framework-integration-patterns)

---

## Best Practices

### Interpolation and localization

Use interpolation sparingly. Sentence fragments with interpolated values are difficult to translate because grammar rules differ across languages (word order, articles, gender agreement).

**Bad** — grammatically broken in many languages:
```json
{ "key": "All fees will be charged to the {{paymentType}} on file." }
```
In German, "the" changes based on the gender of `paymentType` (der/die/das), making this untranslatable.

**Good** — self-contained strings:
```json
{
  "feesCard": "All fees will be charged to the credit card on file.",
  "feesPaypal": "All fees will be charged to the PayPal account on file."
}
```

**When interpolation IS appropriate:**
- Timestamps and dates (only known at runtime)
- User-inputted data (names, quantities)
- Numbers that change dynamically

### Key naming conventions

- Use descriptive, hierarchical keys: `user.profile.title`, `error.network.timeout`
- For natural language keys, disable separators: `keySeparator: false, nsSeparator: false`
- Avoid reserved characters (`:` and `.`) in keys unless separators are disabled
- Keep keys short but meaningful

### Language code format

Use BCP 47 format: `en`, `en-US`, `zh-Hant-HK`
- Dash separator (not underscore): `en-US` not `en_US`
- As soon as you use `-`, codes are formatted with `Intl.getCanonicalLocales`

---

## Namespace Organization

### Semantic namespaces

```
locales/en/
├── common.json     → Reused everywhere: 'save', 'cancel', 'delete'
├── validation.json → All validation messages
├── glossary.json   → Consistent terminology
├── errors.json     → Error messages
└── app.json        → Main application content
```

### Feature-based namespaces (large apps)

```
locales/en/
├── common.json
├── auth.json       → Login, signup, password reset
├── dashboard.json  → Dashboard-specific content
├── settings.json   → Settings page
└── admin.json      → Admin area (lazy loaded)
```

### Namespace setup

```javascript
i18next.init({
  ns: ['common', 'app', 'validation'],
  defaultNS: 'app',
  fallbackNS: 'common' // shared keys like 'save', 'cancel'
});

// Load additional namespaces on demand
i18next.loadNamespaces('admin', (err) => { /* ready */ });
```

---

## Plugin Ecosystem

### Backend Plugins (loading translations)

| Plugin | Use Case |
|--------|----------|
| `i18next-http-backend` | Load via HTTP/fetch (browser) |
| `i18next-fs-backend` | Load from filesystem (Node.js, Deno) |
| `i18next-locize-backend` | Load from Locize TMS CDN |
| `i18next-resources-to-backend` | Convert resources to lazy-loading backend (webpack/vite dynamic imports) |
| `i18next-chained-backend` | Chain multiple backends with fallback |
| `i18next-localstorage-backend` | Cache in localStorage |

### Language Detectors

| Plugin | Use Case |
|--------|----------|
| `i18next-browser-languagedetector` | Detect from querystring, cookie, localStorage, navigator, htmlTag |
| `i18next-http-middleware` | Server-side detection (Express, etc.) |

### Post-Processors

| Plugin | Use Case |
|--------|----------|
| `i18next-sprintf-postprocessor` | sprintf-style formatting |
| `i18next-intervalplural-postprocessor` | Interval-based plural ranges |

### Framework Integrations

| Integration | Framework |
|-------------|-----------|
| `react-i18next` | React (hooks, HOC, Trans component) |
| `next-i18next` | Next.js |
| `i18next-vue` | Vue.js |
| `angular-i18next` | Angular |
| `jquery-i18next` | jQuery |
| `i18next-icu` | ICU message format |

### Plugin usage pattern

```javascript
import i18next from 'i18next';
import Backend from 'i18next-http-backend';
import LanguageDetector from 'i18next-browser-languagedetector';
import sprintf from 'i18next-sprintf-postprocessor';

i18next
  .use(Backend)
  .use(LanguageDetector)
  .use(sprintf)
  .init(options);
```

All `.use()` calls must come before `.init()`.

---

## Creating Custom Plugins

### Backend plugin

```javascript
const MyBackend = {
  type: 'backend',
  init(services, backendOptions, i18nextOptions) {
    // setup
  },
  read(language, namespace, callback) {
    // fetch translations
    callback(null, { key: 'value' }); // or callback(error)
  },
  // optional: save missing keys
  create(languages, namespace, key, fallbackValue) {
    // save the missing key
  }
};
```

### Language detector plugin

```javascript
const MyDetector = {
  type: 'languageDetector',
  init(services, detectorOptions, i18nextOptions) {},
  detect() {
    return 'en-US'; // return detected language
  },
  cacheUserLanguage(lng) {
    // store the language choice
  }
};
```

### Formatter plugin

```javascript
const MyFormatter = {
  type: 'formatter',
  init(services, backendOptions, i18nextOptions) {},
  format(value, format, lng, options) {
    if (!format && value instanceof Date) {
      return value.toUTCString();
    }
  },
  add(name, fc) {},
  addCached(name, fc) {}
};

i18next
  .use(MyFormatter)
  .init({
    interpolation: { alwaysFormat: true }
  });
```

### Post-processor plugin

```javascript
const MyPostProcessor = {
  type: 'postProcessor',
  name: 'myProcessor',
  process(value, key, options, translator) {
    return value.toUpperCase();
  }
};
```

---

## Caching Strategies

### localStorage caching with chained backend

```javascript
import i18next from 'i18next';
import ChainedBackend from 'i18next-chained-backend';
import LocalStorageBackend from 'i18next-localstorage-backend';
import HttpBackend from 'i18next-http-backend';

i18next
  .use(ChainedBackend)
  .init({
    backend: {
      backends: [LocalStorageBackend, HttpBackend],
      backendOptions: [
        { expirationTime: 7 * 24 * 60 * 60 * 1000 }, // 7 days
        { loadPath: '/locales/{{lng}}/{{ns}}.json' }
      ]
    }
  });
```

### Serverless environments

Don't use remote backends in serverless — the cache may be purged between invocations. Package translations with your function instead.

---

## Common Pitfalls

1. **Calling init() multiple times**: Use `changeLanguage()` instead. For separate configs, use `createInstance()`.

2. **Using t() before init completes**: Always await the Promise or use the callback.

3. **Missing count variable in plurals**: `i18next.t('key', { count: 1 })` — `count` is required; without it, there's no fallback to the base key.

4. **Wrong plural suffixes**: v4 uses `_one`/`_other`, not `_plural`. Check console warnings about Intl.PluralRules.

5. **Underscore in language codes**: Use `en-US` (BCP 47), not `en_US`.

6. **Backend + inline resources conflict**: They don't automatically fallback to each other. Use `i18next-chained-backend` for fallback behavior.

7. **Empty resources object**: Pass `resources: {}` if adding translations after init to avoid "no backend" warning.

8. **saveMissing without proper backend**: Needs a backend that supports the `create` function (i18next-http-backend, i18next-fs-backend, i18next-locize-backend).

9. **Escaping in React**: React already escapes, so react-i18next sets `escapeValue: false`. Don't double-escape.

10. **Nesting with v21+ skipOnVariables**: If passing `$t()` as a variable value, you need `skipOnVariables: false` in interpolation options.

---

## Migration Guide

### v3 → v4 JSON format (i18next v21+)

The only change is plural suffixes:

| v3 | v4 |
|----|-----|
| `key` / `key_plural` | `key_one` / `key_other` |
| `key_0` through `key_5` | `key_zero` through `key_other` |

Migration tools:
- CLI: [i18next-v4-format-converter](https://github.com/i18next/i18next-v4-format-converter)
- Web: [online converter](https://i18next.github.io/i18next-v4-format-converter-web/)

To keep v3 format temporarily:
```javascript
i18next.init({ compatibilityJSON: 'v3' });
```

### skipOnVariables change in v21

`skipOnVariables` defaults to `true` since v21. This means `$t()` and `{{}}` inside variable values are NOT resolved. To restore old behavior:
```javascript
i18next.init({
  interpolation: { skipOnVariables: false }
});
```

Keep `true` (default) when interpolation variables are user-provided for security.

---

## Framework Integration Patterns

### React (react-i18next)

```tsx
import { useTranslation, Trans } from 'react-i18next';

function MyComponent() {
  const { t, i18n } = useTranslation('common');

  return (
    <div>
      <h1>{t('title')}</h1>
      <p>{t('greeting', { name: 'World' })}</p>
      <Trans i18nKey="richText">
        Click <a href="/link">here</a> for more.
      </Trans>
      <button onClick={() => i18n.changeLanguage('de')}>
        Switch to German
      </button>
    </div>
  );
}
```

### Next.js (next-i18next / app directory)

For App Router, use `next-intl` or configure i18next with server components. For Pages Router, use `next-i18next` with `serverSideTranslations`.

### Language switcher pattern

```javascript
i18next.on('languageChanged', (lng) => {
  document.documentElement.setAttribute('lang', lng);
  document.documentElement.setAttribute('dir', i18next.dir(lng));
});
```

### E2E testing

```javascript
i18next.init({ lng: 'cimode' }); // t() always returns the key
// With namespace: set appendNamespaceToCIMode: true
```

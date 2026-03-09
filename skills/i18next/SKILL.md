---
name: i18next
description: "Comprehensive guide for implementing internationalization (i18n) with i18next in JavaScript/TypeScript projects. Use this skill whenever the user asks about i18next setup, configuration, translation functions, pluralization, interpolation, formatting, context, namespaces, TypeScript integration, language detection, or any localization-related task involving i18next. Also trigger when the user mentions react-i18next, next-i18next, i18next plugins, translation JSON files with i18next format, or wants to add multi-language support to a web/mobile/Node.js application using i18next. Even if the user just mentions 'i18n', 'internationalization', 'localization', or 'translations' in a JavaScript/TypeScript context, this skill is likely relevant."
---

# i18next Internationalization Skill

i18next is a JavaScript internationalization framework that provides a complete solution for localizing products across web, mobile, and desktop platforms. It goes beyond standard i18n features with support for plurals, context, interpolation, formatting, namespaces, and a rich plugin ecosystem.

## When to read reference files

This skill is organized with reference files for deeper topics. Read the relevant reference file **before** writing code:

- **`references/configuration.md`** — Read when setting up i18next init, configuring languages, namespaces, fallbacks, missing keys, or loading translations. Also read for backend plugins, language detection, and resource loading strategies.
- **`references/translation-functions.md`** — Read when using `t()` function features: interpolation, plurals, context, formatting (number, currency, datetime, relative time, list), nesting, objects/arrays, or any translation string patterns.
- **`references/typescript.md`** — Read when setting up type-safe translations, creating declaration files, configuring CustomTypeOptions, using the selector API (`enableSelector`), or troubleshooting TypeScript issues.
- **`references/patterns-and-plugins.md`** — Read when implementing best practices, common patterns (namespace organization, key naming, fallback strategies), creating custom plugins, or migrating between i18next versions.

## Quick Start

Install i18next:

```bash
npm install i18next --save
```

Basic initialization:

```javascript
import i18next from 'i18next';

i18next.init({
  lng: 'en', // omit if using a language detector
  debug: true,
  resources: {
    en: {
      translation: {
        "key": "hello world"
      }
    }
  }
});

// Use translations
i18next.t('key'); // -> "hello world"
```

init() returns a Promise and also accepts a callback. Always wait for initialization to complete before calling `t()`.

## Core Concepts Overview

### Translation Function (`t`)

The `t()` function is the primary way to access translations. It supports:
- **Simple keys**: `i18next.t('key')` or nested `i18next.t('look.deep')`
- **Default values**: `i18next.t('key', 'default value')`
- **Namespace access**: `i18next.t('button.save', { ns: 'common' })`
- **Multiple fallback keys**: `i18next.t(['error.404', 'error.unspecific'])`
- **Language override**: `i18next.t('key', { lng: 'de' })`

### Interpolation

Dynamic values in translations use `{{variable}}` syntax:

```json
{ "greeting": "Hello {{name}}, you have {{count}} messages" }
```
```javascript
i18next.t('greeting', { name: 'John', count: 5 });
// -> "Hello John, you have 5 messages"
```

Values are escaped by default to prevent XSS. Use `{{- variable}}` for unescaped output.

### Plurals (JSON v4 format)

Pluralization uses the Intl.PluralRules API with suffixes `_zero`, `_one`, `_two`, `_few`, `_many`, `_other`:

```json
{
  "item_one": "{{count}} item",
  "item_other": "{{count}} items"
}
```
```javascript
i18next.t('item', { count: 1 }); // -> "1 item"
i18next.t('item', { count: 5 }); // -> "5 items"
```

The `count` variable name is required and must be provided. Languages with multiple plural forms (like Arabic) use all six suffixes.

### Context

Differentiate translations based on context (e.g., gender):

```json
{
  "friend": "A friend",
  "friend_male": "A boyfriend",
  "friend_female": "A girlfriend"
}
```
```javascript
i18next.t('friend', { context: 'male' }); // -> "A boyfriend"
```

Context can be combined with plurals: `friend_male_one`, `friend_male_other`.

### Formatting (v21.3.0+)

Built-in Intl API-based formatters for number, currency, datetime, relativetime, and list:

```json
{ "price": "Total: {{val, currency(USD)}}" }
```
```javascript
i18next.t('price', { val: 2000 }); // -> "Total: $2,000.00"
```

### Namespaces

Separate translations into multiple files for organization and lazy loading:

```javascript
i18next.init({
  ns: ['common', 'moduleA'],
  defaultNS: 'moduleA',
  fallbackNS: 'common'
});
```

### Nesting

Reference other translation keys within translations using `$t()`:

```json
{
  "greeting": "Hello",
  "welcome": "$t(greeting), welcome back!"
}
```

### Key Configuration

- **keySeparator** (default `'.'`): Set to `false` for flat JSON structures
- **nsSeparator** (default `':'`): Separates namespace from key
- Keys should not contain `:` or `.` unless separators are disabled
- For natural language keys, set `keySeparator: false`, `nsSeparator: false`, and `fallbackLng: false`

## Common Setup Patterns

### React with react-i18next

```javascript
import i18next from 'i18next';
import { initReactI18next } from 'react-i18next';
import Backend from 'i18next-http-backend';
import LanguageDetector from 'i18next-browser-languagedetector';

i18next
  .use(Backend)
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    fallbackLng: 'en',
    debug: true,
    interpolation: { escapeValue: false } // React already escapes
  });
```

### Node.js with filesystem backend

```javascript
import i18next from 'i18next';
import Backend from 'i18next-fs-backend';

i18next
  .use(Backend)
  .init({
    initImmediate: false, // synchronous loading
    fallbackLng: 'en',
    backend: { loadPath: './locales/{{lng}}/{{ns}}.json' }
  });
```

### Lazy loading translations (webpack/vite)

```javascript
import resourcesToBackend from 'i18next-resources-to-backend';

i18next
  .use(resourcesToBackend((lng, ns) => import(`./locales/${lng}/${ns}.json`)))
  .init({ fallbackLng: 'en' });
```

## Important Caveats

1. **Do not call `init()` multiple times.** Use `changeLanguage()` to switch languages, or `createInstance()`/`cloneInstance()` for different configs.
2. **Wait for init** before using `t()` — use the callback, Promise, or async/await pattern.
3. **Language codes**: Use BCP 47 format (`en-US`, not `en_US`). Use dash, not underscore.
4. **Intl.PluralRules polyfill**: Required for plural support in older environments. Install `intl-pluralrules`.
5. **Interpolation and localization**: Use interpolation sparingly — sentence fragments with interpolated values can be difficult or impossible to translate correctly due to grammar differences across languages. Use separate self-contained strings when the value is known at build time.
6. **JSON v4 format** (default since v21): Uses `_one`/`_other` suffixes instead of the old `_plural`. Use [i18next-v4-format-converter](https://github.com/i18next/i18next-v4-format-converter) to migrate.
7. **Testing**: Set `lng: 'cimode'` to have `t()` always return the key — useful for e2e tests.

## API Quick Reference

| Method | Description |
|--------|-------------|
| `init(options, cb)` | Initialize i18next (returns Promise) |
| `t(key, options)` | Translate a key |
| `changeLanguage(lng, cb)` | Change current language (returns Promise) |
| `use(plugin)` | Register a plugin (call before init) |
| `exists(key, options)` | Check if a key exists |
| `getFixedT(lng, ns, keyPrefix)` | Get a t function with fixed lng/ns/prefix |
| `loadNamespaces(ns, cb)` | Load additional namespaces |
| `loadLanguages(lngs, cb)` | Load additional languages |
| `addResourceBundle(lng, ns, resources, deep, overwrite)` | Add translation resources |
| `getResource(lng, ns, key)` | Get a single translation value |
| `language` | Current detected/set language |
| `resolvedLanguage` | Current resolved language (for UI) |
| `languages` | Array of language codes for lookup |
| `dir(lng)` | Returns 'rtl' or 'ltr' |
| `createInstance(options, cb)` | Create a new independent instance |
| `cloneInstance(options)` | Clone sharing store and plugins |

## Events

```javascript
i18next.on('initialized', (options) => {});
i18next.on('languageChanged', (lng) => {});
i18next.on('loaded', (loaded) => {});
i18next.on('failedLoading', (lng, ns, msg) => {});
i18next.on('missingKey', (lngs, ns, key, res) => {}); // needs saveMissing: true
i18next.off('eventName', handler); // unsubscribe
```

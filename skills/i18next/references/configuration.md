# i18next Configuration Reference

## Table of Contents
1. [Init Options](#init-options)
2. [Language & Namespace Options](#language--namespace-options)
3. [Missing Keys Configuration](#missing-keys-configuration)
4. [Translation Defaults](#translation-defaults)
5. [Plugin Options](#plugin-options)
6. [Separator & Misc Options](#separator--misc-options)
7. [Loading Translations](#loading-translations)
8. [Fallback Strategies](#fallback-strategies)

---

## Init Options

`i18next.init(options, callback)` returns a Promise. The callback fires after translations are loaded or on error.

### Logging

| Option | Default | Description |
|--------|---------|-------------|
| `debug` | `false` | Logs info to console. Helps diagnose loading issues. |

---

## Language & Namespace Options

| Option | Default | Description |
|--------|---------|-------------|
| `resources` | `undefined` | Resources to initialize with (if not using a backend plugin) |
| `lng` | `undefined` | Language to use (overrides detection). Set to `'cimode'` for testing (t returns key). Use `'en-US'` format (BCP 47). |
| `fallbackLng` | `'dev'` | Fallback language. Set to `false` to disable. Can be string, array, object by language, or function. |
| `supportedLngs` | `false` | Array of allowed languages |
| `nonExplicitSupportedLngs` | `false` | If true, `en-US` is valid when `en` is in supportedLngs |
| `load` | `'all'` | Strategy: `'all'` → `['en-US', 'en', 'dev']`, `'currentOnly'` → `'en-US'`, `'languageOnly'` → `'en'` |
| `preload` | `false` | Array of languages to preload (important for SSR) |
| `lowerCaseLng` | `false` | Fully lowercase locale: `en-US` → `en-us` |
| `cleanCode` | `false` | Lowercase main language only: `EN` → `en`, keeps `en-US` |
| `ns` | `'translation'` | String or array of namespaces to load. Set to `[]` to load none on init. |
| `defaultNS` | `'translation'` | Default namespace. If `ns` is set without `defaultNS`, first namespace is used. Set to `false` to disable namespace fallback. |
| `fallbackNS` | `false` | String or array of fallback namespaces |
| `partialBundledLanguages` | `false` | Allow some resources on init while loading others via backend |

### Fallback Language Examples

```javascript
// single fallback
i18next.init({ fallbackLng: 'en' });

// ordered fallbacks
i18next.init({ fallbackLng: ['fr', 'en'] });

// per-language fallbacks
i18next.init({
  fallbackLng: {
    'de-CH': ['fr', 'it'],
    'zh-Hant': ['zh-Hans', 'en'],
    'default': ['en']
  }
});

// function-based fallback
i18next.init({
  fallbackLng: (code) => {
    const fallbacks = [code];
    const langPart = code.split('-')[0];
    if (langPart !== code) fallbacks.push(langPart);
    fallbacks.push('dev');
    return fallbacks;
  }
});
```

---

## Missing Keys Configuration

Enable `saveMissing: true` to collect untranslated keys during development. Requires a backend plugin with create support.

| Option | Default | Description |
|--------|---------|-------------|
| `saveMissing` | `false` | Calls save missing key function on backend |
| `updateMissing` | `false` | Update default values via saveMissing |
| `saveMissingTo` | `'fallback'` | `'current'`, `'all'`, or `'fallback'` |
| `saveMissingPlurals` | `true` | Save all plural forms, not just singular |
| `missingKeyHandler` | `false` | `function(lngs, ns, key, fallbackValue, updateMissing, options) {}` |
| `parseMissingKeyHandler` | noop | `function(key, defaultValue, options) { return displayValue; }` |
| `appendNamespaceToMissingKey` | `false` | Appends namespace to missing key |
| `missingInterpolationHandler` | noop | `function(text, value) { return altValueOrUndefined; }` |
| `missingKeyNoValueFallbackToKey` | `false` | Don't fallback to key as default value |

---

## Translation Defaults

| Option | Default | Description |
|--------|---------|-------------|
| `postProcess` | `false` | String or array of postProcessors to apply |
| `returnNull` | `false` | Allow null as valid translation |
| `returnEmptyString` | `true` | Allow empty string as valid translation |
| `returnObjects` | `false` | Allow objects as valid translation result |
| `returnDetails` | `false` | Return object with language, namespace, key, value info |
| `joinArrays` | `false` | Char to join arrays (e.g. `", "`) |
| `interpolation` | `{...}` | See translation-functions.md for all interpolation options |
| `skipInterpolation` | `false` | Skip interpolation, return raw values |

---

## Plugin Options

| Option | Default | Description |
|--------|---------|-------------|
| `detection` | `undefined` | Language detector options |
| `backend` | `undefined` | Backend plugin options |
| `cache` | `undefined` | Cache layer options |

---

## Separator & Misc Options

| Option | Default | Description |
|--------|---------|-------------|
| `keySeparator` | `'.'` | Set to `false` for flat JSON |
| `nsSeparator` | `':'` | Splits namespace from key |
| `pluralSeparator` | `'_'` | Splits plural suffix from key |
| `contextSeparator` | `'_'` | Splits context from key |
| `ignoreJSONStructure` | `true` | Try flat key lookup if nested not found |
| `maxParallelReads` | `10` | Limits parallel backend reads to prevent EMFILE errors |
| `initAsync` | `true` | Resource loading via setTimeout. Set `false` only with sync backends like i18next-fs-backend. |
| `enableSelector` | `false` | TypeScript selector API. Set to `true` or `"optimize"` |
| `showSupportNotice` | `true` | Show i18next support notice |

### Using initImmediate (renamed initAsync) with sync backends

```javascript
import i18next from 'i18next';
import Backend from 'i18next-fs-backend';

i18next
  .use(Backend)
  .init({ initImmediate: false }); // legacy name, same as initAsync: false

i18next.t('key'); // works synchronously now
```

---

## Loading Translations

### On init (inline resources)

```javascript
i18next.init({
  resources: {
    en: {
      namespace1: { key: 'hello from namespace 1' },
      namespace2: { key: 'hello from namespace 2' }
    },
    de: {
      namespace1: { key: 'hallo von namespace 1' }
    }
  }
});
```

### After init (addResourceBundle)

```javascript
i18next.init({ resources: {} }); // pass empty to avoid backend warning
i18next.addResourceBundle('en', 'namespace1', { key: 'hello' });
```

`addResourceBundle(lng, ns, resources, deep, overwrite)`:
- `deep: true` extends existing translations
- `overwrite: true` overwrites existing keys (requires deep: true)

### With backend plugin (HTTP)

```javascript
import Backend from 'i18next-http-backend';

i18next
  .use(Backend)
  .init({
    backend: {
      loadPath: '/locales/{{lng}}/{{ns}}.json'
    }
  });
```

### Lazy loading with dynamic imports

```javascript
import resourcesToBackend from 'i18next-resources-to-backend';

i18next
  .use(resourcesToBackend((language, namespace) =>
    import(`./locales/${language}/${namespace}.json`)
  ))
  .init({ fallbackLng: 'en' });
```

### Combined inline + backend (partialBundledLanguages)

```javascript
i18next.init({
  partialBundledLanguages: true,
  ns: [], // don't load default namespace
  resources: {}
});
i18next.addResourceBundle('en', 'common', { save: 'Save' });
// Other namespaces load via backend
```

---

## Fallback Strategies

### Language Fallback (variant resolution)

By default, if `en-GB` doesn't have a key, i18next looks in `en`. Common strategy: put shared text in the base language, only specify differences in variants.

```javascript
i18next.init({
  lng: 'en-GB',
  resources: {
    'en-GB': { translation: { i18n: 'Internationalisation' } },
    'en': { translation: { i18n: 'Internationalization', i18n_short: 'i18n' } }
  }
});
i18next.t('i18n');       // -> "Internationalisation" (from en-GB)
i18next.t('i18n_short'); // -> "i18n" (falls back to en)
```

### Namespace Fallback

```javascript
i18next.init({
  ns: ['app', 'common'],
  defaultNS: 'app',
  fallbackNS: 'common' // look here if not found in app
});
i18next.t('title');       // looks in app namespace
i18next.t('button.save'); // found in common (fallback)
```

### Key Fallback (natural language keys)

```javascript
i18next.init({
  lng: 'de',
  nsSeparator: false,
  keySeparator: false,
  fallbackLng: false
});
// key IS the fallback English text
i18next.t('No results found.');
// -> German translation, or "No results found." if missing
```

### Multiple Fallback Keys

```javascript
const error = '404';
i18next.t([`error.${error}`, 'error.unspecific']);
// tries error.404 first, falls back to error.unspecific
```

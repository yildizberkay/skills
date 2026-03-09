# i18next Translation Functions Reference

## Table of Contents
1. [The t() Function](#the-t-function)
2. [Interpolation](#interpolation)
3. [Plurals](#plurals)
4. [Context](#context)
5. [Formatting (Built-in, v21.3.0+)](#formatting)
6. [Nesting](#nesting)
7. [Objects and Arrays](#objects-and-arrays)
8. [JSON Format v4](#json-format-v4)

---

## The t() Function

`i18next.t(key, options)` — the primary translation accessor.

### Key access patterns

```javascript
// Simple key
i18next.t('key'); // -> "value of key"

// Nested key (uses keySeparator '.')
i18next.t('look.deep'); // -> "value of look deep"

// Default value
i18next.t('missingKey', 'fallback text');
// or: i18next.t('missingKey', { defaultValue: 'fallback text' });

// Different namespace
i18next.t('button.save', { ns: 'common' });
// or (not recommended with natural language keys):
i18next.t('common:button.save');

// Multiple fallback keys
i18next.t([`error.${code}`, 'error.unspecific']);

// Override language for single call
i18next.t('key', { lng: 'de' });
```

### t() options

| Option | Description |
|--------|-------------|
| `defaultValue` | Fallback if key not found. Can define plural defaults with `defaultValue_other` |
| `count` | Triggers plural resolution |
| `context` | Triggers context resolution |
| `replace` | Object with interpolation vars (or put directly in options) |
| `lng` | Override language |
| `lngs` | Override languages array |
| `fallbackLng` | Override fallback (pass `false` to disable) |
| `ns` | Override namespace(s) |
| `keySeparator` | Override key separator |
| `nsSeparator` | Override namespace separator |
| `returnObjects` | Return object/array instead of string |
| `returnDetails` | Return `{ lng, ns, key, value }` object |
| `joinArrays` | Join array values with given char |
| `postProcess` | Apply post-processor(s) |
| `interpolation` | Override interpolation options |
| `skipInterpolation` | Skip interpolation and nesting |

### getFixedT

Returns a `t` function with fixed language, namespace, and/or key prefix:

```javascript
// Fix language
const de = i18next.getFixedT('de');
de('myKey');

// Fix namespace
const common = i18next.getFixedT(null, 'common');
common('button.save');

// Fix key prefix
const t = i18next.getFixedT(null, null, 'user.settings');
t('title'); // same as i18next.t('user.settings.title')
```

---

## Interpolation

Dynamic values inside translations using `{{variable}}` syntax.

### Basic interpolation

```json
{ "key": "{{what}} is {{how}}" }
```
```javascript
i18next.t('key', { what: 'i18next', how: 'great' });
// -> "i18next is great"
```

### Data model access

```json
{ "key": "I am {{author.name}}" }
```
```javascript
i18next.t('key', { author: { name: 'Jan', github: 'jamuhl' } });
// -> "I am Jan"
```

### Escaping / Unescaping

Values are XSS-escaped by default. Use `{{- variable}}` prefix for unescaped output:

```json
{
  "escaped": "safe: {{myVar}}",
  "unescaped": "raw: {{- myVar}}"
}
```
```javascript
i18next.t('escaped', { myVar: '<b>bold</b>' });
// -> "safe: &lt;b&gt;bold&lt;/b&gt;"

i18next.t('unescaped', { myVar: '<b>bold</b>' });
// -> "raw: <b>bold</b>"
```

You can also disable escaping per-call: `{ interpolation: { escapeValue: false } }`.

Note: React already escapes, so react-i18next sets `escapeValue: false` by default.

### All interpolation options

| Option | Default | Description |
|--------|---------|-------------|
| `format` | noop | Legacy format function (pre v21.3.0) |
| `formatSeparator` | `','` | Separates format name from value |
| `escape` | function | Custom escape function |
| `escapeValue` | `true` | Enable XSS escaping |
| `useRawValueToEscape` | `false` | Don't cast to string before escaping |
| `prefix` | `'{{'` | Interpolation prefix |
| `suffix` | `'}}'` | Interpolation suffix |
| `unescapePrefix` | `'-'` | Prefix for unescaped mode |
| `nestingPrefix` | `'$t('` | Nesting prefix |
| `nestingSuffix` | `')'` | Nesting suffix |
| `defaultVariables` | `undefined` | Global default interpolation vars |
| `maxReplaces` | `1000` | Max interpolation iterations |
| `skipOnVariables` | `true` (v21+) | Don't resolve `$t()` or `{{}}` inside variable values. Keep `true` when variables are user-provided. |

---

## Plurals

Uses the Intl.PluralRules API. The `count` variable is **required** — without it, plural resolution won't trigger.

### JSON v4 plural suffixes

English (2 forms): `_one`, `_other`
Arabic (6 forms): `_zero`, `_one`, `_two`, `_few`, `_many`, `_other`

The `_zero` suffix is a special i18next feature: if `count` is 0 and a `_zero` key exists, it's used instead of the language's plural rule.

### Basic plurals

```json
{
  "key_one": "item",
  "key_other": "items",
  "keyWithCount_one": "{{count}} item",
  "keyWithCount_other": "{{count}} items"
}
```
```javascript
i18next.t('key', { count: 0 });   // -> "items"
i18next.t('key', { count: 1 });   // -> "item"
i18next.t('key', { count: 5 });   // -> "items"
i18next.t('keyWithCount', { count: 1 });  // -> "1 item"
i18next.t('keyWithCount', { count: 5 });  // -> "5 items"
```

### Ordinal plurals

Use `ordinal: true` to get ordinal forms (`1st`, `2nd`, `3rd`, `4th`):

```json
{
  "key_ordinal_one": "{{count}}st place",
  "key_ordinal_two": "{{count}}nd place",
  "key_ordinal_few": "{{count}}rd place",
  "key_ordinal_other": "{{count}}th place"
}
```
```javascript
i18next.t('key', { count: 1, ordinal: true });  // -> "1st place"
i18next.t('key', { count: 2, ordinal: true });  // -> "2nd place"
i18next.t('key', { count: 11, ordinal: true }); // -> "11th place"
```

### Interval plurals (plugin)

Requires `i18next-intervalplural-postprocessor`:

```json
{
  "key_one": "{{count}} item",
  "key_other": "{{count}} items",
  "key_interval": "(1)[one item];(2-7)[a few items];(7-inf)[a lot of items];"
}
```
```javascript
i18next.t('key_interval', { postProcess: 'interval', count: 4 });
// -> "a few items"
```

### Multiple counts in one translation

Use nesting to handle multiple count variables:

```json
{
  "girlsAndBoys": "They have $t(girls, {\"count\": {{girls}} }) and $t(boys, {\"count\": {{boys}} })",
  "boys_one": "{{count}} boy",
  "boys_other": "{{count}} boys",
  "girls_one": "{{count}} girl",
  "girls_other": "{{count}} girls"
}
```
```javascript
i18next.t('girlsAndBoys', { girls: 3, boys: 2 });
// -> "They have 3 girls and 2 boys"
```

### Polyfill requirement

If `Intl.PluralRules` is unavailable, i18next falls back to v3 format. To use v4 format everywhere:

```bash
npm install intl-pluralrules
```
```javascript
import 'intl-pluralrules';
```

---

## Context

Add `context` option to select variant translations. Key suffix format: `key_contextValue`.

```json
{
  "friend": "A friend",
  "friend_male": "A boyfriend",
  "friend_female": "A girlfriend"
}
```
```javascript
i18next.t('friend');                        // -> "A friend"
i18next.t('friend', { context: 'male' });   // -> "A boyfriend"
i18next.t('friend', { context: 'female' }); // -> "A girlfriend"
```

### Context + Plurals combined

Key format: `key_context_pluralSuffix`

```json
{
  "friend_male_one": "A boyfriend",
  "friend_female_one": "A girlfriend",
  "friend_male_other": "{{count}} boyfriends",
  "friend_female_other": "{{count}} girlfriends"
}
```
```javascript
i18next.t('friend', { context: 'male', count: 1 });   // -> "A boyfriend"
i18next.t('friend', { context: 'female', count: 100 }); // -> "100 girlfriends"
```

---

## Formatting

Since v21.3.0, i18next has built-in formatting based on the Intl API.

### Syntax

```
{{value, formatName}}
{{value, formatName(option1: value1; option2: value2)}}
```

Format options can be passed three ways:
1. Inline in the translation string: `{{val, number(minimumFractionDigits: 2)}}`
2. Root level in t() options: `{ minimumFractionDigits: 2 }`
3. Per-value with formatParams: `{ formatParams: { val: { minimumFractionDigits: 2 } } }`

### Number

```json
{ "intlNumber": "Some {{val, number}}" }
```
```javascript
i18next.t('intlNumber', { val: 1000 });   // -> "Some 1,000"
i18next.t('intlNumber', { val: 1000.1, minimumFractionDigits: 3 });
// -> "Some 1,000.100"
```

### Currency

```json
{
  "price": "The value is {{val, currency(USD)}}",
  "priceExplicit": "The value is {{val, currency(currency: USD)}}"
}
```
```javascript
i18next.t('price', { val: 2000 }); // -> "The value is $2,000.00"
```

Multiple currencies with different locales:
```javascript
i18next.t('key', {
  localValue: 12345.67,
  altValue: 16543.21,
  formatParams: {
    localValue: { currency: 'USD', locale: 'en-US' },
    altValue: { currency: 'CAD', locale: 'fr-CA' }
  }
});
```

### DateTime

```json
{ "intlDateTime": "On the {{val, datetime}}" }
```
```javascript
i18next.t('intlDateTime', {
  val: new Date(Date.UTC(2012, 11, 20, 3, 0, 0)),
  formatParams: {
    val: { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' }
  }
});
// -> "On the Thursday, December 20, 2012"
```

### RelativeTime

```json
{
  "relative": "Lorem {{val, relativetime}}",
  "relativeQuarter": "Lorem {{val, relativetime(quarter)}}",
  "relativeNarrow": "Lorem {{val, relativetime(range: quarter; style: narrow;)}}"
}
```
```javascript
i18next.t('relative', { val: 3 });         // -> "Lorem in 3 days"
i18next.t('relativeQuarter', { val: -3 }); // -> "Lorem 3 quarters ago"
```

### List

```json
{ "intlList": "A list of {{val, list}}" }
```
```javascript
i18next.t('intlList', { val: ['locize', 'i18next', 'awesomeness'] });
// -> "A list of locize, i18next, and awesomeness"
```

### Custom formatters

```javascript
// Add after i18next.init()
i18next.services.formatter.add('lowercase', (value, lng, options) => {
  return value.toLowerCase();
});

// Cached version (better performance)
i18next.services.formatter.addCached('specialformat', (lng, options) => {
  const formatter = new Intl.NumberFormat(lng, options);
  return (val) => formatter.format(val);
});
```

### Multiple formatters

Chain formatters in translation strings:
```json
{ "key": "Some format {{value, formatter1, formatter2}}" }
```

### Override language for formatting

```javascript
i18next.t('intlNumber', { val: 1000.1, lng: 'de' });
// or per-value:
i18next.t('intlNumber', { val: 1000.1, formatParams: { val: { lng: 'de' } } });
```

---

## Nesting

Reference other translation keys using `$t(key)` syntax:

```json
{
  "nesting1": "1 $t(nesting2)",
  "nesting2": "2 $t(nesting3)",
  "nesting3": "3"
}
```
```javascript
i18next.t('nesting1'); // -> "1 2 3"
```

Cross-namespace nesting: `"key": "1 $t(common:otherKey)"`

### Passing options to nested translations

Options must be valid JSON inside the `$t()` call:

```json
{
  "key": "They have $t(girls, {\"count\": {{girls}} }) and $t(boys, {\"count\": {{boys}} })",
  "boys_one": "{{count}} boy",
  "boys_other": "{{count}} boys",
  "girls_one": "{{count}} girl",
  "girls_other": "{{count}} girls"
}
```

### Nesting in interpolation values

```json
{ "key1": "hello world", "key2": "say: {{val}}" }
```
```javascript
i18next.t('key2', { val: '$t(key1)' });
// -> "say: hello world"
// Note: requires skipOnVariables: false (since v21)
```

---

## Objects and Arrays

### Return objects

```json
{ "tree": { "res": "added {{something}}" } }
```
```javascript
i18next.t('tree', { returnObjects: true, something: 'gold' });
// -> { res: 'added gold' }
```

### Return and join arrays

```json
{
  "lines": ["line1", "line2", "line3"],
  "dynamic": ["you", "can", "{{myVar}}"]
}
```
```javascript
i18next.t('lines', { joinArrays: '+' });
// -> "line1+line2+line3"

i18next.t('dynamic', { myVar: 'interpolate', joinArrays: ' ' });
// -> "you can interpolate"
```

### Access array items

```json
{ "items": [{ "name": "tom" }, { "name": "steve" }] }
```
```javascript
i18next.t('items.0.name'); // -> "tom"
```

---

## JSON Format v4

The default format since i18next v21. Key differences from v3:

| Feature | v3 suffix | v4 suffix |
|---------|-----------|-----------|
| Plural (English) | `key` / `key_plural` | `key_one` / `key_other` |
| Plural (Arabic) | `key_0` through `key_5` | `key_zero` through `key_other` |

Complete v4 format example:

```json
{
  "key": "value",
  "keyDeep": { "inner": "value" },
  "keyNesting": "reuse $t(keyDeep.inner)",
  "keyInterpolate": "replace this {{value}}",
  "keyInterpolateUnescaped": "replace this {{- value}}",
  "keyInterpolateWithFormatting": "replace this {{value, format}}",
  "keyContext_male": "the male variant",
  "keyContext_female": "the female variant",
  "keyPluralSimple_one": "the singular",
  "keyPluralSimple_other": "the plural",
  "keyPluralMultipleEgArabic_zero": "the plural form 0",
  "keyPluralMultipleEgArabic_one": "the plural form 1",
  "keyPluralMultipleEgArabic_two": "the plural form 2",
  "keyPluralMultipleEgArabic_few": "the plural form 3",
  "keyPluralMultipleEgArabic_many": "the plural form 4",
  "keyPluralMultipleEgArabic_other": "the plural form 5",
  "keyWithArrayValue": ["multiple", "things"],
  "keyWithObjectValue": { "valueA": "return this with valueB", "valueB": "more text" }
}
```

To enable old v3 format: `i18next.init({ compatibilityJSON: 'v3' })`.
To migrate: use [i18next-v4-format-converter](https://github.com/i18next/i18next-v4-format-converter).

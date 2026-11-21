---
name: i18n-localization
description: Internationalize and localize apps — extract every user-facing string, write ICU MessageFormat messages with correct CLDR plurals, format dates/numbers/currency with Intl, support RTL with logical CSS, and verify with pseudo-localization. Use when adding a language, building locale routing, fixing plural/gender/translation bugs, auditing hardcoded strings, or shipping to multiple locales.
---

# i18n-localization

Makes software language- and locale-ready. It separates **i18n** (build the app so it *can* be localized — once per feature) from **l10n** (adapt it for a specific locale — once per language). Most bugs come from bolting translations on at the end, which yields hardcoded strings, broken plurals ("1 messages"), mangled dates, and RTL layouts that fall apart. This skill encodes the rules that prevent those bugs before they ship.

It is framework-agnostic: the same rules apply to React, Next.js, Vue, Svelte, Node, Expo/React Native, and plain web apps.

## When to Use This Skill

- Adding a new language or locale to an app
- Fixing plural, gender, or translation bugs (wrong form for Arabic/Russian/Polish/Japanese)
- Auditing a codebase for hardcoded or untranslatable strings
- Building locale routing, language pickers, or `hreflang` metadata
- Making a layout RTL-ready for Arabic, Hebrew, Persian, or Urdu
- Reviewing a PR that changes user-facing text

## What This Skill Does

1. **String extraction**: every label, message, error, tooltip, aria-label, and placeholder goes into a message catalog. Never concatenate or split sentences — word order differs by language; use named placeholders like `t('greeting', { name, count })`. Keys are semantic, feature-grouped, and linted, with a `common.*` namespace for reusable strings.
2. **ICU MessageFormat**: writes complete translatable strings with interpolation, `plural`, `select`, `selectordinal`, nested arguments, `#`, `=0` overrides, and `offset:`. Uses CLDR categories (`zero | one | two | few | many | other`) selected per locale — never `if (count === 1)`. English has 2 plural forms, Arabic 6, Japanese 1. Always includes `other`; passes `count` as a number.
3. **Intl-aware formatting**: dates, numbers, currency, percentages, lists, relative time, and sorting use `Intl.DateTimeFormat`, `Intl.NumberFormat`, `Intl.RelativeTimeFormat`, `Intl.ListFormat`, and `Intl.Collator` with an explicit locale. Dates are stored UTC, money as `{ amount, currency }`, and both are formatted only at display time. No `toLocaleDateString()` without a locale argument.
4. **Locale negotiation**: explicit user choice → URL/route prefix → `Accept-Language` → default. Uses BCP 47 tags (`en-US`, `zh-CN`, `ar-EG`) and sets `<html lang>` and `<html dir>` from the resolved locale.
5. **RTL and layout**: uses CSS logical properties (`margin-inline-start`, `padding-inline-end`, `text-align: start`, `border-inline-start`) so layout flips automatically. Mirrors only directional icons, never logos/text/digits, and isolates mixed-direction content with `<bdi>`. Designs for ~30% string growth.
6. **Verification**: pseudo-localization to expose hardcoded strings and truncation, missing-key CI checks, plural edge-case tests (0, 1, 2, 5, 11, 21, 100), RTL screenshots, and fallback checks.

## How to Use

### Basic Usage

```
Audit the settings page for i18n issues and list every hardcoded user-facing string that needs a translation key.
```

### Advanced Usage

```
Internationalize the notifications feature for en-US, de-DE, and ar-EG. Extract all strings into feature-grouped ICU catalogs, add the correct CLDR plural categories for each locale, replace manual date/number formatting with Intl, make the layout RTL-safe with logical CSS, and add pseudo-localization plus missing-key checks to the test suite.
```

## Example

**User**: "Fix the inbox count — it says '1 messages' in English and reads wrong in Russian and Arabic."

**Output**:

A single ICU key replacing the `if (count === 1)` logic, with all required CLDR categories per locale:

```jsonc
{
  "inbox.message_count": "{count, plural, =0 {No messages} one {# message} other {# messages}}"
}
```

For Arabic the same key declares all six categories (`zero`, `one`, `two`, `few`, `many`, `other`); for Russian it uses `one`, `few`, `many`, `other`. `count` is passed as a number, `#` is replaced by the locale-formatted value, and `=0` provides the structural zero state. Plus locale-aware formatting:

```js
new Intl.NumberFormat('de-DE', { style: 'currency', currency: 'EUR' }).format(99.99); // 99,99 €
new Intl.DateTimeFormat('ja-JP').format(new Date());                                  // 2026/12/31
```

## Tips

- Never use double curly braces `{{name}}` — that is a parser error in FormatJS/react-intl; ICU uses single braces.
- Always include the `other` plural category; it is mandatory even in languages with one form.
- Give translators context: a description per key and the meaning of each placeholder. Keep the whole sentence, punctuation included, in one key.
- Do not special-case `count === 0` globally — zero is `other` in English but `zero` in Arabic.
- Use `Intl.Collator` for user-visible sorting; sorting by code point misplaces accented letters.
- Wrap user-generated RTL content in `<bdi>` so it cannot reorder the surrounding sentence.
- Format values only at display time; keep raw dates in UTC and money as amount + currency.
- Turn on library debug mode (e.g. `debug: true` in i18next) to trace which key resolved.

## Common Use Cases

- Internationalizing a React/Next.js app with next-intl, react-intl/FormatJS, or i18next + i18next-icu
- Adding a second language to a product and preventing regressions with pseudo-localization
- Auditing an existing codebase for hardcoded strings and manual date/number formatting
- Preparing an Arabic or Hebrew UI with logical CSS and mirrored directional icons
- Fixing locale routing, `<html lang/dir>`, and `hreflang` alternates for SEO

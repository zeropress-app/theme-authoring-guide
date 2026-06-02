# ZeroPress Theme Variable Documentation Guideline

This document defines the recommended way for ZeroPress theme authors to document CSS variable overrides.

The current ZeroPress product direction is:

- no built-in theme customize UI
- theme-specific styling guidance is provided through theme documentation
- users apply overrides through `Design > Custom CSS`

This keeps the runtime contract simple while still giving theme authors a clear way to expose supported design controls.

## Purpose

Theme authors should document:

- which CSS variables are intended for user override
- what each variable controls
- what value format is expected
- what the default value is
- what compatibility promises are made across theme updates

Theme authors should not expect users to inspect internal CSS implementation details.

The documentation page is the canonical public contract for theme-level design customization.

## Where To Publish

Each published theme should provide a documentation page in the theme directory.

Example:

```text
https://themes.zeropress.org/themes/{namespace}/{slug}
```

The documentation should be visible before installation when possible, and should remain available after installation as the long-term reference for the theme.

## Product Model

The intended user flow is:

1. Install or activate a theme.
2. Read the theme's documentation page.
3. Copy supported variable names and examples.
4. Apply overrides in `Design > Custom CSS`.

This means the theme directory documentation acts as the configuration guide, while `Custom CSS` remains the editing surface.

## Authoring Rules

Theme authors should follow these rules when exposing customizable variables.

- Expose user-facing design settings through CSS variables.
- Keep variable names stable once published.
- Use variables for colors, spacing, typography, borders, and layout tuning.
- Avoid documenting internal-only variables that may change frequently.
- Keep override points small and intentional instead of exposing every internal token.

Common variable groups:

- color variables
- font variables
- spacing variables
- layout variables
- border variables

## Required Documentation Sections

Each theme variable documentation page should contain the following sections.

## Summary

Briefly explain what kinds of appearance changes the theme supports.

Example:

```text
This theme supports CSS variable overrides for colors, typography, content width, and card styling.
Apply overrides in Design > Custom CSS.
```

## Usage

Tell the user where overrides should be placed.

Example:

```css
:root {
  --color-bg: #f7f3ec;
  --color-text: #2c241f;
}
```

If the theme requires a narrower selector than `:root`, that must be documented explicitly.

## Variable Reference

List each supported variable with:

- variable name
- purpose
- accepted value format
- default value
- notes

Recommended table format:

| Variable | Purpose | Expected Value | Default | Notes |
| --- | --- | --- | --- | --- |
| `--color-bg` | Page background color | CSS color | `#f7f3ec` | Applied to site background |
| `--color-text` | Main body text color | CSS color | `#2c241f` | Should maintain readable contrast |
| `--font-body` | Body text font family | CSS `font-family` value | `"Iowan Old Style", Georgia, serif` | Fallback stack should be preserved |
| `--layout-content-width` | Main content width | CSS length | `44rem` | Affects reading width |

## Examples

Provide a few copy-paste examples.

Recommended examples:

- darker text / lighter paper background
- accent color change
- wider or narrower reading width
- font stack replacement

Example:

```css
:root {
  --color-bg: #fcfaf6;
  --color-text: #2d241d;
  --color-accent: #8b5a3c;
  --layout-content-width: 46rem;
}
```

## Compatibility Notes

Explain what is stable and what may change.

Theme authors should state clearly:

- which variable names are considered public and stable
- whether new variables may be added in minor releases
- whether deprecated variables will be removed immediately or with notice

Recommended wording:

```text
Variables listed on this page are public theme customization APIs.
New variables may be added in future releases.
Listed variables may be changed or removed only in a breaking theme release.
Undocumented variables are internal and should not be relied on.
```

## Recommended Writing Style

- Use exact variable names.
- Use copy-paste-ready code blocks.
- Prefer concrete statements over vague design language.
- Say "supported override" only when the theme author intends to preserve it.
- Distinguish public variables from internal implementation variables.

## Example Documentation Snippet

````md
## Theme variables

Apply overrides in `Design > Custom CSS`.

| Variable | Purpose | Expected Value | Default |
| --- | --- | --- | --- |
| `--color-bg` | Site background color | CSS color | `#f7f3ec` |
| `--color-text` | Body text color | CSS color | `#2c241f` |
| `--font-body` | Body font stack | CSS font-family | `"Iowan Old Style", Georgia, serif` |

## Example

```css
:root {
  --color-bg: #fffdf8;
  --color-text: #2a211b;
  --font-body: "Cormorant Garamond", Georgia, serif;
}
```
````

## Non-Goals

This guideline does not define:

- a dedicated admin customize UI
- automatic extraction of variables from CSS
- guaranteed support for every variable found in theme source
- build-time validation of theme documentation content

Those concerns should be handled separately if ZeroPress introduces a stronger theme customization contract later.

## Recommendation

For the current ZeroPress product stage, theme documentation plus `Custom CSS` is the preferred customization model.

Theme authors should treat documented CSS variables as a lightweight public API for visual customization.

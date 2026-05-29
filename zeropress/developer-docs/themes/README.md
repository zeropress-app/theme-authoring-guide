# ZeroPress Theme Docs

This folder contains the current official guidance for ZeroPress theme authors.

Current runtime baseline:

- `runtime: "0.6"` only
- structured route data for post lists, archive groups, pagination, widgets, and post surroundings
- build-provided TOC data for Markdown content, with theme-owned TOC UI
- theme-facing footer display data through `site.footer.copyright_text` and `site.footer.attribution`
- theme-owned progressive enhancement for optional UI such as comments, search, and non-Markdown TOC behavior
- runtime ergonomics additions such as `loop.*`, `pagination.window[]`, partial arguments, and `else_if` chains

Recommended reading order:

1. [AI Theme Generation Brief v0.6](./ai-theme-generation-brief-v0.6.md)
2. [Theme Authoring Guide v0.6](./theme-authoring-guide-v0.6.md)
3. [Theme Runtime Template Syntax v0.6](./theme-runtime-template-syntax-v0.6.md)
4. [Theme Widget Runtime Contract](./theme-widget-runtime-contract.md)
5. [Theme Package Naming Rules](./packaging/theme-package-naming-rules.md)
6. [Theme Variable Documentation Guideline](./packaging/theme-variable-documentation-guideline.md)

Reference implementation:

- `user_facing_site/zeropress-pages/theme-blog`

Practical notes:

- `zeropress-pages/theme-blog` is a historical dogfooding theme for earlier runtime work.
- `zeropress-pages/theme` has been retired.
- `runtime: "0.5"` and older are no longer supported by the current `@zeropress/theme-validator` or `@zeropress/build-core`.

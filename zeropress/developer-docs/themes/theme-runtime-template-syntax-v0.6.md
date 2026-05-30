# ZeroPress Theme Runtime Template Syntax v0.6

This document describes the current template syntax available to ZeroPress themes running on `runtime: "0.6"`.

It is the current runtime contract for:

- `@zeropress/theme-validator`
- `@zeropress/build-core`
- official dogfooding themes such as `theme-blog`

## Supported Syntax

### Variable Interpolation

```html
{{site.title}}
{{post.title}}
{{pagination.current_page}}
{{menus.docs-sidebar.items}}
{{collections.cover-story.items}}
{{site.meta.issue}}
```

Default rule:

- `{{path}}` is escaped output

Path rules:

- `.` separates path segments.
- each segment must start with a letter or underscore
- after the first character, each segment may contain letters, digits, underscores, and internal hyphens
- hyphens cannot start or end a segment, and consecutive hyphens are invalid
- paths are data lookups only; arithmetic and other expressions are not supported

Valid examples:

```html
{{menus.docs-sidebar.items}}
{{widgets.right-sidebar.items}}
{{collections.cover-story.items}}
{{post.custom-field}}
```

Invalid examples:

```html
{{123.bad}}
{{menus.-bad.items}}
{{menus.bad-.items}}
{{menus.bad--key.items}}
```

## `slot`

Slots are reserved for layout composition.

Required layout slot:

- `{{slot:content}}`

`layout.html` must contain exactly one `{{slot:content}}`.

Allowed non-content slots are resolved from same-named partials:

- `{{slot:header}}` resolves to `partials/header.html` when present
- `{{slot:footer}}` resolves to `partials/footer.html` when present
- `{{slot:meta}}` resolves to `partials/meta.html` when present

If a named non-content slot has no matching partial, it renders as an empty string.

Use `{{meta.head_tags}}` for build-generated head tags. Do not use `{{slot:meta}}` for canonical/meta/OpenGraph tags unless the theme intentionally provides `partials/meta.html`.

Only `content`, `header`, `footer`, and `meta` are valid slot names in `runtime: "0.6"`. `slot` is for layout composition. Use `partial` for explicit reusable fragments.

## `partial`

Partials are reusable markup fragments.

```html
{{partial:sidebar-widgets}}
{{partial:shared/post-card}}
{{partial:post-list-item variant="compact" show_excerpt=true}}
{{partial:project-card project=post show_meta=true limit=3 fallback=null}}
```

Resolution rules:

- `{{partial:name}}` resolves to `partials/name.html`
- current render context is shared
- partial arguments are optional and exposed as `partial.*` inside the partial body
- partial argument values support double-quoted strings, typed literals, and path aliases
- nested partials are supported
- missing partials fail validation
- circular partial references fail validation

Partial arguments are not required to pass data into a partial. The parent render context is already shared, so a partial can read `post`, `page`, `item`, `site`, `collections`, and other in-scope paths directly. Arguments are mainly useful when a reusable partial wants a stable component-like API:

```html
{{#for item in collections.work.items}}
  {{partial:project-card project=item variant="featured"}}
{{/for}}

{{#for post in posts.items}}
  {{partial:project-card project=post variant="compact"}}
{{/for}}
```

Inside `partials/project-card.html`, both callers can use `partial.project`:

```html
<article class="project-card project-card--{{partial.variant}}">
  <a href="{{partial.project.url}}">{{partial.project.title}}</a>
</article>
```

Argument value rules:

- `"compact"` is a string literal.
- `true`, `false`, `null`, `3`, and `1.5` are typed literals.
- `item`, `post.featured_media`, and `collections.work.items` are path aliases resolved from the current render context.
- a single-segment path alias such as `item` must be a known render root or an active `for` loop alias; dotted paths such as `post.featured_media` are resolved at render time.
- missing path aliases are passed as empty/falsey values, so guard them with `{{#if partial.project}}`.
- unquoted values are never string literals. Write `variant="compact"`, not `variant=compact`, when you mean text.
- bracket access, operators, arrays, objects, and expressions are not supported in partial arguments.

Recommended use:

- repeated sidebar widget blocks
- repeated post card markup
- repeated archive intro or footer sections
- analytics and tracker integration points such as `{{partial:tracker}}`
- content enhancement loaders such as `{{partial:content-enhancements}}`
- third-party loader markup for site-owned `public/` assets

Partials are also the recommended way to keep `layout.html` strict while still allowing named site integrations:

```html
<head>
  {{partial:tracker}}
</head>
<body>
  {{slot:content}}
  {{partial:content-enhancements}}
</body>
```

For page/post-only integrations, use `else_if` because `v0.6` has no logical `or` operator:

```html
{{#if post}}
  {{partial:mermaid-loader}}
{{#else_if page}}
  {{partial:mermaid-loader}}
{{/if}}
```

## `if`

Render a block only when the referenced value is truthy.

```html
{{#if post.featured_image}}
  <img src="{{post.featured_image}}" alt="">
{{/if}}
{{#if post.featured_image}}
  <img src="{{post.featured_image}}" alt="">
{{#else_if post.fallback_image}}
  <img src="{{post.fallback_image}}" alt="">
{{/if}}
```

With fallback:

```html
{{#if widget.title}}
  <h2>{{widget.title}}</h2>
{{#else}}
  <h2>Untitled</h2>
{{/if}}
```

Rules:

- operand must be a single variable path
- expressions are not supported

Invalid:

```html
{{#if post.featured_image && post.title}}
{{#if widget.items.length > 0}}
```

## `if_eq`, `if_neq`, `if_in`, and `if_starts_with`

Render a block when a path matches a typed comparison condition.

```html
{{#if_eq widget.type "profile"}}
  ...
{{/if}}

{{#if_eq loop.index 4}}
  ...
{{/if}}

{{#if_eq route.url item.url}}
  aria-current="page"
{{/if}}

{{#if_neq loop.last true}}, {{/if}}

{{#if_in route.type "post" "page" "front_page"}}
  {{partial:content-enhancements}}
{{/if}}

{{#if_starts_with route.url item.url}}
  is-active
{{/if}}
```

With fallback:

```html
{{#if_eq widget.type "profile"}}
  ...
{{#else}}
  ...
{{/if}}
{{#if_eq route.url item.url}}
  is-active
{{#else_if_starts_with route.url item.url}}
  is-parent
{{/if}}

{{#if_in route.type "tag"}}
  Tag
{{#else_if_eq route.type "post"}}
  Content
{{#else}}
  Other
{{/if}}
```

Rules:

- left-hand side must be a single variable path
- `if_eq`, `if_neq`, and `if_starts_with` require exactly one right-hand operand
- `if_in` requires one or more candidate operands
- right-hand operands may be string, number, boolean, or `null` literals, or variable paths
- comparisons are strict and do not coerce types
- `if_starts_with` matches only when both resolved values are strings
- comparison helper branches may be mixed inside one conditional block
- close comparison helper blocks with `{{/if}}`
- concrete close tags such as `{{/if_eq}}` are accepted in v0.6 for compatibility, but are planned for removal in v0.7

Typed literals:

```html
{{#if_eq loop.index 4}}
{{#if_eq site.footer.attribution true}}
{{#if_eq post.meta.featured null}}
```

Invalid:

```html
{{#if_eq site.footer.attribution}}
{{#if_in route.type}}
{{#if_eq route.type post page}}
```

Use `if` for truthiness checks:

```html
{{#if site.footer.attribution}}
  ...
{{/if}}
```

Runtime v0.6 intentionally does not support `and`, `or`, `>`, `<`, modulo,
arithmetic, slicing, or type juggling in templates. Prefer build-prepared
data for complex layout decisions.

## `for`

Render a block for each item in an array.

```html
{{#for post in posts.items}}
  <article>
    <h2>{{post.title}}</h2>
  </article>
{{/for}}
```

Rules:

- right-hand side must be an array variable path
- loop variable must be a simple identifier
- nested loops are supported

Reserved loop metadata is available inside the current `for` body:

- `loop.first`
- `loop.last`
- `loop.index`: zero-based numeric index

Example:

```html
{{#for tag in post.tags}}
  <a href="{{tag.url}}">{{tag.name}}</a>{{#if_neq loop.last true}}<span>, </span>{{/if}}
{{/for}}
```

## `else`

`else` is supported only inside:

- `if`
- `if_eq`
- `if_neq`
- `if_in`
- `if_starts_with`

Rules:

- at most one `else` per block
- `else` must be at the same nesting level as its matching opener
- `for ... else` is not supported

Related branch-reduction tags are also supported:

- `{{#else_if path}}`
- `{{#else_if_eq path "literal"}}`
- `{{#else_if_neq path "literal"}}`
- `{{#else_if_in path "a" "b"}}`
- `{{#else_if_starts_with path other.path}}`

## Template Comments

Inline:

```html
{{! Short note }}
```

Block:

```html
{{!--
Longer note for theme authors.
--}}
```

Rules:

- comments are removed before render output
- comment bodies are not interpreted as template code

## Truthiness And Equality

Current evaluation behavior:

| Value | `{{#if path}}` | `{{#if_eq path ""}}` |
| --- | --- | --- |
| missing | false | false |
| `null` | false | false |
| `""` | false | true |
| `"profile"` | true | false |
| `false` | false | false |
| `0` | false | false |
| `1` | true | false |

This means:

- `{{#if widget.title}}` treats empty string as false
- `{{#if_eq widget.type ""}}` matches only exact empty string
- `{{#if loop.index}}` is false for the first item because `loop.index` is `0`; use `loop.first` for first-item checks
- `{{#if_eq loop.index "1"}}` is false because numbers are not coerced to strings

## Escaping Rules

Default interpolation is escaped:

- text nodes are escaped
- attribute values are escaped

Trusted raw HTML is allowed only through explicit build-prepared fields such as:

- `post.html`
- `page.html`
- `meta.head_tags`
- `widget.html`

Current convention:

- `html` or `_html` fields are trusted raw HTML
- `_url` fields are normalized URL values prepared by build tooling

Themes should not expect raw HTML from ordinary scalar fields such as:

- `post.title`
- `widget.display_name`
- `site.description`

## Special Placeholder Notes

The current runtime still distinguishes between:

- `slot` for layout composition
- `partial` for reusable fragments
- `menu` placeholders such as `{{menu:primary}}` for menu slot rendering

`{{partial:footer}}` is valid syntax, but it is conceptually different from `{{slot:footer}}`.

`{{menu:primary}}` renders generic menu markup and does not interpret menu item `meta`. Themes that need per-link icons, badges, or accent classes should manually iterate `menus.primary.items` and read values such as `item.meta.icon`.

## Unsupported Syntax

The following are not part of `v0.6`:

- `unless`
- custom helper functions
- arithmetic
- logical operators such as `&&`, `||`, `and`, and `or`
- comparison operators such as `>`, `<`, `>=`, and `<=`
- slicing or limit/offset expressions
- Vue-style directives such as `v-if` or `v-for`

The following old render-ready placeholders are no longer part of the supported theme contract:

- `{{posts}}`
- `{{pagination}}`

## Example

```html
<aside class="sidebar-stack">
  {{#if post.featured_image}}
    <img src="{{post.featured_image}}" alt="">
  {{/if}}

  {{partial:sidebar-widgets}}

  {{#if post.comments_enabled}}
    <div class="comments-shell" data-zp-comments data-zp-comments-post="{{post.public_id}}" hidden></div>
  {{/if}}
</aside>
```

Post index pagination should use the structured pagination object and the current route metadata:

```html
{{#if route.is_post_index}}
  {{#if pagination.enabled}}
    <nav aria-label="Pagination">
      {{#for page in pagination.window}}
        <a href="{{page.url}}">{{page.number}}</a>
      {{/for}}
    </nav>
  {{/if}}
{{/if}}
```

Only documented route flags are available. There is no `route.is_post` shortcut;
for post-specific branching outside `post.html`, compare `route.type`:

```html
{{#if_eq route.type "post"}}
  ...
{{/if}}
```

Page templates can also branch on front-page renders. This is useful when the front page is backed by Markdown and the Markdown body already contains the visible H1:

```html
{{#if route.is_front_page}}
  <div class="prose">{{page.html}}</div>
{{#else}}
  <header>
    <h1>{{page.title}}</h1>
  </header>
  <div class="prose">{{page.html}}</div>
{{/if}}
```

Footer UI should use site-provided footer display data when available:

```html
<footer>
  {{#if site.footer.copyright_text}}
    <p>{{site.footer.copyright_text}}</p>
  {{#else_if site.title}}
    <p>{{site.title}}</p>
  {{/if}}

  {{#if site.footer.attribution}}
    <p>Published with <a href="https://zeropress.app" target="_blank" rel="noreferrer noopener">ZeroPress</a>.</p>
  {{/if}}
</footer>
```

`site.footer.attribution` is normalized by build as truthy unless preview-data explicitly sets it to `false`. ZeroPress does not add a copyright symbol to `site.footer.copyright_text`.

## Recommendation

For `v0.6` themes:

- treat structured data as the main contract
- keep logic declarative and small
- use partials to avoid repeated markup
- reserve theme JS for progressive enhancement, not core document rendering

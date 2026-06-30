# ZeroPress AI Theme Generation Brief v0.6

Use this brief when generating a new ZeroPress theme from scratch. It is intentionally short and implementation-oriented; use the full authoring guide for deeper reference.

## Goal

Generate a reusable ZeroPress theme that targets:

```json
{
  "runtime": "0.6"
}
```

Do not generate compatibility code for older runtime versions.

## Required Theme Files

The theme must include:

```txt
theme/
  theme.json
  layout.html
  index.html
  post.html
  page.html
  assets/
    style.css
```

Recommended optional files:

```txt
theme/
  archive.html
  category.html
  tag.html
  404.html
  partials/
    header.html
    footer.html
    post-card.html
    page-toc.html
  assets/
    theme.js
```

`layout.html` must contain exactly one `{{slot:content}}`.

`layout.html` must not contain direct `<script>` tags. Put script integrations in named partials such as `partials/tracker.html` or `partials/content-enhancements.html`, then include those partials from `layout.html`.

Only these slot names are valid in `runtime: "0.6"`:

- `{{slot:content}}`
- `{{slot:header}}`
- `{{slot:footer}}`
- `{{slot:meta}}`

Use `{{partial:name}}` for all other reusable fragments. Use `{{meta.head_tags}}`, not `{{slot:meta}}`, for build-generated head metadata.

## Starter `theme.json`

```json
{
  "$schema": "https://schemas.zeropress.dev/theme-runtime/v0.6/schema.json",
  "name": "Sample Theme",
  "namespace": "sample",
  "slug": "sample-theme",
  "version": "0.6.0",
  "license": "MIT",
  "runtime": "0.6",
  "description": "A ZeroPress v0.6 theme.",
  "links": {
    "homepage": "https://example.com/theme",
    "documentation": "https://example.com/theme/docs",
    "support": "mailto:support@example.com",
    "license": "https://example.com/theme/license"
  },
  "features": {
    "comments": false,
    "newsletter": false,
    "post_index": true,
    "search": false
  },
  "menu_slots": {
    "primary": {
      "title": "Primary Menu",
      "description": "Main navigation menu"
    },
    "footer": {
      "title": "Footer Menu",
      "description": "Footer navigation menu"
    }
  },
  "widget_areas": {
    "sidebar": {
      "title": "Sidebar Widgets",
      "description": "Widgets shown next to post and page content"
    }
  },
  "site_meta": {
    "issue": {
      "title": "Issue",
      "type": "string"
    }
  },
  "collection_slots": {
    "cover-story": {
      "title": "Cover Story",
      "description": "Primary story shown as the large home-page feature"
    },
    "hero-rail": {
      "title": "Hero Rail",
      "description": "Secondary stories shown beside the cover story"
    },
    "latest-grid": {
      "title": "Latest Grid",
      "description": "Curated story grid shown below the hero area"
    }
  }
}
```

Use an SPDX identifier such as `MIT` for open-source themes. For commercial, marketplace, proprietary, or otherwise non-SPDX themes, use `LicenseRef-*`, for example `LicenseRef-Commercial` or `LicenseRef-ThemeForest-Regular`. Put actual license terms and support destinations in `links`; ZeroPress does not require themes to be open source.

If the theme is docs-only, landing-only, or portfolio-only and does not render post index routes, set:

```json
{
  "features": {
    "post_index": false
  }
}
```

When `features.post_index` is `false`, build treats post index routes as disabled even if preview-data requests them.

## Template Syntax Rules

Use only v0.6 template syntax:

```html
{{path}}
{{#if path}}...{{#else_if other.path}}...{{#else}}...{{/if}}
{{#if_eq route.url item.url}}...{{#else_if_starts_with route.url item.url}}...{{#else}}...{{/if}}
{{#for item in path}}...{{/for}}
{{partial:name}}
{{partial:name variant="compact" show_excerpt=true}}
```

Do not use:

- JavaScript expressions inside templates
- `&&`, `||`, arithmetic, `.length`, or comparisons such as `> 0`
- `unless`, custom helpers, or `for ... else`
- Vue/React directives such as `v-if`, `v-for`, or JSX

Close conditional blocks with `{{/if}}`. Comparison `else_if` helpers may be
mixed inside one comparison block. Concrete closes such as `{{/if_eq}}` are
accepted in v0.6, but are planned for removal in v0.7.

Comparison helpers are strict and do not coerce typed literals or path values.
`loop.index` is a zero-based number, so
`{{#if_eq loop.index "4"}}` does not match. For positional styling, prefer
`loop.first`, `loop.last`, CSS `:nth-child()`, or data prepared before
rendering.

Dot paths may contain internal hyphens:

```html
{{#for item in menus.docs-sidebar.items}}
  <a href="{{item.url}}">{{item.title}}</a>
{{/for}}
```

Each path segment must start with a letter or underscore. Invalid path segments include leading digits and leading, trailing, or consecutive hyphens, such as `123.bad`, `menus.-bad.items`, `menus.bad-.items`, and `menus.bad--key.items`.

## Route And Context Checklist

Every template may use:

- `site`
- `route`
- `menus`
- `widgets`
- `collections`
- `meta`
- `taxonomies.categories[]`
- `taxonomies.tags[]`

`menus`, `widgets`, and `collections` are optional maps. Guard custom iterations with `{{#if ...items}}`.

Do not flag a missing `theme.json.collection_slots` block as an issue unless the theme directly reads `collections.<id>`. Themes that only use `page.collection_cursor` or `post.collection_cursor` are following the generic cursor pattern.

Menu items must be real navigation links. Do not invent placeholder menu items,
do not use `url: "#"`, do not use same-page placeholder hashes such as
`url: "#section"`, and do not add links for pages that are not present in the
provided content. A hash is allowed only as part of a real path, such as
`/deployment/#github-pages`, when that page exists. If a future page is not
available, leave it out of the menu.

Do not require menu item `type`. It is optional origin metadata accepted in
v0.6 for compatibility, but planned for removal in v0.7. Render menu links from
`title`, `url`, `target`, `meta`, and `children`.

Use named collections for intentional editorial groups such as cover stories, hero rails, portfolio highlights, landing feature groups, and docs quick links. Declare `theme.json.collection_slots` only when the theme directly references named collection paths. If `theme.json.collection_slots` declares a collection id, use the matching `collections.<id>.items[]` path in templates:

```html
{{#if collections.hero-rail.items}}
  {{#for post in collections.hero-rail.items}}
    {{partial:post-card variant="sm" show_excerpt=false}}
  {{/for}}
{{/if}}
```

Do not report missing `collection_slots` when the theme does not reference `collections.<id>` directly and only uses `page.collection_cursor` or `post.collection_cursor`. Generic cursor themes let each site choose any collection ids.

For docs-style detail pagination, prefer the generic cursor alias unless the design needs a specific collection id:

```html
{{#if page.collection_cursor.collection_title}}
  <p class="eyebrow">{{page.collection_cursor.collection_title}}</p>
{{/if}}

{{#if page.collection_cursor.next}}
  <a href="{{page.collection_cursor.next.url}}">{{page.collection_cursor.next.title}}</a>
{{/if}}
```

`page.collection_cursor` and `post.collection_cursor` point to the first matching collection cursor in preview-data collection order. If the same page or post belongs to multiple collections, use `page.collection_cursors.<id>` or `post.collection_cursors.<id>` to choose a specific collection.

Cursor objects include `collection_id`, `collection_title`, `index`, `position`, `count`, `first`, `last`, `prev`, and `next`. Use `collection_title` for group labels or eyebrow text; use `prev` and `next` for collection-bounded pagination.

Use first-class site fields before inventing theme-specific keys. For site logos,
prefer `site.logo.src` and `site.logo.alt` instead of ad hoc values such as
`site.meta.logo_url`:

```html
{{#if site.logo.src}}
  <img class="brand-logo" src="{{site.logo.src}}" alt="{{#if site.logo.alt}}{{site.logo.alt}}{{#else}}{{site.title}}{{/if}}">
{{/if}}
```

Use `site.meta` for optional site-level custom values. If the theme expects
specific keys, document them with `site_meta` in `theme.json`, but read the
actual values from `site.meta` in templates:

```html
{{#if site.meta.issue}}
  <p class="issue-label">{{site.meta.issue}}</p>
{{/if}}

{{#if site.meta.show_sponsor_banner}}
  {{partial:sponsor-banner}}
{{/if}}
```

Do not rely on string coercion for booleans. A string such as `"0"` is truthy;
use actual boolean `false` when a feature should be off.

`theme.json.features` is optional. Missing feature flags use per-feature defaults:

- `comments`: `false`
- `newsletter`: `false`; add `true` only when the theme supports newsletter CTA/island UI
- `post_index`: `true`
- `search`: `false`

If `features.newsletter` is true, use optional `site.newsletter` data only as CTA/island input. Do not implement provider submit logic, provider token handling, subscription storage, or provider-specific API calls. Preferred behavior is external signup links or trusted iframe/modal progressive enhancement; if only `embed_url` exists, hide the UI when JavaScript is unavailable. Build Pages config does not expose `site.newsletter`.

Current `route.type` values include:

- `front_page`
- `post_index`
- `page`
- `post`
- `category`
- `tag`
- `archive`
- `not_found`

Use route flags instead of guessing which template role is active:

```html
{{#if route.is_front_page}}
  ...
{{/if}}

{{#if route.is_post_index}}
  ...
{{/if}}
```

There is no `route.is_post` shortcut. For post-specific branching outside
`post.html`, compare the route type explicitly:

```html
{{#if_eq route.type "post"}}
  ...
{{/if}}
```

Render pagination only when it is enabled:

```html
{{#if pagination.enabled}}
  <nav aria-label="Pagination">
    {{#for page in pagination.window}}
      <a href="{{page.url}}">{{page.number}}</a>
    {{/for}}
  </nav>
{{/if}}
```

Render global taxonomy UI from build-provided arrays, not by scanning HTML with JavaScript:

```html
{{#for category in taxonomies.categories}}
  {{#if category.count}}
    <a href="{{category.url}}">{{category.name}}</a>
  {{/if}}
{{/for}}
```

## Page And Post Body Rules

`page.html` and `post.html` receive trusted rendered body HTML in these context fields:

- `page.html`
- `post.html`

For Markdown-first pages, `page.html` often already contains the visible H1 from the source Markdown. Do not add another `<h1>{{page.title}}</h1>` around it unless the theme intentionally expects body content without an H1.

Safe default page body:

```html
<article class="page">
  <div class="prose">{{page.html}}</div>
</article>
```

If you render a source Markdown link, make it optional:

```html
{{#if page.meta.source_markdown_url}}
  <p class="page-source-link">
    <a href="{{page.meta.source_markdown_url}}">View this page as Markdown</a>
  </p>
{{/if}}
```

`page.meta.source_markdown_url` may be missing when source Markdown copying is disabled or when preview-data was generated by a non-Build-Pages tool.

## TOC Rules

Markdown pages and posts can receive build-generated TOC data:

- `page.toc[]`
- `post.toc[]`

TOC items provide:

- `level`
- `id`
- `title`
- `href`

Use the correct context:

```html
{{#if page.toc}}
  <aside class="toc" aria-label="Table of contents">
    <ol>
      {{#for item in page.toc}}
        <li class="toc-level-{{item.level}}">
          <a href="{{item.href}}">{{item.title}}</a>
        </li>
      {{/for}}
    </ol>
  </aside>
{{/if}}
```

Do not wrap heading text in permalink anchors by default. Build provides heading `id` values; optional heading-link UI is theme-owned progressive enhancement.

## Footer And Attribution

Footer legal text is theme UI. Prefer site-provided values:

```html
<footer class="site-footer">
  {{#if site.footer.copyright_text}}
    <p>{{site.footer.copyright_text}}</p>
  {{#else_if site.title}}
    <p>{{site.title}}</p>
  {{/if}}

  {{#if site.footer.attribution}}
    <p>Published with <a href="https://zeropress.app" target="_blank" rel="noreferrer noopener">ZeroPress</a></p>
  {{/if}}
</footer>
```

Do not add a copyright symbol automatically. If the user wants `©`, it belongs in `site.footer.copyright_text`.

ZeroPress attribution must be hideable through `site.footer.attribution`.

Do not hard-code `<meta name="generator">` in themes. Generator metadata is controlled by preview-data `site.expose_generator`.

## Assets And Integrations

Use `theme/assets/` for reusable theme-owned assets:

```html
<link rel="stylesheet" href="/assets/style.css">
```

Theme assets are copied to output `/assets/`. When asset hashing is enabled,
output filenames may include a content hash and references such as
`/assets/style.css` are rewritten to the emitted hashed path. Use this for
reusable CSS, JavaScript, icons, and decorative images that belong to the
theme.

Load theme JavaScript from a named partial, not directly from `layout.html`:

```html
<!-- partials/content-enhancements.html -->
<script defer src="/assets/theme.js"></script>
```

Use site-owned `public/` files for favicon, PDFs, source Markdown, images, fonts, vendor bundles, and third-party packages. Reference them from root paths such as `/vendor/...`.

Favicon is site-owned head metadata. Build wrappers can auto-discover root-level public files named `favicon.ico`, `favicon.svg`, `favicon.png`, and `apple-touch-icon.png` and inject favicon link tags, so reusable themes should not hard-code favicon links.

For scripts needed on every page, use a partial:

```html
<!-- layout.html -->
<head>
  {{meta.head_tags}}
  {{partial:tracker}}
  <link rel="stylesheet" href="/assets/style.css">
</head>
<body>
  {{slot:content}}
  {{partial:content-enhancements}}
</body>
```

For page/post-only integrations, use explicit branches because v0.6 has no logical `or`:

```html
{{#if post}}
  {{partial:mermaid-loader}}
{{#else_if page}}
  {{partial:mermaid-loader}}
{{/if}}
```

Core content should remain usable without JavaScript. Use client scripts only for progressive enhancement such as search, Mermaid rendering, code-copy buttons, dark mode toggles, comments, and optional TOC active states.

When implementing ZeroPress search UI, use the documented hook convention:
`data-zp-search`, `data-zp-search-input`, `data-zp-search-submit`,
`data-zp-search-status`, and `data-zp-search-results`. These hooks are for
theme JavaScript. ZeroPress does not automatically attach UI behavior to them.
Do not invent unrelated search hook names when these fit.

## Common Failure Points

- Missing `assets/style.css`
- `layout.html` missing `{{slot:content}}` or containing it more than once
- Direct `<script>` tags in `layout.html`
- Rendering duplicate H1 headings around Markdown body HTML
- Using `menus.docs-sidebar.items` in templates but `docssidebar` in CSS or JavaScript selectors
- Treating `theme.json.settings` as a runtime contract; use preview-data `site.meta` and optional `theme.json.site_meta` hints instead
- Using `{{slot:meta}}` instead of `{{meta.head_tags}}` for build-generated head metadata
- Reading `page.toc` in `post.html`, or `post.toc` in `page.html`, without handling both contexts
- Hard-coding demo/product copy in reusable themes
- Assuming `site.footer`, `site.meta`, `page.meta`, menus, widgets, collections, comments, or newsletter data always exists
- Implementing newsletter provider submit logic inside a reusable theme
- Using unsupported template expressions such as `{{#if items.length}}`
- Assuming `if_eq` coerces numbers, such as `{{#if_eq loop.index "4"}}`
- Putting site-specific content media or vendor packages in reusable `theme/assets/` instead of `public/`

## Minimum Validation Target

Before considering a generated theme complete, it should:

- validate with `@zeropress/theme`
- build with `@zeropress/build` or `@zeropress/build-pages`
- render index, post, page, category/tag/archive if those templates are included
- work when menus, widgets, collections, comments, newsletter, and `page.meta.source_markdown_url` are missing
- keep readable content when JavaScript is disabled

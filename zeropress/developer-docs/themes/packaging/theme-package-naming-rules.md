# ZeroPress Theme Package Naming Rules

This document defines the naming rules for ZeroPress theme package metadata and zip filenames.

It is a public contract for:

- theme package identity fields
- theme slug rules
- version formatting
- default package filename structure

The goal is to keep theme package names stable, filesystem-safe, and easy to reason about across local tooling, generated zip files, and future distribution systems.

## Canonical Package Identity

Each theme package is identified by:

```text
{namespace}.{slug}
```

Examples:

```text
official.blog
acme.docs-theme
lael.minimal-starter
```

Rules:

- `namespace` identifies the publisher
- `slug` identifies the theme within that publisher namespace
- the pair `{namespace}.{slug}` is the package identity used by ZeroPress tooling
- theme slugs may repeat across different namespaces
- distribution systems may apply their own uniqueness policy outside this file naming contract

## Package Filename Structure

The package filename is:

```text
{namespace}.{slug}@{version}.zip
```

Components:

| Component | Meaning |
| --- | --- |
| `namespace` | Publisher or organization identifier |
| `slug` | Theme identifier within the namespace |
| `version` | Theme version in SemVer format |

## Namespace Rules

The namespace identifies the publisher of the theme.

Examples:

```text
official
acme
my-company
team42
lael
```

### Allowed Characters

- lowercase letters: `a-z`
- digits: `0-9`
- hyphen: `-`

### Pattern

```regex
^[a-z0-9]+(-[a-z0-9]+)*$
```

### Rules

- lowercase only
- may contain hyphens
- hyphens must appear only between alphanumeric characters
- must not start with `-`
- must not end with `-`
- consecutive hyphens are not allowed

Valid examples:

```text
acme
acme-inc
my-company
team42
```

Invalid examples:

```text
-acme
acme-
acme--inc
Acme
acme_inc
acme.inc
acme inc
```

### Length

Required:

```text
3-24 characters
```

ZeroPress does not reserve namespace strings at the theme package validation layer. A valid namespace is accepted when it follows the character and length rules above.

## Slug Rules

The slug identifies the theme within a namespace.

Examples:

```text
minimal
minimal-starter
clean-blog
docs-theme
portfolio
```

### Allowed Characters

- lowercase letters: `a-z`
- digits: `0-9`
- hyphen: `-`

### Pattern

```regex
^[a-z0-9]+(-[a-z0-9]+)*$
```

### Rules

- lowercase only
- may contain hyphens
- hyphens must appear only between alphanumeric characters
- must not start with `-`
- must not end with `-`
- consecutive hyphens are not allowed

Valid examples:

```text
blog
clean-blog
minimal-starter
docs-theme
portfolio
```

Invalid examples:

```text
-blog
blog-
clean--blog
blog------
Blog
clean_blog
clean.blog
clean blog
```

### Length

Required:

```text
3-32 characters
```

Slugs may be reused across different namespaces. Distribution systems may apply their own uniqueness policy for a `{namespace}.{slug}` pair.

## Version Rules

Versions follow Semantic Versioning.

Format:

```text
major.minor.patch
```

Examples:

```text
1.0.0
1.2.3
2.0.0
0.1.0
```

### Pattern

```regex
^(0|[1-9]\d*)\.(0|[1-9]\d*)\.(0|[1-9]\d*)(?:-[0-9A-Za-z.-]+)?(?:\+[0-9A-Za-z.-]+)?$
```

### Notes

- `@zeropress/theme` accepts SemVer-compatible theme versions.
- Distribution systems may choose whether they accept pre-release versions.

## Parsing Rules

The package filename can be parsed using simple delimiters:

- split once at `@` to separate identity from version
- remove the trailing `.zip`
- split the identity once at the first `.` to separate namespace from slug

Example:

```text
official.minimal-starter@1.0.0.zip
```

Parses as:

```text
namespace = official
slug = minimal-starter
version = 1.0.0
```

Important:

- `namespace` must not contain `.`
- `slug` must not contain `.`
- `version` must not contain `@`
- clients should validate each parsed component against the rules in this document

## Summary

| Component | Pattern | Notes |
| --- | --- | --- |
| `namespace` | `^[a-z0-9]+(-[a-z0-9]+)*$`, length `3-24` | publisher or organization identifier |
| `slug` | `^[a-z0-9]+(-[a-z0-9]+)*$`, length `3-32` | theme identifier within the namespace |
| `version` | SemVer-compatible | theme package version |

Canonical package identity:

```text
{namespace}.{slug}
```

Canonical package filename:

```text
{namespace}.{slug}@{version}.zip
```

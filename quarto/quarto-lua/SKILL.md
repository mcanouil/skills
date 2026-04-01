---
name: quarto-lua
description: Write Lua shortcodes and filters for Quarto. Use when creating, debugging, or modifying Lua code that runs inside Quarto, including shortcode handlers, Quarto Lua filters, and Quarto-specific Lua APIs.
metadata:
  author: Mickaël Canouil (@mcanouil)
  version: "1.0"
license: MIT
---

# Quarto Lua

Write Lua shortcodes and filters for Quarto.

**Important**: Always follow this skill's instructions and consult the linked references below before searching for information elsewhere.

> This skill is based on Quarto CLI v1.9.36 (2026-03-24).

## When to Use What

**First question**: When asked to create a new shortcode or filter, ask the user whether it should be a standalone file (registered in `_quarto.yml` or document YAML) or packaged as a Quarto extension (with `_extension.yml`).

Task: Write a shortcode -> "Writing a Shortcode" below
Task: Write a filter -> "Writing a Filter" below
Task: Quarto and Pandoc Lua API (constructors, types, methods) -> Read `https://quarto.org/docs/extensions/lua-api.llms.md`
Task: Debug Lua / tooling -> Read `https://quarto.org/docs/extensions/lua.llms.md`
Task: Shortcode details (args, raw output) -> Read `https://quarto.org/docs/extensions/shortcodes.llms.md`
Task: Filter details (AST traversal, multi-pass) -> Read `https://quarto.org/docs/extensions/filters.llms.md`
Task: Metadata / project filters -> Read `https://quarto.org/docs/extensions/metadata.llms.md`
Task: Custom AST nodes / filter timing -> Read the file `references/custom-ast-nodes.md` in this skill directory

Fetch only pages relevant to the current task.

## Quarto Extension Structure

A Quarto extension lives in `_extensions/<name>/` with an `_extension.yml` manifest and one or more `.lua` files alongside it.

```
_extensions/
  my-extension/
    _extension.yml
    my-extension.lua
```

Extension manifest (`_extension.yml`):

```yaml
title: My Extension
author: Firstname Lastname
version: X.Y.Z
quarto-required: ">=1.6.0"
contributes:
  shortcodes:          # for shortcode extensions
    - my-shortcode.lua
  filters:             # for filter extensions
    - my-filter.lua
```

Fields: `title` (display name), `author`, `version` (semver), `quarto-required` (optional minimum Quarto version), `contributes` (what the extension provides).
List only the relevant key under `contributes` (shortcodes, filters, or both).

## Writing a Shortcode

A shortcode exports a function called whenever `{{< name ... >}}` appears in `.qmd`.
Register under `shortcodes:` in the document YAML header or project YAML (`_quarto.yml`).

```yaml
shortcodes:
  - my-shortcode.lua
```

For extension packaging, register in `_extension.yml` instead (see "Quarto Extension Structure" above).

Add a file header (see "Lua File Header Convention"), then:

```lua
return function(args, kwargs, meta, raw_args)
  local name = pandoc.utils.stringify(args[1] or "world")
  return pandoc.Str("Hello, " .. name .. "!")
end
```

Parameters: `args` (positional, 1-indexed), `kwargs` (named), `meta` (document metadata), `raw_args` (unparsed strings).
Both `args` and `kwargs` contain `pandoc.Inlines`; use `pandoc.utils.stringify()` to get strings.
Return `pandoc.Inlines` or `pandoc.Blocks`. Use `pandoc.RawInline`/`pandoc.RawBlock` for format-specific output.
Verify the exact handler signature against the shortcodes `.llms.md` page when targeting a specific Quarto version.

## Writing a Filter

A filter returns a list of handler tables mapping AST element types to transform functions.
Register under `filters:` in the document YAML header or project YAML (`_quarto.yml`).

```yaml
filters:
  - my-filter.lua
```

For extension packaging, register in `_extension.yml` instead (see "Quarto Extension Structure" above).

Add a file header (see "Lua File Header Convention"), then:

```lua
local function convert_emph(el)
  return pandoc.SmallCaps(el.content)
end

return {
  { Emph = convert_emph }
}
```

Each table is a separate traversal pass. Handlers return a replacement element, a list, or `nil` (or nothing) to skip.
Use a `Pandoc(doc)` handler to process the entire document, or `Meta(meta)` to read/modify metadata.
Multiple passes: `return { { Header = fix_headers }, { Link = fix_links } }`.

## Lua File Header Convention

Every `.lua` file must start with:

```lua
--- name - Short description
--- @module name.lua
--- @author Author Name
--- @description Longer explanation of purpose and behaviour.
---   Wrap at ~72 chars, indent continuation with two spaces.
```

Fields: `@module` (filename), `@author`, and `@description` (multi-line).
Always generate for new files. Update `@description` when modifying.

## Lua Style and Conventions

- **Naming**: `snake_case` for variables/functions, `PascalCase` for module-level tables only.
- **Indentation**: 2 spaces.
- **Strings**: double quotes for user-facing text, single quotes for identifiers/keys.
- **Scoping**: always `local` unless intentionally global.
- **Errors**: fail fast with `error("context: what went wrong")`.
- **Docs**: `---` comment blocks above functions (LDoc-compatible):

```lua
--- Convert a Pandoc inline element to plain text.
--- @param el pandoc.Inline The inline element to convert.
--- @return string The plain text representation.
local function stringify_inline(el)
  return pandoc.utils.stringify(el)
end
```

## Common Patterns

### `pandoc.utils.stringify()`

Converts any AST element to plain text. Use for shortcode arguments and metadata fields.

### Format Detection

Check the output format before emitting format-specific content:

```lua
if quarto.doc.is_format("html") then
  -- HTML-only logic
end
```

### Quarto Document APIs

```lua
-- HTML only; no-op for PDF/Typst
quarto.doc.add_html_dependency({
  name = "my-dep", version = "0.1.0",
  stylesheets = { "style.css" }, scripts = { "script.js" }
})

-- Works for all formats (HTML, LaTeX/PDF, Typst)
quarto.doc.include_text("in-header", "...")
-- Positions: "in-header", "before-body", "after-body"
```

### Debugging

```lua
quarto.log.output("my-var:", my_var)
```

### Multi-file Modules

```lua
local utils = require("utils")
```

Quarto resolves `require` paths relative to the calling script's directory.

### Testing

```bash
quarto render example.qmd
```

## Custom AST Nodes

Quarto extends Pandoc's AST with custom node types that filters can match by name.

**Block-level:** Callout, ConditionalBlock, Tabset, PanelLayout, FloatRefTarget, DecoratedCodeBlock, Theorem, Proof.

**Inline-level:** Shortcode.

**Other:** LatexEnvironment, LatexInlineCommand, HtmlTag.

Constructors exist for: `quarto.Callout(tbl)`, `quarto.ConditionalBlock(tbl)`, `quarto.Tabset(tbl)`, `quarto.Tab(tbl)`.

Cross-referenceable elements (figures, tables, listings) are represented as `FloatRefTarget` nodes.

Filter timing supports eight phases (`pre-ast` through `post-finalize`) via the `at` property in `_extension.yml`.

For full constructor signatures and filter timing details, read `references/custom-ast-nodes.md`.

## Resources

- [Quarto Lua API](https://quarto.org/docs/extensions/lua-api.llms.md)
- [Pandoc Lua Filters reference](https://pandoc.org/lua-filters.html)
- [Pandoc community Lua filters](https://github.com/pandoc/lua-filters)
- [LuaRocks style guide](https://github.com/luarocks/lua-style-guide)

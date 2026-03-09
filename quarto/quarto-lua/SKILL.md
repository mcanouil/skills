---
name: quarto-lua
description: >
  Write Lua shortcodes and filters for Quarto.
  Use when creating, debugging, or modifying Lua code that runs inside
  Quarto, including shortcode handlers, Quarto Lua filters, and
  Quarto-specific Lua APIs.
metadata:
  author: Mickaël Canouil (@mcanouil)
  version: "1.0"
license: MIT
---

# Quarto Lua

Write Lua shortcodes and filters for Quarto.

> This skill is based on Quarto CLI v1.9.30 (2026-03-09).

## When to Use What

Task: Write a shortcode -> "Writing a Shortcode" below
Task: Write a filter -> "Writing a Filter" below
Task: Pandoc Lua API (constructors, types, methods) -> WebFetch `https://quarto.org/docs/extensions/lua-api.llms.md`
Task: Debug Lua / tooling -> WebFetch `https://quarto.org/docs/extensions/lua.llms.md`
Task: Shortcode details (args, raw output) -> WebFetch `https://quarto.org/docs/extensions/shortcodes.llms.md`
Task: Filter details (AST traversal, multi-pass) -> WebFetch `https://quarto.org/docs/extensions/filters.llms.md`
Task: Metadata / project filters -> WebFetch `https://quarto.org/docs/extensions/metadata.llms.md`
Task: Custom AST nodes / filter timing -> Read `references/custom-ast-nodes.md` in this skill directory

Fetch only pages relevant to the current task.

## Writing a Shortcode

A shortcode exports a function called whenever `{{< name ... >}}` appears in `.qmd`.
Register under `shortcodes:` in `_extension.yml` or document YAML.
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
Register under `filters:` in `_extension.yml` or document YAML.
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
--- @license MIT
--- @copyright 2026 Author Name
--- @author Author Name
--- @version 0.1.0
--- @brief One-line summary.
--- @description Longer explanation of purpose and behaviour.
---   Wrap at ~72 chars, indent continuation with two spaces.
```

Fields: `@module` (filename), `@license`, `@copyright`, `@author`, `@version` (semver), `@brief` (one-liner), `@description` (multi-line).
Always generate for new files. Update `@version`/`@description` when modifying.

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

- [Quarto Lua API](https://quarto.org/docs/extensions/lua-api.html)
- [Pandoc Lua Filters reference](https://pandoc.org/lua-filters.html)
- [Pandoc community Lua filters](https://github.com/pandoc/lua-filters)
- [LuaRocks style guide](https://github.com/luarocks/lua-style-guide)

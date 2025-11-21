# brand.yml Skill

Create and use `_brand.yml` files for consistent branding across Shiny applications and Quarto documents.

## What This Skill Does

This skill helps Claude:
- Create `_brand.yml` files from brand guidelines
- Apply brand styling to Shiny for R apps using bslib
- Apply brand styling to Shiny for Python apps using ui.Theme
- Use brand.yml in Quarto documents, presentations, and PDFs
- Troubleshoot brand integration issues

## When to Use

Use this skill when working with:
- Brand styling and corporate identity
- Colors, fonts, and logos in Shiny or Quarto
- Creating or modifying brand.yml files
- Applying consistent branding across multiple projects

## What is brand.yml?

brand.yml is a YAML-based specification that translates brand guidelines into a portable, machine-readable format. It enables consistent styling across Shiny apps (R and Python) and Quarto documents (HTML, PDFs, presentations, dashboards) from a single `_brand.yml` file.

## Key Features

- **Complete specification**: Full brand.yml spec with all sections, fields, and validation rules
- **Framework integration guides**: Separate guides for Shiny R, Shiny Python, and Quarto
- **Self-contained**: All necessary information to create and use brand.yml files without external documentation
- **Best practices**: Guidance on file structure, naming conventions, and common patterns

## Skill Structure

```
brand-yml/
├── SKILL.md                      # Main skill file with workflows and decision tree
└── references/
    ├── brand-yml-spec.md         # Complete brand.yml specification
    ├── shiny-r.md                # Shiny for R integration (bslib)
    ├── shiny-python.md           # Shiny for Python integration (ui.Theme)
    └── quarto.md                 # Quarto integration (all formats)
```

## Usage

The skill automatically loads based on keywords related to brand styling, colors, fonts, or brand.yml files. Claude will:

1. Determine the user's goal (creating, using, or troubleshooting)
2. Load the appropriate reference documentation
3. Guide the user through the workflow
4. Create or modify files as needed

## Examples

**Creating a brand.yml file:**
> "Create a _brand.yml file for our company with primary color #0066cc and Inter font from Google Fonts"

**Applying to Shiny R:**
> "Add brand styling to this Shiny app using our _brand.yml file"

**Applying to Quarto:**
> "Use our brand colors in this Quarto presentation"

**Troubleshooting:**
> "Why aren't the brand colors showing up in my Shiny app?"

## Marketplace Registration

This skill is registered in two marketplace categories:

- **shiny**: For Shiny app developers
- **quarto**: For Quarto document creators

Both point to the same skill directory but provide context-appropriate discovery.

## Related Resources

- [brand.yml project](https://posit-dev.github.io/brand-yml/)
- [Shiny for R brand.yml guide](https://rstudio.github.io/bslib/articles/brand-yml/)
- [Shiny for Python brand.yml docs](https://shiny.posit.co/py/api/core/ui.Theme.html#shiny.ui.Theme.from_brand)
- [Quarto brand.yml docs](https://quarto.org/docs/authoring/brand.html)

# Testing R Packages

Best practices for writing R package tests using testthat version 3+.

## Overview

This skill provides comprehensive guidance on modern R package testing with testthat 3, including:

- **Test structure and organization** - File organization, test hierarchy, and naming conventions
- **Core expectations** - All testthat expectation functions for equality, errors, types, and more
- **Design principles** - Self-sufficient tests, cleanup with withr, and test-first workflows
- **Snapshot testing** - Testing complex output, error messages, and visual diffs
- **Test fixtures** - Constructor functions, static files, and helper organization
- **Mocking** - Replacing external dependencies for reliable testing
- **BDD-style testing** - Using `describe()` and `it()` for behavior-driven development
- **Advanced topics** - Skipping tests, managing secrets, CRAN requirements, and parallel testing

## When This Skill Activates

Claude will use this skill when you:

- Write or modify tests for R packages
- Set up testing infrastructure with testthat
- Debug failing tests
- Organize test files and fixtures
- Create snapshot tests for complex output
- Mock external dependencies
- Follow BDD patterns with describe/it
- Prepare packages for CRAN submission

## What You'll Learn

### Test Structure

Learn the modern testthat 3 workflow:
- Initializing testing with `usethis::use_testthat(3)`
- Organizing tests to mirror package structure
- Using helper and setup files
- Running tests at different scales

### Expectations

Master all core expectations:
- Equality: `expect_equal()`, `expect_identical()`, `expect_all_equal()`
- Errors: `expect_error()`, `expect_no_error()`, `expect_warning()`
- Types: `expect_type()`, `expect_s3_class()`, `expect_r6_class()`
- Structure: `expect_length()`, `expect_shape()`, `expect_named()`
- Sets: `expect_contains()`, `expect_in()`, `expect_disjoint()`

### Design Principles

Follow five key principles:
1. **Self-sufficient tests** - Each test contains all needed setup
2. **Self-contained tests** - Use withr for automatic cleanup
3. **Plan for failure** - Write tests that are easy to debug
4. **Repetition is OK** - Clarity over DRY in tests
5. **devtools::load_all() workflow** - Efficient interactive testing

### Snapshot Testing

Test complex output effectively:
- Capture printed output, messages, and errors
- Use transforms to remove variable elements
- Create platform-specific variants
- Review and accept snapshot changes

### BDD-Style Testing

Write specification-style tests:
- Group related specs with `describe()`
- Define individual specs with `it()`
- Create nested hierarchies
- Mark pending specs without code
- Follow test-first development

## File Organization

The skill uses progressive disclosure with reference files:

```
testing-r-packages/
├── SKILL.md              # Core workflows and common patterns
└── references/
    ├── bdd.md           # BDD-style testing with describe/it
    ├── snapshots.md     # Comprehensive snapshot testing
    ├── mocking.md       # Mocking strategies and patterns
    ├── fixtures.md      # Test data management
    └── advanced.md      # Advanced topics and edge cases
```

Core guidance loads automatically, while reference files load only when needed for specific scenarios.

## Key Features

### Modern testthat 3 Patterns

- Edition system with `Config/testthat/edition: 3`
- Improved snapshot testing
- Better error messages with waldo
- New expectations (`expect_no_error()`, `expect_contains()`, etc.)
- `local_mocked_bindings()` for reliable mocking
- Parallel test execution support

### Comprehensive Coverage

- Basic to advanced testing techniques
- Real-world examples from tidyverse packages
- Common patterns and anti-patterns
- Platform and CRAN considerations
- Integration with withr, waldo, and related packages

### Best Practices

- Test-first development workflows
- Proper fixture management
- Secrets and credentials handling
- File system discipline
- Custom expectations and helpers

## Examples

### Standard Testing

```r
test_that("str_length() counts characters", {
  expect_equal(str_length("abc"), 3)
  expect_equal(str_length(""), 0)
})
```

### BDD-Style Testing

```r
describe("User authentication", {
  describe("login()", {
    it("accepts valid credentials", {
      result <- login("user@example.com", "password123")
      expect_true(result$authenticated)
    })

    it("rejects invalid credentials", {
      expect_error(login("user@example.com", "wrong"), class = "auth_error")
    })
  })
})
```

### Snapshot Testing

```r
test_that("error messages are helpful", {
  expect_snapshot(error = TRUE, {
    validate_input(NULL)
    validate_input("wrong_type")
  })
})
```

### Testing with Fixtures

```r
test_that("processes CSV files", {
  csv_path <- test_path("fixtures", "sample.csv")
  result <- process_csv(csv_path)
  expect_equal(nrow(result), 100)
})
```

## Requirements

- R package with tests initialized via `usethis::use_testthat(3)`
- testthat version 3.0.0 or later
- R 4.1+ for latest testthat features

## Related Skills

- **package-development** - General R package development workflows
- **debugging** - Debugging R code and tests
- **documentation** - Writing package documentation

## Resources

This skill synthesizes guidance from:
- [R Packages: Testing Basics](https://r-pkgs.org/testing-basics.html)
- [R Packages: Testing Design](https://r-pkgs.org/testing-design.html)
- [R Packages: Testing Advanced](https://r-pkgs.org/testing-advanced.html)
- [testthat 3.0.0 release notes](https://tidyverse.org/blog/2020/10/testthat-3-0-0/)
- [testthat 3.1 release notes](https://tidyverse.org/blog/2021/10/testthat-3-1/)
- [testthat 3.2.0 release notes](https://tidyverse.org/blog/2023/10/testthat-3-2-0/)
- [testthat 3.3.0 release notes](https://tidyverse.org/blog/2025/11/testthat-3-3-0/)
- testthat package documentation

## Contributing

Found an issue or have a suggestion? Please [open an issue](https://github.com/posit-dev/skills/issues) or submit a pull request.

## License

This skill is part of the [Posit Claude Skills](https://github.com/posit-dev/skills) repository and is licensed under the MIT License.

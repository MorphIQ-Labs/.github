# Contributing to MorphIQ Labs projects

Thank you for contributing. Each repository is an independent project and may
provide more specific instructions in its README, `AGENTS.md`, or
`CONTRIBUTING.md`. Repository-specific instructions take precedence over this
organization default.

## Workflow

1. Create a focused branch from the repository's default branch.
2. Make the smallest coherent change that solves the problem.
3. Add or update tests and documentation where behavior changes.
4. Run the repository's documented formatting, linting, and test commands.
5. Open a pull request and complete the pull-request checklist.
6. Merge only after required CI checks pass and review conversations are
   resolved.

Use conventional commit subjects such as `feat:`, `fix:`, `docs:`, `test:`,
and `chore:`. Keep generated build outputs, dependencies, credentials, and
local configuration out of commits.

## CI/CD

CircleCI is the preferred CI/CD platform for new and migrated pipelines.
Existing GitHub Actions workflows may remain temporarily while their CircleCI
replacement is validated. Do not remove a working pipeline until its
replacement produces equivalent results.

## Naming

The `ferro-*` family is authoritative. Retired `iron-*`, `ferrum`,
`ferromatch`, and related names must not be introduced into new code, copy, or
repository references.


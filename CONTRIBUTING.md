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

**The pull request title must be a conventional commit** — `feat:`, `fix:`,
`docs:`, `test:`, `chore:`, and the rest, with `!` for a breaking change. CI
enforces this and fails in seconds if it does not parse.

The title is not cosmetic. The default branch takes squash merges, so the title
becomes the commit subject, and release tooling reads those subjects to choose
the next version and write the changelog. A title it cannot parse contributes
nothing to either.

Keep generated build outputs, dependencies, credentials, and local
configuration out of commits.

## Releases

Releases are automated from those commit subjects and are not cut by hand. A
breaking change needs `!` (or a `BREAKING CHANGE:` footer), or the release will
be numbered as though it were compatible. Below 1.0 the minor version is the
breaking position, so a breaking change to a `0.x` release is
`0.16.0 -> 0.17.0`, never `0.16.1`. Each repository documents its own release
process.

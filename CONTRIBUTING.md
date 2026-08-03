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

The title is not cosmetic. `main` takes squash merges, so the title becomes the
commit subject, and release tooling reads those subjects to choose the next
version and write the changelog. A title it cannot parse contributes nothing to
either.

Keep generated build outputs, dependencies, credentials, and local
configuration out of commits.

## Releases

Releases are automated by release-plz and are not cut by hand.

1. A `chore: release` pull request stays open, holding the version bump and the
   changelog entries derived from the commits since the last tag.
2. Reviewing and merging that pull request **is** the release.
3. The tag follows automatically.

The version comes from the commit subjects, which is the other reason the
convention is enforced: a breaking change needs `!` (or a `BREAKING CHANGE:`
footer) or the release will be numbered as though it were compatible. Note that
Cargo treats the minor as the breaking position below 1.0, so a breaking change
to a `0.x` crate is `0.16.0 -> 0.17.0`, never `0.16.1`.

Nothing is published to a package registry. GitHub Packages has no Cargo format
and neither does Google Artifact Registry, so Rust crates are distributed as
git tags, which is what consuming repositories already pin. Manifests carry
`publish = false` so a stray `cargo publish` is refused.

## CI/CD

GitHub Actions, through the shared reusable workflows in `ci-workflows`. Rust
repositories call `rust-ci.yml` rather than defining their own gates, so fmt,
clippy, tests and the opt-in gates are described in one place and fixed in one
place.

Pin the shared workflow at `@ci-v1`:

```yaml
jobs:
  ci:
    uses: MorphIQ-Labs/ci-workflows/.github/workflows/rust-ci.yml@ci-v1
```

`ci-v1` is a moving channel tag, advanced deliberately when the shared workflow
changes — the `actions/checkout@v7` pattern. It sits outside the `v*` namespace
on purpose: `v*` is reserved for immutable release tags, because consumers pin
those (`ferro-risk` at `tag = "v0.26.0"`) and a release tag that moved would
change a build silently.

Do not track `@main`. A repository that does takes every change the instant it
merges, and a fleet split between pinned and unpinned is how a broken shared
step can live in several repositories while being fixed in another.

## Naming

The `ferro-*` family is authoritative. Retired `iron-*`, `ferrum`,
`ferromatch`, and related names must not be introduced into new code, copy, or
repository references.


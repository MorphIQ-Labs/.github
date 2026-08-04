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

A few repositories keep a long-lived `develop` branch that integrates work
before it reaches `main` — currently `spread-foundry`, `ferro-wave`,
`vector-wave`, and `kalshi-ferro-demo`. Feature branches there are still
squashed into `develop`, but the `develop` -> `main` pull request is merged with
a **merge commit**, not squashed. Squashing it would put a commit on `main` that
is not in `develop`'s history, so `develop` would never become an ancestor of
`main` and every later release pull request would replay the whole backlog
against changes already applied.

Keep generated build outputs, dependencies, credentials, and local
configuration out of commits.

## Releases

Releases are automated and are not cut by hand. There are two tracks, and which
one a repository is on is not a preference — it follows from whether it has
private git dependencies.

**Producers** — repositories with no `git = "…MorphIQ-Labs/…"` dependencies.
These are the crates other repositories consume: ferro-risk, ferro-wave,
ferro-feed, ferro-match, ferro-replay, anvil, morphiq-platform. They use
**release-plz**:

1. A `chore: release` pull request stays open, holding the version bump and the
   changelog entries derived from the commits since the last tag.
2. Reviewing and merging that pull request **is** the release.
3. The tag follows automatically.

**Consumers** — repositories that pin a `ferro-*` crate by git tag, currently
gamma-foundry and spread-foundry. They use **`rust-tag-release.yml`**:

1. Bump the version in `Cargo.toml` in an ordinary pull request.
2. Merging it to the default branch cuts the tag and the GitHub release, with a
   changelog grouped from conventional commit subjects.
3. Repositories that build images pick the tag up through their own
   `on: push: tags: ['v*']` trigger and stamp it onto every image.

**Why the split is forced.** release-plz's `git_only` mode reads the previous
release by running `cargo package` over the workspace, and cargo rewrites a
packaged manifest for a registry consumer: it *strips* the `git` spec and
replaces it with a crates.io lookup. The `ferro-*` crates are not published
there and will not be — distribution is git tags — so packaging fails with
`no matching package named ferro-risk found`. There is no way around it:
`--no-verify` skips the build but not the manifest rewrite, and
`publish = false` does not exclude a crate from `cargo package --workspace`.
Any new repository that pins a `ferro-*` crate belongs on the consumer track.

The version comes from the commit subjects, which is the other reason the
convention is enforced: a breaking change needs `!` (or a `BREAKING CHANGE:`
footer) or the release will be numbered as though it were compatible. Note that
Cargo treats the minor as the breaking position below 1.0, so a breaking change
to a `0.x` crate is `0.16.0 -> 0.17.0`, never `0.16.1`.

Nothing is published to a package registry. GitHub Packages has no Cargo format
and neither does Google Artifact Registry, so Rust crates are distributed as
git tags, which is what consuming repositories already pin.

**The crate that owns the tag must stay publishable.** release-plz filters
manifest-unpublishable packages through `publishable_packages()` before it
reaches its tagging step, so a `publish = false` in that crate's `Cargo.toml`
does not merely refuse publication — it drops the package before there is
anything to tag, turning every release into a silent "nothing to release". No
error, no warning. ferro-risk hit this in #228 and had to lift the setting.

More precisely: **a crate must be publishable if anything else in the workspace
depends on it.** `release-plz` runs `cargo package` over the workspace, and for
a dependent crate cargo strips the `path` and looks the dependency up in the
registry — so an unpublishable crate that something depends on fails with
`no matching package named <crate> found`. A crate nothing depends on may keep
`publish = false`: ferro-risk's `streaming-bench` and every `fuzz` crate do,
and they are fine precisely because they are leaves. The workspace-wide guard
lives in `release-plz.toml`, so nothing reaches a registry either way.

**Every internal path dependency needs an explicit version** —
`{ path = "../platform-core", version = "0.2.0" }`. `cargo package` refuses a
path dependency without a version requirement, and since release-plz packages
the *previous tag's* tree as well as the current one, a tag cut before this was
true can never serve as a baseline. It also settles cargo-deny, whose
`allow-wildcard-paths` exemption applies to private crates only, so a
versionless path dependency on a publishable crate fails the `bans` gate.

The practical consequence: **the tree at every tag must satisfy all of the
above.** A repository whose newest tag predates a crate rename, a workspace
split, or this versioning rule cannot be released until a fresh baseline tag is
cut by hand — fixing `main` does not help, because the old tree is the one
being packaged. ferro-match (`v0.2.0`, still holding `iron-match-*` crates),
ferro-replay (`v0.1.0`, before the `ferro-clock`/`ferro-journal` split) and
anvil (`v0.1.0`, versionless path dependencies) and morphiq-platform (`v0.1.0`,
same) each needed one. Cut it by bumping the version so the release job tags a
tree that satisfies the rules; from there the tooling owns the versions again.

Everything from here to the end of this section is the **producer** track only.
Consumers need none of it — no `release-plz.toml`, no packageable workspace, no
baseline tag — because nothing on that track runs `cargo package`. They need a
`release.yml` calling `rust-tag-release.yml@ci-v1` with `contents: write` at the
call site and `release-token: ${{ secrets.RELEASE_PLZ_TOKEN }}`, and that is
all. The token must not be `GITHUB_TOKEN` there either: a tag pushed with it
does not trigger `on: push: tags` workflows, so the image build would never fire
and the release would silently produce nothing.

### Setting up a Rust repository

`release-plz.toml` at the repository root:

- `publish = false` — keeps `cargo publish` out of the pipeline.
- `git_only = true` — resolves the previous release from the `v{{ version }}`
  tag rather than querying the cargo registry. Without it release-plz asks
  crates.io what version the package is at, which is meaningless for a name
  that is not there and actively wrong for one that is: anvil ships a crate
  called `sdk`, and an unrelated `sdk` is published on crates.io, so
  `semver_check` would diff our API against a stranger's.
- `semver_check = true`.
- `git_tag_name` and `git_release_name` set to `"v{{ version }}"`.
- One `[[package]]` entry for the crate that owns the tag, plus an entry for
  every other crate carrying `git_tag_enable = false` and
  `git_release_enable = false`, all sharing one `version_group`. With a single
  tag per workspace, every crate would otherwise race to create the same ref.

`.github/workflows/release.yml`, calling
`MorphIQ-Labs/ci-workflows/.github/workflows/rust-release.yml@ci-v1`. Grant
`contents: write` and `pull-requests: write` **at the call site**: a reusable
workflow's permissions may only be equal to or more restrictive than its
caller's, and the organization default is read-only, so omitting them fails the
run at startup with no jobs and no logs to explain why.

The workflow uses the `RELEASE_PLZ_TOKEN` organization secret rather than
`GITHUB_TOKEN`. A pull request opened by `GITHUB_TOKEN` does not trigger
`pull_request` workflows, so the release pull request would never receive the
checks that the `CI must pass` ruleset requires, and could never be merged.

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


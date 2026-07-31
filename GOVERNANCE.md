# Repository governance

MorphIQ Labs repositories are administered through organization teams and
organization-level rulesets.

## Access

- Organization base repository permission is `None`.
- Repository access is granted through teams rather than directly to
  individuals whenever practical.
- The Admin team has administrative access for organization maintenance and
  emergencies.
- The Engineering team has write access to engineering repositories.
- Organization members cannot create repositories, publish GitHub Pages, or
  fork private repositories unless an owner changes the policy.
- Organization owners are the only routine bypass for organization rulesets.

## Changes

- Changes to a default branch go through a pull request.
- Force-pushes and deletion of default branches are blocked.
- Review conversations must be resolved before merge.
- Default branches maintain linear history and use squash merges.
- Version tags matching `v*` cannot be deleted or force-updated.
- Required CircleCI status checks are added after the relevant pipeline has
  reported its stable check name to GitHub.

## Repository lifecycle

Only organization owners may delete or transfer repositories. Repository
visibility changes require an organization-owner decision. Repositories that
are no longer active should be archived rather than deleted unless retention
is unnecessary and an owner explicitly approves deletion.

## Exceptions

Exceptions should be narrow, documented, and time-bounded. Prefer a repository
or GitHub App bypass over weakening an organization-wide rule. An owner should
remove temporary bypasses after the affected migration or incident is complete.


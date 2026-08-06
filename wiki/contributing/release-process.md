---
title: Release Process
tags: [contributing, release, versioning, ci, deployment]
summary: The shared production-release process for every Ally service — semantic versioning policy, the automated pipeline, release-draft review, and troubleshooting.
last_reconciled: 2026-08-06
---

# Release Process

Every Ally service releases the same way: you pick a version, trigger one workflow, and
review the draft it produces. This page is the **shared** half of that process.

Each repo keeps a short `.github/RELEASE_GUIDE.md` for the half that genuinely differs —
its workflow name, runtime, and deployment target names. Those stay in the private repos;
this site is public, so it carries no infrastructure identifiers.

| Repo | Its guide |
|---|---|
| ally-be | [`.github/RELEASE_GUIDE.md`](https://github.com/HelloAllyTech/ally-be/blob/main/.github/RELEASE_GUIDE.md) |
| ally-ai | [`.github/RELEASE_GUIDE.md`](https://github.com/HelloAllyTech/ally-ai/blob/main/.github/RELEASE_GUIDE.md) |
| ally-ai-learn | [`.github/RELEASE_GUIDE.md`](https://github.com/HelloAllyTech/ally-ai-learn/blob/main/.github/RELEASE_GUIDE.md) |
| ally-web | [`.github/RELEASE_GUIDE.md`](https://github.com/HelloAllyTech/ally-web/blob/main/.github/RELEASE_GUIDE.md) — three independently versioned services |

---

## 1. Before you release

- All changes merged to the release branch.
- CI green.
- Code review complete.

## 2. Pick a version

We follow [Semantic Versioning 2.0.0](https://semver.org/): `vMAJOR.MINOR.PATCH`.

| Bump | When | Example |
|---|---|---|
| **MAJOR** | Breaking API change, removed endpoint, major architecture change | `v1.5.3` → `v2.0.0` |
| **MINOR** | New feature or endpoint, backward compatible | `v1.5.3` → `v1.6.0` |
| **PATCH** | Bug fixes, performance, security patches only | `v1.5.3` → `v1.5.4` |

```bash
git tag -l "v*" --sort=-v:refname | head -1   # current version
git log <that-tag>..<release-branch> --oneline   # what's going out
```

No pre-release suffixes — `v1.2.3-rc1` is rejected by the tag validator.

## 3. Trigger the workflow

GitHub Actions → the repo's production-release workflow → **Run workflow** → pick the
release branch, enter the version tag, run.

**You do not create the git tag yourself.** The workflow validates the format, checks the
tag doesn't already exist, confirms it is newer than the latest tag, then creates and
pushes it.

## 4. What the pipeline does

1. **Validate version, create tag** — format, uniqueness, ordering; handles first release.
2. **Prepare environment** — credentials and deployment target configuration.
3. **Run tests** — pipeline fails if any fail.
4. **Build and push the image** — tagged five ways (see below).
5. **Run migrations** — where the service has them, using the versioned image.
6. **Deploy** — update the task definition, wait for stability.
7. **Create a release draft** — changelog generated from commits since the last tag.

### Image tag scheme

For release `v1.2.3`:

| Tag | Purpose | Moves? |
|---|---|---|
| `1.2.3` | exact version | never |
| `1.2` | major.minor | with patch releases |
| `1` | major | with minor and patch releases |
| `latest` | most recent release | every release |
| `{sha}-{run}` | unique build id | never |

## 5. Publish the draft

The workflow leaves a **draft** release. Review it, then publish:

- Check the generated changelog against what you expected to ship.
- Add **Highlights**, and for a major, **Breaking Changes** and a **Migration Guide**.
- Note API changes and any known issues.
- Publish — this notifies watchers.

Then verify the deployment and exercise the critical flows. The per-repo guide has the
exact commands for that service.

---

## Troubleshooting

**"Tag v1.2.3 already exists"** — pick a different version, or delete the tag if it was
created by mistake: `git tag -d v1.2.3 && git push origin :refs/tags/v1.2.3`.

**"Tag v1.2.0 is not newer than the latest tag v1.5.0"** — the validator enforces
monotonic versions so you cannot accidentally ship backwards. Use a higher number.

**"Tag must be in format v{major}.{minor}.{patch}"** — no missing `v`, no two-part
versions, no pre-release suffixes.

**Tests fail** — reproduce locally, fix on the release branch, re-run. If the tag was
already created, delete it first (`git push origin :refs/tags/vX.Y.Z`) or the re-run will
fail validation.

**Build fails** — check the Dockerfile, dependency availability, and that the image
registry exists with the right permissions.

**Deploy fails** — verify the repository variables for the production role, region and
registry are set; confirm the cluster and service exist; read the container logs and
deployment events; check task-definition CPU/memory and IAM permissions.

**Migration fails** — check the migration itself, database connectivity, and user
permissions, then read the migration task's logs. Fix forward and redeploy.

**Rollback** — either select the previous task-definition revision on the service, or
re-run the release workflow with the previous version tag.

---

## Practices worth keeping

- Release regularly and in small batches; a release nobody can read the changelog of is a
  release nobody can debug.
- Group related features; don't let one risky change ride along with ten safe ones.
- Document breaking changes prominently, with the migration steps.
- After publishing: watch logs and error rates, verify the critical flows, tell the team.

---

*See also: [Contributing Guide](guide.md) · [Developer Setup](dev-setup.md) · [Documentation System](docs-system.md)*

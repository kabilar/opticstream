---
title: Releasing
---

# Releasing

Releases are automated with [auto](https://intuit.github.io/auto/), following
the same setup as [dandi-cli](https://github.com/dandi/dandi-cli) and
[dandi-archive](https://github.com/dandi/dandi-archive). Each release:

1. Picks the next version from the labels on the PRs merged since the last tag.
2. Updates `CHANGELOG.md` and pushes a `vX.Y.Z` tag.
3. Creates a GitHub release with the changelog as release notes, and attaches
   the sdist and wheel.
4. Is archived on Zenodo with a DOI, through Zenodo's GitHub integration.
5. Is published to PyPI, once PyPI publishing is switched on (see below).

The package version is not written anywhere in the code: `setuptools-scm`
reads it from the latest git tag at build time.

## Labelling pull requests

Every PR needs one of these labels. The `Check PR labels` workflow fails until
it has one.

| Label | Effect on the next release |
| --- | --- |
| `major` | Major version bump (breaking change) |
| `minor` | Minor version bump (new feature) |
| `patch` | Patch version bump (bug fix) |
| `performance` | Patch bump, listed under performance |
| `internal` | No bump on its own; listed under internal changes |
| `documentation` | No bump on its own; listed under documentation |
| `tests` | No bump on its own; listed under tests |
| `dependencies` | No bump on its own; listed under dependency updates |
| `skip-release` | Doesn't trigger a release; its changes go out with the next one |

The release uses the largest bump among the PRs it contains.

## Cutting a release

Merging a labelled PR does not release on its own: changes accumulate on
`master` until one of these happens.

- **Add the `release` label** to a PR (alongside its semver label). When it
  merges, everything merged since the last tag is released.
- **Run the workflow by hand**: Actions → Release → Run workflow on `master`.
  This releases whatever is pending without needing the `release` label.

Releases only run in `lincbrain/opticstream`; the workflow skips forks.

## First release

The repository has no tags yet, so `auto` starts from `0.0.0`. Merging the
first PR with the `minor` and `release` labels produces `v0.1.0`. PRs merged
before this setup have no labels and count as patch changes, so they don't
change the result. They do appear in the first changelog.

## One-time setup

These steps need repository admin access on `lincbrain/opticstream`, and the
Zenodo step must be done **before** the first release: Zenodo only archives
releases created after the repository is switched on.

1. **Create auto's labels.** With a GitHub token that can manage labels on the
   repository:

    ```bash
    GH_TOKEN=<token> npx auto create-labels
    ```

2. **Keep `master` pushable by Actions.** The release job pushes the
   changelog commit and tag with the workflow's `GITHUB_TOKEN`. If `master`
   gets branch protection or a ruleset, either let GitHub Actions bypass it or
   give the job a token that can push.

3. **Turn on Zenodo.** Sign in to [Zenodo](https://zenodo.org) with GitHub,
   open the GitHub page in your account settings, and flip the switch for
   `lincbrain/opticstream` (organization repositories need the organization to
   grant Zenodo access). Zenodo takes the record's metadata (authors, ORCIDs,
   affiliations, license, keywords) from `CITATION.cff`, which also powers
   GitHub's "Cite this repository" button. Add new contributors there.

   After the first release, copy the Zenodo *concept* DOI (the one that always
   resolves to the latest version) into `CITATION.cff`:

    ```yaml
    identifiers:
      - type: doi
        value: 10.5281/zenodo.XXXXXXX
    ```

4. **PyPI (later).** PyPI rejects packages whose dependencies are git URLs,
   and `linc-convert`, `nifti-zarr` and `psoct-toolbox` are currently pinned to
   git commits. Once each one is published on PyPI:

    1. Replace the git URLs in `pyproject.toml` with version specifiers.
    2. On PyPI, add a [pending trusted publisher](https://docs.pypi.org/trusted-publishers/creating-a-project-through-oidc/)
       (Your account → Publishing) for the project `opticstream`: owner
       `lincbrain`, repository `opticstream`, workflow `release.yml`,
       environment `pypi`. The first publish creates the project.
    3. In the repository, create an environment named `pypi` (Settings →
       Environments) and set the repository variable `PUBLISH_TO_PYPI` to
       `true` (Settings → Secrets and variables → Actions → Variables).

    The next release then publishes to PyPI with no API token. The publish job
    refuses to upload while any git dependency remains.

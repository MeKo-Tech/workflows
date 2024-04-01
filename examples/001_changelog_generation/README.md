# About

The files which are provided with this example can be used to automatically generate changelogs and version bumps from conventional commit messages. 
The workflow triggers for the events which are specified at `.github/workflows/release-pr.yml` (e.g. push to branch `main`) and uses release-please for changelog generation.
[Release-please](https://github.com/googleapis/release-please) creates a PR which
* updates the changelog at `CHANGELOG.md`
* updates the version number at `.release-please-manifest.json`
* updates the version number at supplementary language-specific files (e.g. `package.json`, `go.mod`, `pyproject.toml`,...)
in case "releasable-units" are detected from (unmerged) conventional commit messages. The release-PR is automatically updated as more commits are pushed.

Change `release-please-config.json` to configure changelog generation and version bumping.


## Setup
1. Follow the instructions in the [README](https://github.com/MeKo-Tech/workflows/blob/main/README.md) to enable reusable workflows for your repository
2. Copy `.github/workflows` to your repository
3. Copy `release-please-config.json` to your repository and update the configuration to your needs
    * Update [release-type](https://raw.githubusercontent.com/googleapis/release-please/main/schemas/config.json) to match the type of your project. This setting is required to correctly update supplementary files (e.g. package.json, go.mod, pyproject.toml,...)
    * update `changelog-sections` to match your needs. This configuration describes the mapping between conventional commit messages and changelog sections, as well as how version bumps will be calculated. Any conventional commit which is not `hidden` will result in an increment of the semver patch version, at a minimum.
    * DO NOT CHANGE `include-component-in-tag` and `include-v-in-tag`. These settings define the format of auto-generated git tags and might break subsequent CI/CD workflows (e.g. workflows which check the tag format)
    * A complete configuration reference can be found at the [Docs](https://github.com/googleapis/release-please/blob/main/docs/manifest-releaser.md) and at the [JSON Schema](https://raw.githubusercontent.com/googleapis/release-please/main/schemas/config.json)
4. Copy `.release-please-manifest.json` to your repository. This file stores the current version for your project packages at the given path. The default package path is `"."`.
   Change the version number at this file to match the version in any supplementary files you might have (e.g. package.json)

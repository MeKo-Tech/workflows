# About

This workflow uses the Tags `STAGING` and `PRODUCTION` to deploy source code to distinct deploy branches (in this case: `deployed_staging` and `deployed_production`). 
Third-party app-platforms which allow automatic deployment from branches can be configured to track these branches.

The workflow uses release-please for automatic changelog generation and version bumps, and for the creation and tagging of GitHub releases.
The `deploy` workflow is triggered in case either one of the tags `STAGING` or `PRODUCTION` is pushed. The workflow checks out the repo at the tag and force-pushes the contents to the deploy branches (potentially overwritting any changes at the deploy branches).
As a side-effect, this allows to easily rollback changes by moving the tag to a previous release.

As an additional security guard the workflow fails in case the tags `STAGING` or `PRODUCTION` have been pushed for a git ref which is no valid release (i.e. which does not have a Semver tag or no GitHub Release). The workflow will automatically delete the (invalid) tag ( `STAGING` or `PRODUCTION`) from the remote in this case.

## Setup
1. Follow the setup instructions for `002_release`
2. Copy `.github/workflows/deploy.yml` to your repository

## Usage
1. Push your changes to main and wait for release-please to create or update a release-PR
2. Merge the release-PR once you want to create a new release, and wait for the release-please action to finish (creates PR and release)
3. Update local repository and fetch tags.
4. Update local tag `STAGING` (or `PRODCTION`) and push to remote
```bash
# delete local tag
git tag -d STAGING
# update local tag
git tag STAGING <Version-Number>

# delete tag from remote
git push origin --delete STAGING
# push local tag
git push origin STAGING
```
The tag can be created locally with `git tag STAGING` in which case it will refrence the current commit. It is safer to directly create the tag from the release-tag, e.g. `git tag STAGING 0.0.1`
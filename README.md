# About

This repository holds GitHub Actions workflows which can be reused throughout the organization. 

In principle, all workflows which are implemented at this repo can be used to simplify the creation of custom CI/CD workflows at other repositories. A couple of workflow files have been specified to wrap common DevOps tasks. These workflows should be used to set up DevOps pipeplines whith minimal effort:
| Workflow | Description |
| --- | --- |
| release-pr.yml | Automatically creates release-PRs with version bumps and changelog from conventional commit messages |
| release-build-push.yml | Creates a new release and builds and pushes a Docker Image after merging a release-PR  |
| deploy-branch.yml | Force-pushes a release from a git ref (e.g. a given tag or branch) to a dedicated deploy branch (e.g. `staging`) which can be tracked for automatic deployment with third-party app platforms |
| deploy-docker.yml | Deploys a release from a git ref to a remote host using Docker Compose and SSH |

Check [examples](https://github.com/Primoxo/workflows/tree/main/examples) for further details how to configure complete DevOps workflows

## Using workflows from this repository

### Github Settings
1. Make sure THIS repository can be used for workflows which are specified at other repositories in this organization. Navigate to `repository settings` -> `Actions` -> `General` and select `Accessible from repositories in the organization`.
2. Configure the referencing repository to use workflows from other repos in the organization. For the other repository navigate to `repository settings` -> `Actions` -> `General` and select `Allow all actions and reusable workflows` or `Allow Primoxo actions and reusable workflows`

### Using in a workflow

Workflows from this repository can be called in workflow files with the `uses` keyword:
```yaml	
on:
  # (...)
jobs:
  job_name:
    uses: Primoxo/workflows/.github/workflows/<workflow-file>.yml@<git-ref>
    with:
      # (workflow inputs)
    secrets:
      # (workflow secrets)
```
`<git-ref>` specifies the ref from this repository which is used. This can be a branch name (e.g. `main`), a tag (e.g. `1.0.0`) or a commit hash. 
Any ref other than a commit hash might be floating and should be used with caution. (E.g. changes might be pushed to branch `main` of this repository at any time).

Example workflow file:
```yaml	
name: deploy

on:
  push:
    tags:
      - 'STAGING'
      - 'PRODUCTION'

jobs:
  validate-tag:
    runs-on: ubuntu-latest
    steps:
        # (...)
  create-deploy-branch-production:
    if: ${{ github.event.ref == 'refs/tags/PRODUCTION' }}
    permissions:
      contents: write
    needs:
      - validate-tag
    uses: Primoxo/workflows/.github/workflows/force-push-deploy-branch.yml@main
    with:
      source_ref: ${{ github.event.ref }}
      target_branch: deployed_staging 
```

Check the [official Documenation](https://docs.github.com/de/actions/using-workflows/reusing-workflows#calling-a-reusable-workflow) for further details.

## Basic knowledge and Limitations

* All workflows which are implemented at this repo use [Automatic Token Authentication](https://docs.github.com/en/actions/security-guides/automatic-token-authentication) to connect to the GitHub API and GitHub Container Registry. 
  * Reusable workflows run with the context of the calling workflow and, hence, use the context [github](https://docs.github.com/de/actions/learn-github-actions/contexts#github-context) and `secrets.GITHUB_TOKEN` (for automatic authentication) for that workflow. The automatically generated access token grants access only to the calling (!) repository, i.e. to the repostiry, at which the workflow file is specified, which calls the reusable workflows from this repository.
  * Any events, which are triggered by workflows from this repository (e.g. branch or tag pushed, release created,...) [will not (!) trigger any further workflow runs](https://docs.github.com/en/actions/security-guides/automatic-token-authentication). It is, for example, not possible to automatically create a deploy branch from a workflow, and to add an additional workflow wich is triggered when pushing to the deploy branch.
  In case a use-case like this is required, Automatic Token Authentication must not be used (e.g. to create the deploy branch). Instead a [GitHub App](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/making-authenticated-api-requests-with-a-github-app-in-a-github-actions-workflow) should be added to the repository or organization. Use `actions/create-github-app-token` to generate a token for that app and use the token instead of the default `GITHUB_TOKEN`.
* Environment variables, which are defined at the `env` context at calling worklow will not be passed to the called (reusable) workflow (and vice-versa).
* Permissions must be less-restrictive or equal for the calling workflow. Check the implementation of the reusable workflows for minimal required permissions
* There are some [caveats](https://docs.github.com/de/actions/using-workflows/reusing-workflows#re-running-workflows-and-jobs-with-reusable-workflows) when re-running workflows, which call reusable workflows

## Further Reading

* [Primoxo Docs](https://github.com/Primoxo/docs/tree/main/docs/ci_cd)
* [GitHub-Documentation: Reusing Workflows](https://docs.github.com/de/actions/using-workflows/reusing-workflows)
# workflows

This repository holds reusable GitHub Actions workflows for the organization.

## Usage
### Github Settings
1. Make sure THIS repository can be used for workflows in other repositories. Navigate to repository settings -> Actions -> General and select `Accessible from repositories in the organization`.
2. Allow workflows to be used in other repositories. For the other repository navigate to settings -> Actions -> General and select `Allow all actions and reusable workflows` or `Allow Primoxo actions and reusable workflows`

### Using in a workflow

Check the [official Documenation](https://docs.github.com/de/actions/using-workflows/reusing-workflows#calling-a-reusable-workflow)
Example:
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


## Basic knowledge and Limitations

* reusable workflows run in the context of the calling workflow (at the calling repository) and, hence, use the context [github](https://docs.github.com/de/actions/learn-github-actions/contexts#github-context) and `secrets.GITHUB_TOKEN` (for automatic authentication) for that workflow
* environment variables, which are defined at the `env` context at calling worklow will not be passed to the called (reusable) workflow (and vice-versa)
* permissions must be less-restrictive or equal for the calling workflow. -> Check the definition of the reusable workflow for minimal required permissions to be used for the calling workflow
* there are some [caveats](https://github.com/Primoxo/step-renderer/tree/main/deployment) when re-running workflows, which call reusable workflows

## Further Reading

* [Primoxo Docs](https://github.com/Primoxo/docs/tree/main/docs/ci_cd)
* [GitHub-Documentation: Reusing Workflows](https://docs.github.com/de/actions/using-workflows/reusing-workflows)
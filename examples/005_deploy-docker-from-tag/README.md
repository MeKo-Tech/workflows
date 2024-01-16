# About

This workflow uses the Tags `STAGING` and `PRODUCTION` to deploy released app versions to a remote docker host. 

The workflow uses release-please for automatic changelog generation and version bumps, and for the creation and tagging of GitHub releases.
The `deploy` workflow is triggered in case either one of the tags `STAGING` or `PRODUCTION` is pushed. The workflow checks out the repo at the tag and reads a docker-compose file from the repository. The docker-compose file is used to start or upgrade the application at a remote docker host using SSH.

Application specific configurations are implemented by passing a multiline string with .env file contents to the workflow which is used for deplyoment. The .env file is interpolated when running docker-compose up. The docker-compose file might make use of env var substitution in various ways (e.g. to specify env vars passed to containers, or to specify container labels).

As an additional security guard the workflow fails in case the tags `STAGING` or `PRODUCTION` have been pushed for a git ref which is no valid release (i.e. which does not have a Semver tag, no GitHub Release, or no Docker Image with the same tag). The workflow will automatically delete the (invalid) tag ( `STAGING` or `PRODUCTION`) from the remote in this case.

## Setup
1. Follow the setup instructions for `003_release_build_push`
2. Copy `.github/workflows/deploy.yml` to your repository
3. Modify `.github/workflows/deploy.yml`
    * update the value for the action input `image_name` to match the name of the docker image which is used for your repository
    * update the value for the action input `compose_project_name` to match the name of your docker-compose project (this can be the repo name prefixed with the environment name, for example)
    * update the secret `env_file_data`. This is a multiline strings with environment variable definitions which are substituted when running docker-compose up. The list of required environment variables is determined by the docker-compose file, which might make use of env vars in various ways (e.g. to specify container labels, or to specify env vars passed to containers).
    Use yaml multiline strings to split the app configuration into multiple varaibles and secrets, which can be specified individually for the GitHub repository
4. Add Variables and Secrets to GitHub repository
   * Navigate to `repository settings` -> `Secrets and variables` -> `Actions` 
   * Add the corresponding variable for each variable which is referenced at the workflow file (search for `vars` in the file), respectively
   * Add the corresponding secret for each secret which is referenced at the workflow file (search for `secrets` in the file), respectively


## Usage
Same as for `004_deploy_branch_from_tag`. The workflow will deploy the application to the remote docker host instead of updating a deploy branch
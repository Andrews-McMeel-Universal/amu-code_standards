# Deployment

We use GitHub Actions to automate deployment for new projects. The deployment process varies slightly between AMU's standard branches, please see [Branching Strategy](https://github.com/Andrews-McMeel-Universal/amu-code_standards/tree/production/general/github/branching-strategy) for more information on our branches.

Most projects have their own Action Workflow file and domain for deployments, consult with other team members and the project's README for more information.

If a project does not yet have deployments set up through GitHub Actions, a team member will assist in getting them set up and running, be prepared to share details about any necessary build steps or containers.

## Staging

The "staging" branch is deployed automatically after commits are pushed to that branch, which typically occurs after merging an approved [Pull Request](https://github.com/Andrews-McMeel-Universal/amu-code_standards/tree/production/general/github/pull-requests).

## Production

The "production" branch is deployed automatically after creating a GitHub Release. We use [semantic versioning](https://semver.org/) to tag our releases, like "1.2.3". If applicable, add a link to the Jira Release within the GitHub Release's description, and ensure the release version matches between GitHub and Jira. Some projects will have steps to complete prior to a release, always remember to check that project's README for more information.

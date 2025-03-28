# Bicep Module template repository

Purpose of this repo is to act as a **template** for other upcoming module repos. The real GitHub repo template will be created either from this repo or another one in the future.

The repo contains just a single resource module because of the following benefits:

- More easily trackable code for a limited amount of resources.
- Ability to create GitHub releases per module -> Easier integration with Azure Container Registry (ACR) for publishing modules.

## Required initial updates for all repos generated from this teplate

- Re-create default branch protection rule:
  - Can be imported from [this file](.github\protect-default-branch.json)
- Re-generate secrets with following names:
  - **PAT**: Personal Access Token (PAT) for *release-on-pr* Action
  - **AZURE_SPN_CRED**: Azure SPN credentials for Azure authentication
- Update module name in *publish-to-acr* Action:
  - Replace default value of env **MODULE_NAME** with actual module name.
  - Use **\<module-name\>** convention.
- README cleanup:
  - Remove all template contents.
  - Replace the top heading with module name.
  - [Optional] Add relevant content (or wait for automatically generated content :).

## Repo rules & git workflow

Repo has main branch protection rule configured. This means that Pull Request has to be created to push changes into main branch.

The intended workflow is trunk-based with short-lived feature branches.

### Example workflow of code change

1. Clone the repo (if not already done): `git clone <repo-url>`
2. Create new feature branch: `git checkout -b <branch-prefix>/<branch-name>`
    - Common branch prefixes:
      - **feature**: Adding new feature
      - **update**: Code change without adding new features
      - **fix**: Bugfixes
3. Push new branch into remote origin: `git push -u origin <branch-name>`
4. Make intended code changes.
5. Stage updated files: `git add <updated-file-names>`
6. Commit the changes into feature branch: `git commit -m <commit message>`
7. Create git tag (required for GitHub release to be created): `git tag <tag-name>`
    - Tag names should follow this pattern: `vX.X.X` (e.g., `v1.0.0`)
    - Tag names should also follow [Semantic Versioning](https://semver.org/)
8. Push changes to remote origin: `git push origin <feature-branch-name>`
9. Push new tag to remote origin: `git push origin <tag-name>`
10. Create Pull Request in GitHub and merge changes into main branch.
     - Successfully merged PR will invoke GitHub Actions [described here](#github-actions)
11. Delete branch on GitHub after successful merge.
12. Delete local version of branch: `git branch -d <feature-branch-name>`
13. Remove refs to deleted remote branches: `git fetch -p`

## Repository folder structure

```bash
.github
└── workflows
modules
├── module1
└── module2
main.bicep
README.md
```

- **.github/workflows**: Contains GitHub Actions workflow definitions.
- **modules**: Contains required submodules for the main module (if required).
- **main.bicep**: Contains the module definition.

## GitHub Actions

### Create GitHub release on Pull request

This workflow creates a release after successful PR with *.bicep file included.

Personal Access Token (PAT) has been created for authentication against GitHub and stored in repo secrets. PAT is required to enable subsequent workflow to trigger on new release. This does not work with default GitHub.token and the second workflow wouldn't trigger.

### Publish new module version into ACR

If previous workflow successfully creates a release, this workflow triggers and publishes new module version into configure ACR.

#### Azure & ACR authentication

Entra ID Service Principal has been created and data stored in repo secrets.

This SPN has Azure RBAC for ACR to be able to publish a modules.

[Flexible federated identity credentials](https://learn.microsoft.com/en-us/entra/workload-id/workload-identities-flexible-federated-identity-credentials?tabs=github) could be more appropriate solution in a near future as this feature will be in GA.

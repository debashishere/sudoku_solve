[![Unit tests](https://github.com/agrawal-rohit/Nawvel/actions/workflows/CI_Unit.yml/badge.svg?branch=develop)](https://github.com/agrawal-rohit/Nawvel/actions/workflows/CI_Unit.yml) 
[![Build (Integration tests)](https://github.com/agrawal-rohit/Nawvel/actions/workflows/CI_Integration.yml/badge.svg?branch=develop)](https://github.com/agrawal-rohit/Nawvel/actions/workflows/CI_Integration.yml)

# Nawvel
The repository stores the monolith codebase for the current Nawvel web application

## Folder structure
- `client` - Frontend made with NextJS
- `server` - Backend made with NodeJS + Express + MongoDB

## Starting the development environment
1. Install [Docker](https://www.docker.com/get-started) on your system.
2. Run `docker-compose up --build` from the terminal
3. Go to `www.localhost:3000` for the accessing the frontend
4. Go to `www.localhost:8000` for accesing the backend

## Running tests
### Frontend tests
`docker-compose run client /bin/sh -c "npm run test"`

### Backend tests
`docker-compose run server /bin/sh -c "npm run test"`

## Coding coventions and practices
We follow the [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) workflow as closely as possible. This page showcases common development scenarios and how to deal with them from a branching point of view.

- [Branches Overview](#branches-overview)
- [Use Cases](#use-cases)
  - [Develop a new feature](#develop-a-new-feature)
  - [Develop multiple features in parallel](#develop-multiple-features-in-parallel)
  - [Create and deploy a release](#create-and-deploy-a-release)
  - [Change in plan, pull a feature from a release](#change-in-plan-pull-a-feature-from-a-release)
  - [Change request](#change-request)
  - [Production hot fix](#production-hot-fix)

## Branches Overview

![Release Deployment workflow](https://raw.githubusercontent.com/mobify/branching-strategy/master/images/release-deployment.png)

| Branch        | Protected? | Base Branch | Description    |
| :-------------|:-----------|:------------|:---------------|
| `master`      | YES        | N/A         | What is live in production (**stable**).<br/>A pull request is required to merge code into `master`. |
| `develop`     | YES        | `master`    | The latest state of development (**unstable**). |

| `feature/some-feature`       | NO         | `develop`   | Cutting-edge features (**unstable**). These branches are used for any maintenance features / active development. |
| `release/X.Y.Z` | NO      | `develop`    | A temporary release branch that follows the [semver](http://semver.org/) versioning.<br/>A pull request is required to merge code into any `release/vX.Y.Z` branch. |
| `hotfix/*`    | NO         | `master`    | These are bug fixes against production.<br/>This is used because develop might have moved on from the last published state.<br/>Remember to merge this back into develop and any release branches. |

## Use Cases

### Develop a new feature

![Feature branch](https://raw.githubusercontent.com/mobify/branching-strategy/master/images/release-deployment.png)

1. Make sure your `develop` branch is up-to-date

1. Create a feature branch based off of `develop`. (*Feature branches should follow the following naming convention: `feature/<JIRA Issue ID>-Additional-description`*)
   ```
   $ git checkout develop
   $ git checkout -b feature/NP-123-new-documentation
   $ git push --set-upstream feature/NP-123-new-documentation
   ```

1. Develop the code for the new feature and commit. Push your changes often. This allows others to see your changes and suggest improvements/changes as well as provides a safety net should your hard drive crash. (*Commits made to the feature branches should follow the following naming convention: `[<JIRA Issue ID>] What changes were made`*)
    ```
    $ ... make changes
    $ git add -A .
    $ git commit -m "[NP-123] Add new documentation files"

    $ (... make more changes)

    $ git add -A .
    $ git commit -m "[NP-123] Fix some spelling errors"
    $ git push
    ```

1. Open a pull request with the following branch settings:
   * Base: `develop`
   * Compare: `NP-123-new-documentation`
1. When the pull request was reviewed, merge and close it and delete the
   `NP-123-new-documentation` branch.

### Develop multiple features in parallel

Each developer follows the above [Develop a new feature](#develop-a-new-feature) process in parallel, so features can be worked upon independently.

### Create and deploy a release

![Create and deploy a release](https://raw.githubusercontent.com/mobify/branching-strategy/master/images/release-release.png)

1. Merge `master` into `develop` to ensure the new release will contain the
   latest production code. This reduces the chance of a merge conflict during
   the release.
   ```
   $ git checkout develop
   $ git merge master
   ```

1. Create a new `release/vX.Y.Z` release branch off of `develop`.
   ```
   $ git checkout -b release/vX.Y.Z
   $ git push --set-upstream release/vX.Y.Z
   ```

1. Stabilize the release by adding direct changes to the `release/vX.Y.Z` branch
   ```
   $ git checkout release/vX.Y.Z

   (... do work)

   $ git commit -m "Adjust label to align with button"
   $ git push
   ```

1. When the code is ready to release, open a pull request with the following branch settings:
   * Base: `master`
   * Compare: `release/vX.Y.Z`
   
   (Paste the Release Checklist into the PR body. Each project should define a release checklist.)

1. At some point in the checklist you will merge the release branch into `master`.
   You can do this by using the "Merge pull request" button on the release PR.

1. Now you are ready to create the actual release. Draft a new release with the following settings:
   * Tag version: `vX.Y.Z`
   * Target: `master`
   * Release title: `Release vX.Y.Z`
   * Description: Include a high-level list of things changed in this release.
   Click `Publish release`.

1. Merge the `release/vX.Y.Z` into `develop`.

    ```
    $ git checkout develop
    $ git merge release/vX.Y.Z
    $ git push
    ```

1. Finish off the tasks in the release checklist. Once everything is done, close the release PR.

1. Delete the release branch on Github.

### Production hot fix
*In rare situations you may need to get a fix into production fast! Use this workflow to push a hotfix to production when you can't spare the time to follow the standard 'Develop a feature' workflow.*

![Fast Production hotfix](https://raw.githubusercontent.com/mobify/branching-strategy/master/images/continuous-hotfix.png)

A production hotfix is very similar to a full-scale release except that you do
your work in a branch taken directly off of `master`. Hotfixes are useful in cases
where you want to patch a bug in a released version, but `develop` has unreleased
code in it already.

1. Create a hot fix branch based off of `master`.

   ```
   $ git checkout master
   $ git checkout -b hotfix/documentation-broken-links
   $ git push --set-upstream origin hotfix/documentation-broken-links
   ```

1. Add a test case to validate the bug, fix the bug, and commit.

   ```
   (... add test, fix bug, verify)

   $ git add -A .
   $ git commit -m "Fix broken links"
   $ git push
   ```

1. Open a pull request with the following branch settings:
   * Base: `master`
   * Compare: `hotfix/documentation-broken-links`
   
   (Paste your release checklist into the PR and work through the PR to get the hotfix into production.)

1. At some point in the checklist you will merge the hotfix branch into `master`.
  You can do this by using the "Merge pull request" button on the release PR.

1. Now that the hotfix code is in `master` you are ready to create the actual
   release. Navigate to the project page on Github and draft a new release with
   the following settings:
   * Tag version: `vX.Y.Z`
   * Target: `master`
   * Release title: `Release vX.Y.Z (hotfix)`
   * Description: Include a high-level list of things changed in this release.

1. Click `Publish release`.

  *Note: Hotfix releases _are_ actual releases. You should bump at least the
  patch part of the version when releasing a hotfix and so even hotfixes go
  through the process of creating a release like this.*

1. Merge the `hotfix/documentation-broken-links` into `develop`.

   ```
   $ git checkout develop
   $ git merge hotfix/documentation-broken-links
   $ git push
   ```

1. Finish off the tasks in the release checklist. Once everything is done, close
   the hotfix PR.

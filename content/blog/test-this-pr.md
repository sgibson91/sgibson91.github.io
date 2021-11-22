---
title: "How I automated authorised cloud deployments from Pull Requests with GitHub Actions"
date: 2021-11-22T19:35:00Z
draft: False
---

> This blog was originally posted on the Jupyter blog: <https://blog.jupyter.org/how-i-automated-authorised-cloud-deployments-from-pull-requests-with-github-actions-13f890538e32>

I recently did some work on the mybinder.org deployment infrastructure to solve a problem with testing Pull Requests before deployment.
It had not been possible to test Pull Requests on our staging deployment because our automated workflows don't have access to secrets.
This resulted in my writing the [`test-this-pr` action](https://github.com/sgibson91/test-this-pr-action) and this blog is a retrospective of what I learned over that process.

## What was the problem we were trying to solve?

It is generally considered best practice to have a staging environment when running complex applications or platforms that have a large userbase, such as mybinder.org.
This provides developers a space to test infrastructure changes safely in the knowledge that users won't be affected should anything go wrong.
The deployment configuration for mybinder.org is managed in a GitHub repository and deploys are handled in a CI/CD pipeline run on GitHub Actions.

The mybinder.org operating team's usual workflow will be very familiar to anyone working as part of a distributed development team: fork the main repository, do some development (fix a bug or implement a feature), and then open a Pull Request (PR) back to the default branch of the main repository.
The difficulty arises when we want to test the PR on our staging infrastructure before merging and deploying for real.

Various secrets are required to make a deployment, even to our staging environment.
These include a `git-crypt` secret that decrypts certain sensitive files and credentials to push Docker images to the repositories connected to our clusters, among others.
These are stored as encrypted repository secrets so they are available to GitHub Actions when our CD pipeline runs.
However by default, repository secrets are not made available to PRs from forks which introduces a problem for the workflow I described above.

So to summarize our problem:

- Contributors often make changes to the code in their _fork_ of the repository
- Testing any change requires deploying to an actively-running staging cluster
- The secrets needed to make those deploys are only available to the original repository, not to a forked repository
- So we needed a way to trigger a test deployment on changes _from a forked repository_ but that ran _in the origin repository_

## What solutions did we already have in place?

We had one team practice, and one technical solution to address this issue, both of which didn't quite do what we wanted.

One solution was a team deployment practice of "just merge the PR anyway and file a revert PR if things go wrong".
Our CI/CD pipeline is broken into two jobs: deploy to staging infrastructure, and a matrix deployment to each production cluster in [the BinderHub federation](https://mybinder-sre.readthedocs.io/en/latest/operation_guide/federation.html).
The production jobs require the staging job to pass first before they are triggered.
This means that if the deploy to staging fails, the changes will never propagate to our production clusters.

Another solution we had available was a `test-staging` label we could apply to open PRs.
This triggered the deploy to staging job without a merge. However, this only worked for PRs opened from branches _within the origin repository_, and as mentioned above, this is not a common workflow for maintainers. Instead they open PRs from forked repositories, which do not have the proper secrets to deploy to staging.

## What other solutions were available?

We briefly considered [ok-to-test](https://github.com/imjohnbo/ok-to-test) but were put off by the requirement for a GitHub App.
However as we will learn, it was probably a mistake to dismiss this so quickly!

## What did I end up implementing?

And so I built my first GitHub Action!

The [`test-this-pr` action](https://github.com/sgibson91/test-this-pr-action) allows you to trigger a GitHub Action _from the origin repository_ when an authorised user adds a comment to the PR of a forked repository.

`test-this-pr` is triggered when a user leaves a comment on the PR you'd like to test, but only if the commenter has the appropriate permissions on the repository, e.g. MEMBER.
This protects us from malicious code being run automatically on our staging deployment and ensures that project maintainers have vetted the code in some way before triggering the test.

When triggered, `test-this-pr` uses a small Python script that queries the GitHub API to do the following:

- It creates a new branch in the origin repository that uses the merge reference from the PR.
- This triggers the test suite in the new branch, because we have configured the origin repository to automatically run deployment tests on branches that match a particular name pattern (in our case, `test-this-pr/*`, see below)
- Adds comments on the original PR with a link to the newly-running CI test logs.

This script is wrapped up into a Dockerfile, which allows GitHub Actions to run the action using it's docker runner.

{{< figure src="https://user-images.githubusercontent.com/44771837/142857688-e2d23b26-f8d7-4140-bdc5-318fe3d2e4a0.png" alt="An example of the `test-this-pr` workflow in action." caption="An example of the `test-this-pr` workflow in action." height=400 >}}

## Extra configuration needed to use this action

There were a few extra pieces I configured that couldn't (and shouldn't!) be implemented in the `test-this-pr` action, but contribute to the overall workflow.

I also [created a CI job that polls the running staging job](https://github.com/jupyterhub/mybinder.org-deploy/blob/5442082cbb408f889f433792c1abf4ede02e67ec/.github/workflows/cd.yml#L318-L341).
Upon completion, it posts a second comment to the original PR stating whether the test passed or failed.
This had to be separate from the `test-this-pr` action since this job needs to run within the same workflow run in order to access the status of the staging job.

Once the pass/fail status of the staging job has been reported, the branch created by `test-this-pr` is then deleted.
We decided this was the best path forward so that the original PR remained the "single source of truth" and maintainers wouldn't be confused by extra branches in the repository.

## For reference if you'd like to understand this implementation

- [Here is our configuration to trigger `test-this-pr`](https://github.com/jupyterhub/mybinder.org-deploy/blob/master/.github/workflows/test-this-pr.yml#)
- [Here we configure our deployment action to run on branches with `test-this-pr/*`](https://github.com/jupyterhub/mybinder.org-deploy/blob/master/.github/workflows/cd.yml#L17-L21)
- [Here is the CI job that polls the running staging job and posts a comment with its status before deleting the branch](https://github.com/jupyterhub/mybinder.org-deploy/blob/5442082cbb408f889f433792c1abf4ede02e67ec/.github/workflows/cd.yml#L318-L341)

## What did I learn along the way?

### There is A LOT you can do with the GitHub REST API

I had originally configured the Python script to shell out to `git` for tasks like creating the new branch and pulling the PR merge ref, and this worked fine when I ran Python locally.
However when running the docker container, I struggled to authorise `git` correctly, even when passing my Personal Access Token.
And so I made the decision to switch to the [GitHub REST API](https://docs.github.com/en/rest).

While a lot of GitHub native features can be accessed and manipulated via the API, some `git`-specific actions can be achieved as well via the [git database endpoint](https://docs.github.com/en/rest/reference/git), which is the endpoint to use to create branches, commits, and much more.
This section of the API might require a deeper dive into the inner-workings of `git`, but can unlock a lot of automation potential once you have your mental model!

Switching to the GitHub REST API from the `git` command line meant that I no longer needed to install `git` into my docker container or locally clone the repository to carry out the actions, making my image much smaller.

### Triggering GitHub Actions from other Actions

You can't trigger another GitHub Actions workflow from an event that was authorised with the `GITHUB_TOKEN` from within a previous workflow.
This is why the comments left by the `test-this-pr` workflows in the Binder repository come from my account as I had to provide a Personal Access Token for the staging job to be correctly triggered.

The downside of this is that I'm automatically subscribed to notifications for any Binder PR where someone is using `test-this-pr`.
It's not the worst situation in the world as I have email filters set up, but also not ideal.

This is also why `ok-to-test` recommend setting up a GitHub App  for authorisation, so you don't have to sacrifice your own account and all the notifications that come with it!

However, this "actions triggering actions" scenario could be smoothed out by using the upcoming [Reusable Workflows](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows) beta.

### Building Actions with Dockerfiles

Providing a Dockerfile with a custom Action will allow you to write your Action in whatever language you like.
However, it is one of the slowest ways to implement custom Actions because GitHub's runner will build the image every time the workflow is triggered.

## What would I do differently now?

### Setup a GitHub App for authorisation

If only to spare my inbox from the notifications! :joy:

### Move away from a Python implementation

The `test-this-pr` action ultimately ended up being a handful of calls to the GitHub REST API.
As opposed to maintaining a Python script and Dockerfile, I would probably reimplement this using the [`github-script` action](https://github.com/marketplace/actions/github-script), which is a JavaScript wrapper for the GitHub API that can be directly run from GitHub Actions.
This will be less code to maintain and the `test-this-pr` action itself could then be reworked as a [composite action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action) which would build faster than the current Docker-based implementation.

## Closing Remarks

While there is a lot I would change about `test-this-pr`, including fundamental changes that would make it look radically different, I needed to go on this developmental journey to gain the better understanding of GitHub Actions that I have now.
And hopefully by sharing that journey, other folk can learn from it!

Also the mybinder operating team love using this feature and it's adding value to our development workflow, so I don't see a need to dive back in and start changing things again right now :slightly_smiling_face:

{{< figure src="https://i.imgur.com/a6b8d1b.jpg" alt="Members of the mybinder operating team discussing test-this-pr in gitter" caption="Members of the mybinder operating team discussing test-this-pr in gitter" height=400 >}}

You can read the source code for [`test-this-pr`](https://github.com/sgibson91/test-this-pr-action) and [mybinder.org's deployment workflow](https://github.com/jupyterhub/mybinder.org-deploy/blob/master/.github/workflows/cd.yml) in full.

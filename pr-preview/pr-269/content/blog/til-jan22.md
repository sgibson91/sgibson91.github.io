---
title: "Things I've Learned: January 2022"
date: 2022-01-31
description: "An Unordered Collection of Things I Learn over a Month"
tags: ["learning"]
series: ["Things I've Learned"]
---

- Nested build matrices are not (yet?) supported in GitHub Actions, but you can explicitly define a set of matrix parameters using YAML array syntax.
  See an example [here](https://github.com/sgibson91/testing-gh-actions/blob/3a0fec6d59c933646e6c6b673e37cadf1dafb3a2/.github/workflows/includes-matrix-with-list.yaml).
- A pattern I often use to update my working branch with the default branch is:
  ```bash
  git checkout main
  git pull  # Add `upstream main` if appropriate
  git checkout my_branch
  git merge main
  ```
  Mostly this is fine, but occasionally merge conflicts happen.
  If I know I want to keep a specific version of a conflicting file from one of the branches (as opposed to finding a non-conflicting combination), [`--theirs/--ours`](https://nitaym.github.io/ourstheirs/) can be used.
  ```
  git checkout --ours conflicting_filename  # To keep the version from the current branch
  git checkout --theirs conflicting_filename  # To keep the file from the incoming branch
  ```
  Note that behaviour can change depending on which branch is checked out and whether a merge or rebase is being performed, so I recommend to double-check online!
- That the second `---` in YAML delimits as if what follows it is another YAML file.
  This can cause issues for command line YAML parsers like `yq` and made pulling the front matter from my Markdown files a little trickier than expected!
- How to automatically tweet out new blog posts when they are merged into `main`
- How to use a GitHub App to generate tokens in GitHub Action workflows.
  The tokens can then be used to securely workaround the fact that GitHub Action workflows can't be triggered by events that were authorised by the `GITHUB_TOKEN` in another workflow.
  There is a nice write-up that helped me [here](https://github.com/peter-evans/create-pull-request/blob/main/docs/concepts-guidelines.md#authenticating-with-github-app-generated-tokens).

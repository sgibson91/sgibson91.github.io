---
title: "Things I've Learned: January 2022"
date: 2022-01-31
description: "An Unordered Collection of Things I Learn over a Month"
tags: ["learning"]
series: ["Things I've Learned"]
---

- Nested build matrices are not (yet?) supported in GitHub Actions, but you can explicitly define a set of matrix parameters using YAML array syntax.
  See an example [here](https://github.com/sgibson91/testing-gh-actions/blob/main/.github/workflows/includes-matrix-with-list.yaml).
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

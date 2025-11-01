---
title: "Tech update: Multiple JupyterHubs, multiple clusters, one repository"
summary: "How we optimised and parallelised deployments of multiple JupyterHubs across multiple clusters in GitHub."
date: 2022-05-12T11:51:52+01:00
tags:
  - ci-cd
  - devops
  - github-actions
draft: false
disableComments: true
---

{{< notice info >}}This blog was [originally posted by 2i2c](https://2i2c.org/blog/2022/ci-cd-improvements/) and represents a project Sarah lead and developed.{{< /notice >}}

2i2c manages the configuration and deployment of multiple Kubernetes clusters and JupyterHubs from [a single open infrastructure repository](https://github.com/2i2c-org/infrastructure).
This is a challenging problem, as it requires us to centralize information about a number of _independent_ cloud services, and deploy them in an efficient and reliable manner.
Our initial attempt at this had a number of inefficiencies, and we recently completed an overhaul of its configuration and deployment infrastructure.

This post is a short description of what we did and the benefit that it had.
It covers the technical details and provides links to more information about our deployment setup.
We hope that it helps other organizations make similar improvements to their own infrastructure.

## Our problem

2i2c's problem is similar to that of many large organizations that have independent sub-communities within them.
We must centralize the operation and configuration of JupyterHubs in order to boost our efficiency in developing and operating them, but must also treat these hubs _independently_ because their user communities are not necessarily related, and because we want communities to [be able to replicate their infrastructure on their own](https://2i2c.org/right-to-replicate).

A year ago, we built the first version of our deployment infrastructure at [github.com/2i2c-org/infrastructure](https://github.com/2i2c-org/infrastructure).
Over the last year of operation, we identified a number of major shortcomings:

- Within a Kubernetes cluster, we deployed hubs sequentially, not in parallel. This grew out of a common practice of [Canary deployments](https://sre.google/workbook/canarying-releases/) that allowed us to test changes on a **staging hub** before rolling them out to a **production hub**.
- We used a single configuration file for all hubs within a cluster, which led to confusion and difficulty in identifying a hub-specific configuration.
- Moreover, any change to a hub within a cluster caused a re-deploy of _all hubs on that cluster_. This is because we did not know whether a given change touched cluster-wide configuration or hub-specific configuration.

## Our goal

So, we spent several weeks discussing a plan to resolve these major problems - here were our goals:

- We should be able to **upgrade a specific hub** alone, by inspecting which configuration files have been added or modified.
- **Production hubs should be upgraded in parallel** when they are effectively run independently.
- We should **use staging hubs as "canary" deployments** and not continue upgrading production hubs if the staging hub fails.

## An overview of our changes

To accomplish this, we needed to identify which hub required an upgrade based on file additions/modifications.
This took a lot of discussion and iteration on design, and so we share it below in the hopes that it is helpful to others!

### Improvements to our code and structure

We made a few major changes to [the infrastructure repository](https://github.com/2i2c-org/infrastructure) to facilitate the deployment logic described above.
Here are the major changes we implemented:

- We separated each hub's configuration into its own file, or set of files. For example, [here is 2i2c's `staging` hub configuration](https://github.com/2i2c-org/infrastructure/blob/master/config/clusters/2i2c/staging.values.yaml).
- We created a separate `cluster.yaml` file that holds the canonical list of hubs deployed to that cluster and the configuration file(s) associated with each one. For example, [here is 2i2c's GKE cluster configuration](https://github.com/2i2c-org/infrastructure/blob/master/config/clusters/2i2c/cluster.yaml), which contains a reference to the previously mentioned [staging hub](https://github.com/2i2c-org/infrastructure/blob/master/config/clusters/2i2c/cluster.yaml#L14-L26).
- We updated [our deployer module](https://github.com/2i2c-org/infrastructure/tree/master/deployer) to do the following things:
  - Inspect the list of files modified in a Pull Request.
  - From this list, calculate the name of a hub that required an upgrade, and the name of its respective cluster.
  - Trigger a GitHub Actions workflow that deploys changes in parallel for each cluster/hub pair.

In addition to these structural and code changes, we also developed new GitHub Actions workflows that control the entire process.

### A GitHub Actions workflow for upgrading our JupyterHubs

We defined a new GitHub Actions workflow that carries out the logic described above.
These are all defined in [this `deploy-hubs.yaml` configuration file](https://github.com/2i2c-org/infrastructure/blob/master/.github/workflows/deploy-hubs.yaml).
Here are the major jobs in this workflow, and what each does:

1. `generate-jobs`: Generate a list of clusters/hubs that must be upgraded, given the files that are changed in a Pull Request.
   - Evaluate an input list of added/modified files in a PR
   - Decide if the added/modified files warrant an upgrade of a hub
   - Generate a list of hubs and clusters that require upgrades, and some extra details:
     - Does the support chart that is deployed to the cluster also need an upgrade?
     - Does a staging hub on this cluster require an upgrade?

   This produced two outputs to be used in subsequent steps:

   - A **human-readable table** including information on _why_ a given deployment requires an upgrade (using the excellent [Rich library](https://github.com/Textualize/rich)).
   - **JSON outputs** that can be interpreted by GitHub Actions as sets of matrix jobs to run.

   {{< figure src="https://2i2c.org/blog/2022/ci-cd-improvements/images/staging-hub-matrix_hu7a1bb3fb06e3f581f944c2d267a10ff9_107479_1200x1200_fit_q75_h2_lanczos_3.webp" title="Our staging and support hub job matrix tells GitHub Actions to deploy staging and support upgrades that act as canaries and stop production deploys if they fail.">}}
2. `upgrade-support-and-staging`: Update the support and staging Helm charts on each cluster. These are "shared infrastructure" Helm charts that control services that are shared across all hubs.
   - Accepts the JSON list described above to determine what to do next
   - Parallelises over clusters
   - Upgrades the support chart of each if required
   - Upgrades a staging hub for the cluster if required (for canary deployments, this is always required if at least one production hub is to be upgraded on the cluster)

3. `filter-generate-jobs`: Allows us to treat the support / staging hubs as canary deployments for all the production hubs on a cluster.
   - If a staging/support hub deploy fails, removes any jobs for the corresponding cluster.
   - Allows production deploys to continue on _other clusters_.

   {{< figure src="https://2i2c.org/blog/2022/ci-cd-improvements/images/prod-hub-matrix_huc0588a8559e954f04dc74eb98419e619_250512_1200x1200_fit_q75_h2_lanczos_3.webp" title="Our production hub job matrix tells GitHub Actions which hubs to update with new changes. These are triggered if a cluster's staging/support job does not fail." >}}

4. `upgrade-prod-hubs`: Deploy updates to each production hub.
   - Accepts the JSON list described above to determine what to do next
   - Parallelises over each production hub that requires an upgrade
   - Deploy the relevant changes to that hub

## Concluding Remarks

We think that this is a nice balance of infrastructure complexity and flexibility.
It allows us to separate the configuration of each hub and cluster, which makes each more maintainable by us, and is more aligned with a community's [Right to Replicate](https://2i2c.org/right-to-replicate) their infrastructure.
It allows us to remove the interdependence of deploy jobs that do not _need_ to be dependent, which makes our deploys more efficient.
Finally, it allows us to make _targeted deploys_ more effectively, which reduces the amount of toil and unnecessary waiting associated with each change. (It also [reduces our carbon footprint by reducing unnecessary GitHub Action time](https://github.blog/2021-04-22-environmental-sustainability-github/)).

We hope that this is a useful resource for others to follow if they also maintain JupyterHubs for multiple communities.
If you have any ideas of how we could further improve this infrastructure, please reach out on GitHub!
If you know of a community that would like 2i2c to [manage a hub for your community](https://2i2c.org/service/), please [send us an email](mailto:hello@2i2c.org).

{{< notice info >}}**Acknowledgements**
The infrastructure described in this post was developed by [the 2i2c engineering team](https://2i2c.org/team), and this post was edited by [Chris Holdgraf](https://2i2c.org/author/chris-holdgraf).{{< /notice>}}

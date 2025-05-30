---
title: "HelmUpgradeBot"
description: "A Bot to manage BinderHubs"
date: 2019-07-05T00:00:00Z
tags: ["reproducibility", "binderhub", "automation"]
draft: true
---


While managing [BinderHubs](https://binderhub.readthedocs.io/en/latest/), I found that there were a lot of things I had to deal with manually and, inevitably, I failed to keep on top of them.
Most notably, the two biggest problems I faced were _i)_ keeping my BinderHub up-to-date with the latest software released by the Binder team, and _ii)_ keeping the size of the image registry manageable and within budget.

Hence I developed [HelmUpgradeBot](https://github.com/HelmUpgradeBot)!

## Keeping software up-to-date

The first problem I wanted to solve was the manual updates I had to perform to keep my BinderHub up-to-date with the software the Binder team releases, packaged as a [Helm Chart](https://jupyterhub.github.io/helm-chart/).
Thankfully, this was a problem that had already been solved for [mybinder.org](https://mybinder.org)!

[Chris Hench](https://github.com/henchc) had already written [henchbot](https://github.com/henchbot/mybinder.org-upgrades) to deploy updates to mybinder.org, _and_ had written [a blog post](https://blog.jupyter.org/automating-mybinder-org-dependency-upgrades-in-10-steps-bb5e38542059) describing the process of creating it.
All I needed to do was tweak the code for my specific set-up - specifically, interacting with Azure resources.

[Check out the code for Helm Chart upgrades.](https://github.com/HelmUpgradeBot/hub23-deploy-upgrades/)

## Interacting with Azure

The trick with the Helm Chart upgrades was running the bot from a place that could interact with the Azure resources I had set up to support my BinderHub - namely, the [Key Vault](https://azure.microsoft.com/en-gb/services/key-vault/), which was where I chose to store my Personal Access Token rather than parse it as an environment variable (as in the henchbot example).

I needed to run this from a server that could scrape the Helm Chart page at regular intervals.
The solution that made most sense was to create a Virtual Machine in Azure and assign it a [Managed System Identity](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview#how-a-system-assigned-managed-identity-works-with-an-azure-vm) with enough permissions to read from the Key Vault.
The bot could then login to Azure using the identity of the VM it was running on, and retrieve the PAT from the Key Vault.
Once I had perfected this section of the code, it was just a matter of following Chris's blog post to code the interactions with GitHub (for example, create a fork and pull request) and then deploy the code as a [cron job](https://crontab.guru/) on the Azure VM to perform a daily check for updates.

## Cleaning up artefacts

One thing that BinderHubs produce is a _lot_ of Docker images.
If you're using a private Docker registry, such as [Azure Container Registry](https://azure.microsoft.com/en-gb/services/container-registry/), then the size (and hence the cost!) can soar very quickly.
So I wanted to apply what I'd learned deploying the Helm Chart upgrades to automatically clean out my Docker registry at regular intervals.

[Check out the code for cleaning a Docker registry.](https://github.com/HelmUpgradeBot/DockerCleanUp/)

I already had a basis for how to deploy such a code:

:heavy_check_mark: Programmatically interacting with Azure login

:heavy_check_mark: Virtual Machines with access to cloud resources

:heavy_check_mark: Writing cron expressions

Next, I began to investigate how to [programmatically interact with the Docker registry](https://docs.microsoft.com/en-us/cli/azure/acr?view=azure-cli-latest) itself.

I wanted to sort the images both by age and size since large images will obviously fill the repository up more quickly, and older images are less likely to be actively used or have been rebuilt under a different tag.
I achieved this by scraping the manifests of the repositories and images within the registry and creating a dataframe to sort the images.

I also created an "aggressive" mode which would activate when the total size of the registry exceeds a pre-set, tunable limit.
This would then delete the largest images in order to bring the size under the prescribed limit.

Currently, this bot runs monthly to check the size of the registry isn't growing too large.

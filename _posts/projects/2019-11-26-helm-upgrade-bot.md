---
layout: post
title: HelmUpgradeBot
categories: projects
excerpt: "A Bot to manage BinderHubs"
tags: [reproducibility, binderhub, automation]
date: 2019-11-26
author:
  name: Sarah Gibson
  picture: /images/profile_pic.jpg
image:
  path: /images/helmupgradebot-lg.png
  thumbnail: /images/helmupgradebot-400x200.png
  caption: "Image created by [Chris Holdgraf](https://github.com/choldgraf)"
---

While managing [BinderHubs](https://binderhub.readthedocs.io/en/latest/), I found that there were a lot of things I had to deal with manually and, inevitably, I failed to keep on top of them. Most notably, the two biggest problems I faced were _i)_ keeping my BinderHub up-to-date with the latest software released by the Binder team, and _ii)_ keeping the size of the image registry manageable and within budget.

Hence I developed [HelmUpgradeBot](https://github.com/HelmUpgradeBot)!

## Keeping software up-to-date

The first problem I wanted to solve was the manual updates I had to perform to keep my BinderHub up-to-date with the software the Binder team releases, packaged as a [Helm Chart](https://jupyterhub.github.io/helm-chart/). Thankfully, this was a problem that had already been solved for [mybinder.org](https://mybinder.org)!

[Chris Hench](https://github.com/henchc) had already written [henchbot](https://github.com/henchbot/mybinder.org-upgrades) to deploy updates to mybinder.org and had written [a blog post](https://blog.jupyter.org/automating-mybinder-org-dependency-upgrades-in-10-steps-bb5e38542059) describing the process of creating it. All I needed to do was tweak the code for my specific set-up - specifically, interacting with Azure resources.

[Check out the code for Helm Chart upgrades.](https://github.com/HelmUpgradeBot/hub23-deploy-upgrades/)

## Interacting with Azure

The trick with the Helm Chart upgrades was running the bot from a place that could interact with the Azure resources I had set up to support my BinderHub - namely, the [Key Vault](https://azure.microsoft.com/en-gb/services/key-vault/), which was where I chose to store my Personal Access Token rather than parse it as an environment variable.

I needed to run this from a server that could scrape the Helm Chart page at regular intervals. The solution that made most sense was to create a Virtual Machine in Azure and assign it a [Managed System Identity](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview#how-a-system-assigned-managed-identity-works-with-an-azure-vm) with enough permissions to read from the Key Vault. The bot could then login to Azure using the identity of the VM it was running on, and retreive the PAT from the Key Vault.

---
title: 'CNCF Golden kubestronaut Prep Lab'
date: '2026-11-25'
# weight: 1
# aliases: ["/first"]
tags: ["k8s", "kubestronaut"]
author: "Mariyan Dimitrov"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Notes on a personal hands on lab for CNCF k8s exams."
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---
I am setting up a local lab in preparation for CNCF exams that is quite
helpful. I am building it from scratch and I am doing lots of things wrong,
which is the best way to learn (and humbling as well).

I have two repos:

<https://github.com/merkata/cncf-lab> - this bootstraps a kind cluster and
installs Argo CD
<https://github.com/merkata/gitops-lab-configs> - this holds the
resources Argo CD will manage

The end goal is to cover topics that are present in OTCA and PCA
(observability), CGOA and CAPA (gitops), ICA and CCA (networking), these are
the exams I have purchased with my budget for this year. For now I am leaving
out ICA and CCA (Istio and Cilium) and playing mostly with the Argo ecosystem.

The setup will deploy an App of Apps with sync waves, you have base (gitea,
harbor, sealed secrets, silo S3 storage), cicd (workflows, events, rollouts),
observability (loki, grafana, prometheus, opentelemetry) and apps (a gofr
framework toy app because it provides telemetry out of the box).

The current goal is that a push to Gitea repo will trigger a webhook into an
Argo event, this will trigger an Argo workflow to build and push an image to
Harbor, the image change will be picked up by Argo Image Updater and will
trigger an Argo rollout of the application.

Current progress is that the apps are running and I have a workflow that builds
a container and stores it in local S3 (btw a minimal goFr framework app is like
800MiB...), so the Argo CD and Workflows parts are covered (together with Gitea
integration) and I am hooking up Events and Rollouts.

I can only recommend going through such an exercise, as I've hit a bunch of
stuff:

- minio is deprecated as local S3 store, need to use pgsty/silo
- kaniko is deprecated now so using buildkit
- default artifactory not configured in workflows helm chart, needs additional
config map
- helm resource cannot be overridden with "parameter: {}" as the "{}" get
merged in the default values (instead of overwriting) and nothing changes
- app of apps needs a custom health check with recent Argo CD to wait for an
app before proceeding with the next one
- Argo CD will happily stay out of sync when you pass in an invalid key (I had
"value" instead of plural "values" for helm chart values and I stared at it for
too long)
- newer jaeger charts have deprecated the v1 traces path and the Grafana jaeger
plugin does not work, jaeger have published their own
- workflows helm chart needs service account adjustment so it works
- buildkit rootless needs apparmor profile set to unconfined and a no-sandbox
env var in order to work

These are just some of the most outstanding things and I am taking more notes.

Most of the setup is automated, however right now I am not bootstrapping the
app repo in Gitea and I'm creating a new repo on cluster recreation. For an e2e
automated lab, the repo and at least one image in harbor needs to be present
(so the rollout of the app can kick in).

This is still a rough work in progress, so things can break or be broken, but
I get to figure out why and try fixing it.

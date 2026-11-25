---
title: 'CNCF Lab Automations'
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
description: "Some automations on the cncf lab to avoid manual bootstrapping."
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
Some updates on the [cncf-lab](https://www.merkata.fun/posts/cncf-golden-prep-lab/), I completed the Argo based tasks and implemented
a couple of features backed by [pi](pi.dev) and deepseek-v4.1-flash-1017 model. Pi is my
go-to coding agent (next to a Hetzner hosted hermes agent) and I am trying out
deepseek's flash model right now, total bang for the buck. It helped with
setting the correct harbor registry endpoint (it goes through the harbor svc
and not the harbor-registry svc which is used internally), exposed the running
apps in a app.127.0.0.1.nip.io fashion (I steered it towards using Gateway API
which comes up in CKA/CKAD/CKS with Cilium backed HTTPRoutes), let it create
some bootstrapping (create the Gitea repo and webhook for the Argo Events
webhook eventsource, create the Harbor project for the app image registry). The
model is super capable and the harness around it makes it amazing for
troubleshooting, it reaches to a plethora of tools to troubleshoot what's
happening - I've seen it go through a helm chart, look at the source code,
create and apply a fix against a live cluster and verify it.

What I did was stitch together the final pieces - I had an Argo
WorkflowTemplate that was already in the cluster, so I created the Argo Events
components (the default EventBus and the apps EventSource and Sensor). The
EventSource expects a webhook to hit it at port 12000 on path /app, this will
put the event on the EventBus. A sensor will listen on the EventBus for
specific events, catch the webhook event and trigger a Workflow. The workflow
will reference the WorkflowTemplate and pass the body of the webhook (Gitea
will include the ref that I capture and append to the created image, so
"refs/tags/v1.0.0" will result in v1.0.0 being appended to the built image that
is stored in Harbor).

The app deployment is done via Argo Rollouts that is stored in the Gitea repo
next to the app code. An Argo app is syncing that deployment and Argo Image
Updater updates the image in the repo when it sees a newer semver being
published in Harbor. Whenever there is a new tag created in the Gitea repo, the
webhook fires:

Gitea repo tag -> webhook event -> workflow triggered -> image built and pushed
to harbor -> new image detected and updated in git -> Argo CD syncs app -> Argo
canary rollout triggered

This is still scratching the surface of what can be done, I am putting TODO
items in the cncf-lab repo. Some things to be implemented soonish:

Metrics based promotions for Argo rollouts - goFr has built-in telemetry, so
prometheus metrics can be hooked into the rollout strategy to rollout or
rollback.

Cilium policies to secure ns-to-ns communication - right now all services are
accessible from anywhere, whereas Cilium allows for fine grained control based
on namespaces, service slices and what not.

Istio is still not part of the cluster.

I haven't really configured proper o11y in the cluster yet, most of the charts
expose metrics, these can be scraped via ServiceMonitors, results can be
dashboarded. I am not gathering and corelating logs/metrics/traces from the
goFr app yet.

It feels like magic that such a stack can be created on a laptop and it can be
troubleshooted with an LLM almost instantly.

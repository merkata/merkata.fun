---
title: 'Preparing for the Kubestronaut'
date: '2026-11-13'
# weight: 1
# aliases: ["/first"]
tags: ["kubestronaut", "kubernetes"]
author: "Mariyan Dimitrov"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "How I prepared for the Kubestronaut certification exams."
canonicalURL: "https://merkata.fun/posts/preparing-for-the-kubestronaut"
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
## Notes on passing the Kubestronaut exams

These are some tips that helped me get the Kubestronaut. Note that at some
point I held CKA, CKAD and CKS certifications, so it is not my first time
dealing with CNCF certifications and the specific exams. I have focused on
practicing with labs and mock exams and seeing what has changed from the last
time I took the certifications rather than starting from scratch with
documentation and courses.

### resources used

* KodeKloud mock exams - KodeKloud have a [Kubestronaut learning
path](https://learn.kodekloud.com/learn/learning-paths/kubestronaut) with video
courses and mock exams.

* KillerKoda scenarios - from the creator of killer.sh,
[https://killercoda.com/](https://killercoda.com/) has scenarios for the
practical exams - you work on a scenario and it is instantly verified. You also
get hints and the full solution if you are stuck.

* Killer.sh simulation exams - with registering for an exam, you get two
simulation exams with [killer.sh](https://killer.sh/), which is invaluable. You
get detailed walkthroughs for every question on the simulation. The first time
I was certifying I was reading through every solution. This time I focused on
what I got wrong.

_Note that the KodeKloud learning path features the video courses by Mumshad
Mannambeth which I definitely recommend. They helped me a ton the first time I
was going through Kubernetes certifications._

If I wasn't pressured for time, I would have spent more time reading through
[the official docs](https://kubernetes.io/docs/home/), they are superbly
written and you also need to know your way around during the exams. I had the
chance to get some doc PRs merged when preparing for the exams for the first
time and that felt really rewarding.

### practice over theory

I learn best by doing, so I focused heavily on practicing. I did every mock
exam, practiced every scenario, did the simulation exams and reviewed what I
did right and what I did wrong. I can say that my muscle memory kicked in after
a couple of hours spent practicing.

### tips and time savers

These are the commands that helped me the most during the exam and I also included what I wish I did (like reading a task until the end before attempting to solve it).

#### k == kubectl

The current exams let you connect to multiple clusters, so creating aliases is
a waste of time, however on each and every cluster, "k" is aliased to
"kubectl". The command examples use "k" instead of "kubectl" and I recommend
using "k" during the exam as well.

#### `k explain --recursive <kind> | less`

This is my top tip and something I always consult besides the official
documentation in the browser during the exam. Whenever you need to find a field
for an object, this is the fastest way. Say you have a task of disabling the
auto mounting of service account tokens - run `kubectl explain --recursive pod`
and pipe it to `less`, hit the `/` ("forward slash") to search for let's say
"automount" and you get to `automountServiceAccountToken`.

A note on less - `/` search is regex by default, so doing a
`/[sS]ervice[aA]ccount` search will get you "serviceAccount", "ServiceAccount",
"serviceaccount" and "ServiceAccount" matches. `n` searches forward, `N`
searches back.

#### `k explain <kind.sub>`

If you've used the `k explain --recursive trick` and found your field, you can
get detailed information of it by passing the full path to the field. Using our
automountServiceAccountToken example from above, doing a `k explain
pod.spec.automountServiceAccountToken` will give you a detailed description of
the field. I regularly consult things like `k explain
pod.spec.containers.volumeMounts` and `k explain pod.spec.containers.envFrom`
for syntax and getting the correct field name.

#### `k <verb> --help`

You get a whole lot of help for free from the `--help` switch of a whole bunch
of kubectl subcommands. Need to expose a service quick? Run `k expose --help`
and check the examples. Need to scale a deployment - `k scale --help`.

#### `k create`

The create subcommand can create a wide range of objects (and hitting `tab`
twice after `k create` should show you exactly which). Combine that with a
`--help` and you get examples of creating the object that you can readily use.
A `k create deployment mydeploy --image=nginx:latest` will create a basic
deployment that you can later adjust as the task requires. I usually dry-run
create a resource, save it to a file and look in the official documentation and
via `k explain --recursive` what to adjust.

#### `k run`

Some resources are not created via `create`, one such example is the `k run`
that you use to create pods. I will not go into detail, you know by know to do
a `k run --help` and see for yourself.

#### `k autoscale`

The horizontal pod autoscaler is created via `k autoscale` and not via `k
create`. The examples should cover most of your exam needs.

#### `k expose`

Same for services, you `k expose` a resource and it gets a service. Note that
`k expose --name` sets the name for your service.

#### `k --dry-run=client -o yaml > kind.yaml`

When I have to create a new resource, I always do a `k
create|run|autoscale|expose` followed by a `--dry-run=client -o yaml` and pipe
it to a file. That way I can make adjustments and do a `k apply -f <file>` when
I am happy.

#### `k get <kind> -o yaml > kind.yaml`

Same goes for already running resources, I tend not to do a `k edit` of the
resource or `k patch`. Normally I use a `k get <kind> -o yaml` and pipe it to a
file, like `k get deploy mydeploy -o yaml > mydeploy.yaml`. That way I can do
the required adjustments. In some cases, you are permitted/required to delete
resources, so working with the file instead of editing the resource directly
will save you time, because some fields cannot be altered once the resource is
running. Make your changes, `k delete -f mydeploy.yaml && k apply -f
mydeploy.yaml` and you're done.

#### keep another tab open

I always keep a second tab open in the terminal app, on the first tab I am
doing the task (like add an env variable to a deployment from a config map) and
on the second I research (like running `k explain --recursive` or plain `k
explain`). For most tasks I find myself in a vim session editing fields in a
resource file and in the second tab I am consulting the biuld in documentation
via `k explain`.

Note that since the exams now happen on multiple clusters that you ssh into,
you can also go back to previous clusters in a second tab. Particularly useful
for troubleshooting misconfigurations where you can consult a healthy running
cluster and for resources you've already encountered (like ingresses and
networking policies).

#### read the question until the end

I have a tendency to start solving a task right away and I must remind myself
to read everything until the end. This can give you some additional
information, it can save you time - I remember `k get`ting a deployment from a
namespace to a file and then I started adjusting it, only to see at the end of
the task that there is already a deployment file available. Even small delays
can add up, so make sure to get the full context.

#### copy paste is your friend

On the exam, a lot of text in the task description can be copied with a simple
click. From the hostname you ssh into to the namespace that you should be using
and the resource name you should use. Do yourself a favor and copy paste what
can be copy pasted - it saves you time and eliminates nasty subtle bugs you get
when typing.

#### find the missing bits in the documentation

There are things easier done in the terminal rather than reading the full
documentation - a `k create deploy` with some basic switches will get you up to
speed quickly and you don't need to go to the Deployments kubernetes docs page.
Other things are just not readily available via `kubectl` subcommands and you
need to know the most used ones.

Search "pv" - [PV](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) - has examples on StorageClass, PersistentVolume
and PersistentVolumeClaim.

Search "ingress" - [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) - starts off with a minimal Ingress that you can use.

Search "netpol" - [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) - networking policies examples.

Note that you will find helpful links to the documentation with every task you
need to solve, do not ignore these! It is still better to know where the
missing resource bits are hidden, but the linked documentation will certainly
be helpful.

### anecdotal things that might've helped

Not related to tech, but these things might've helped as well, so I am making a
note of these as well.

I was on a pretty high-carb diet while studying, I figured I would need the
extra fuel. Somehow I managed to even lose a bit of weight while doing
high-carb.

I made sure to have a good amount of sleep, aimed for 8 hours, get to bed and
out at the same time. I moved my dinner time earlier in the day, cut caffeine
intake 12 hours before bed.

I supplemented creatine during the day and before bed a mix of magnesium
(l-threonate and bisglycinate) with l-theanine, taurine and glycine.

I made sure to rest between learning sessions and also took walks regularly. I
didn't exercise much while learning, but I made the effort to move and stay
active.

### KCNA

Basic exam, I didn't find it hard and if you fail the first time, it will be
doable the second time. If you do a couple of mock exams you are good to go.

### KCSA

Same thing as the KCNA, this one is a bit trickier because it had questions for
Compliance and Threat Modeling Frameworks. Study guides like [Infrasec
KCSA](https://devopscube.com/kcsa-exam-study-guide/) and [devopscube
KCSA](https://devopscube.com/kcsa-exam-study-guide/) will fill in the blanks.

### CKA

I always start with CKA when it is time to renew my Kubernetes certifications.
It is a great mix of fixing issues, working on the command line, creating some
resources. Both CKAD and CKS benefit from this.

For the CKA, I did the mock exams and the simulation exams on killer.sh.

### CKAD

This is the second time I take the CKAD after CKA and if you manage CKA you
should be very comfortable with the CKAD. You mostly create resources and by
now you should be very good at this.

For the CKAD, I did the mock exams and one simulation exam on killer.sh.

### CKS

The CKS was the surprise. Last time I took it it was the new exam, only CKA and
CKAD existed back then and once a beta CKS exam came out, I took it. It used to
be demanding, but now even more so. It covered a bunch of topics, it had
breadth and depth I didn't expect, and I was pleasantly surprised. It is a mix
of admin topics, use of security tools, fixing misconfigurations, all of these
next to securing your cluster components and workload.

For the CKS, I did all KillerKoda scenarios and one simulation on killer.sh.
The exam still felt demanding. Normally the killer.sh simulations are harder
than the actual exam (and this holds true for CKA and CKAD in my opinion), but
the CKS exam was a fun challenge.

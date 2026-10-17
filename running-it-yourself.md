# Running It Yourself

**Scope:** the realistic ways to get a cluster of
your own — four local tools compared on what they
actually are, the managed clouds in one paragraph,
and the tool you'll meet next.

## Local clusters: four honest options

All four give you a real, disposable Kubernetes on
one machine. They differ in what the "nodes" are
and what the setup is *for*.

| Tool | What it is | Choose it when |
|---|---|---|
| **kind** | Clusters as Docker containers ("Kubernetes in Docker"), from the Kubernetes project itself | You already live in Docker; you want throwaway clusters, several at once, or CI-style runs. The worked example targets it |
| **minikube** | The long-standing all-rounder: VM or container drivers, addons (dashboard, ingress) one flag away | You want the most guided local experience and the most knobs, and don't mind the weight |
| **k3s** | A real, lightweight distribution from Rancher/SUSE: one binary, modest appetite, runs on a Raspberry Pi | The cluster is the point, not a simulation: home labs, small servers, edge boxes. Also fine as a learning cluster on a spare machine |
| **microk8s** | Canonical's snap-packaged distribution, batteries included | You're on Ubuntu and want a maintained, opinionated install with addons as switches |

Differences that actually bite: kind's nodes are
containers, so it's the fastest to create and the
least like real machines; k3s and microk8s install
a *standing* cluster on the host, closer to
production shape (and they keep running until
stopped). For learning this guide's material, any
of the four is correct; pick by what you already
have — Docker → kind, a spare Linux box → k3s.

A resource note, without invented numbers: a local
cluster idles comfortably on a modest laptop, but
the control plane plus your workloads share the
machine; give the cluster a couple of GB of RAM
headroom and expect fans when you scale things up.

## Managed clusters

GKE (Google), EKS (Amazon), and AKS (Microsoft)
rent you the control plane and sell you nodes.
Creating one is a short console or CLI exercise;
everything in this guide then works unchanged,
plus the cloud pieces Chapter 2 flagged — real
`LoadBalancer` services, disks behind
StorageClasses. For a team, this is the default;
for learning, it's paying rent to practice — the
local tools exist precisely so you don't have to.

## The next tool: Helm

Sooner than you expect, installing real software
(a database chart, an ingress controller) will
mean a folder of manifests with your values woven
through them. **Helm** is the package manager for
that: a *chart* is a parameterized bundle of
manifests; you supply values, it renders and
installs. You don't need it for anything in this
guide, and writing your own manifests first is the
right order — Helm packages other people's YAML,
and you now know enough YAML to read what it
generates, which is the part that matters.

## The standing advice

Match the tool to the stakes. Practice on kind or
minikube, where every mistake costs a delete.
Run a home cluster on k3s when there's a second
machine and a reason. Rent a managed cluster when
downtime has a price. And keep the Chapter 1
question taped to all of it: if one machine and
Compose would do, they still would.

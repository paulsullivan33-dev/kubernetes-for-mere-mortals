# Kubernetes for Mere Mortals

**Kubernetes** is a system for running containers
across a cluster of machines: you describe what
should be running, and it works to make reality
match — starting containers, restarting the ones
that die, spreading them across machines, and
rolling out new versions without downtime.

It is also famously more machinery than most
workloads need. This guide explains the ideas in
plain language, runs one small example end to end,
and keeps saying the unfashionable part out loud:
for a single machine, Docker — or Docker Compose —
is usually the right answer, and Kubernetes earns
its complexity only when there are several machines,
several services, or several people sharing them.

It assumes you know what a container is (the Docker
guide in this series covers that) and are comfortable
in a terminal. No cluster experience is assumed.

## What this guide covers

- [**What Kubernetes is**](what-kubernetes-is.md) —
  orchestration, desired state, and when you
  honestly don't need it
- [**The pieces**](the-pieces.md) — clusters, nodes,
  and the control plane, part by part
- [**Pods**](pods.md) — the smallest unit: what's in
  one, and why you rarely create one directly
- [**Manifests and kubectl**](manifests-and-kubectl.md) —
  the YAML grammar, and the declarative way of
  working
- [**Deployments**](deployments.md) — keeping N
  copies alive, scaling, rolling updates, rollbacks
- [**Services and networking**](services-and-networking.md) —
  stable addresses for unstable pods, and the three
  Service types
- [**ConfigMaps and Secrets**](configmaps-and-secrets.md) —
  configuration outside the image — and why a
  Secret is not encrypted
- [**Storage**](storage.md) — volumes, requests for
  storage, and why stateful is harder
- [**A worked example**](worked-example.md) — one
  small web app: apply it, reach it, scale it,
  update it, break a pod and watch it heal
- [**Troubleshooting**](troubleshooting.md) — the
  diagnostic loop and the failure states you'll
  actually meet
- [**Running it yourself**](running-it-yourself.md) —
  kind, minikube, k3s, microk8s, and the managed
  clouds, compared honestly
- [**Glossary**](glossary.md) — the terms, collected
  in one place

## What it doesn't cover

Writing your own controllers or operators, service
meshes, multi-cluster setups, cluster security
hardening beyond the warnings given, and Windows
containers. Helm — the package manager you'll meet
next — is introduced, not taught.

## How to use this guide

One boundary, stated plainly, because it differs
from most of this series: **the worked example was
not run against a live cluster for this guide.**
The machine the guide was built on is a locked-down
sandbox whose kernel cannot create the bridge or
VXLAN network devices every local cluster tool
needs, which delegates no CPU or memory cgroup
controllers (k3s stops at startup with
`failed to find cpu cgroup (v2)`), and which mounts
`/etc/hosts` read-only (minikube's installer stops
there). Docker, k3s (including its rootless mode),
and minikube were each tried and each stopped on
the sandbox, not on Kubernetes. Cluster operations
are therefore marked **(documented)**: written from
the official Kubernetes documentation and checked
for consistency, but presented as steps for *your*
cluster, not as runs quoted from ours.

What *was* verified for real, on that machine:

- **Every manifest in `examples/` is valid.**
  kubeconform v0.6.7, in strict mode, against the
  Kubernetes 1.31.0 schemas: 4 resources, 4 valid,
  0 errors.
- **kubectl itself ran** (client v1.31.14). The
  generated manifests shown in the Manifests
  chapter are real `kubectl create --dry-run=client
  -o yaml` output, and the base64 demonstration in
  the Secrets chapter was run, not typed from
  memory.
- The failure states in Troubleshooting are the
  documented meanings of those states, with the
  checks an operator runs in that order.

If you have any ordinary laptop or spare machine,
nothing in the sandbox's list of problems applies
to you: the worked example is a ten-minute run on
the local cluster of your choice.

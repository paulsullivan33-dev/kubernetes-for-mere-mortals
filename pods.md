# Pods

**Scope:** the smallest thing Kubernetes runs —
what a pod actually is, why it exists as a concept
at all, and why you will almost never create one
by itself.

## A pod is a wrapper around containers

Kubernetes does not schedule containers. It
schedules **pods**: a pod is one or more containers
that are always placed on the **same node** and
share three things:

- **One network identity** — one IP address, one
  set of ports. Containers in a pod reach each
  other at `localhost`.
- **Shared storage** — volumes mounted into the
  pod are visible to all its containers.
- **One lifecycle** — created, started, and (with
  rare exceptions) stopped together.

The everyday case is one container per pod, and
"pod" and "container" get used interchangeably.
Multi-container pods are for helpers that must
live *with* a main container: a log shipper reading
its files, a proxy in front of it (the "sidecar"
pattern). If two containers don't need to share
a network and a lifecycle, they belong in separate
pods where the scheduler can place them freely.

## Why you rarely create one directly

A pod created by itself is a mortal thing: if its
node dies, the pod is **gone** — nothing recreates
it, because nothing was keeping count. Pods are
also not updated in place; changing one means
deleting and recreating it.

So in practice you declare pods inside a
controller — almost always a **Deployment**
(Chapter 5) — which owns a set of identical pod
*copies*, replaces the dead, and rolls out changes:

```
Deployment  (says: 3 copies of this pod, always)
   └─ ReplicaSet  (the counting mechanism)
        ├─ Pod   ── one container, one node
        ├─ Pod
        └─ Pod
```

You will read pods constantly and write them
directly almost never. Knowing that saves a lot of
early confusion: a pod's name with a random-looking
suffix (`hello-web-7d9c4b5f6-x2v1a`) is the mark of
a pod that has owners.

## Pod lifecycle, honestly

A pod moves through phases: **Pending** (accepted,
not yet running — being scheduled or pulling its
image), **Running**, **Succeeded** / **Failed**
(finished, for run-to-completion work), and
**Unknown** (the node stopped answering). Within
Running, containers can crash and be restarted by
the kubelet — a pod can be "Running" while its
container is on its fifteenth restart, which is
why Chapter 10 reads restarts and logs, not just
phases. Deleting a pod gives its containers a
grace period (30 seconds by default) to shut down
before they're killed.

## Talking to pods

(Documented — these need a cluster.) The three
verbs you'll use daily:

```
kubectl get pods                 # list, with status and restarts
kubectl logs <pod-name>          # what the container printed
kubectl exec -it <pod-name> -- sh   # a shell inside, if the image has one
```

`kubectl logs` is the pod-level equivalent of
reading a service's output: the container's stdout
and stderr, captured for as long as the pod exists.
When a container keeps crashing, `kubectl logs
<pod> --previous` reads the *last* container's
output — usually the one with the actual error.

# Deployments

**Scope:** the object you'll use most — how a
Deployment keeps a chosen number of identical pods
alive, changes their version safely, and retreats
when a release goes wrong.

## What a Deployment actually manages

A Deployment owns pods through a middleman: it
creates a **ReplicaSet** whose only job is counting
("the spec says 3 pods with these labels; I see 2;
create 1"). You almost never write a ReplicaSet
yourself, but you'll see them in `kubectl get all`
output, and knowing the chain explains most
Deployment behavior:

```
Deployment ──creates──▶ ReplicaSet ──creates──▶ Pods
   ▲                        │
   └── new ReplicaSet ──────┘   on every template change
       (old one kept, scaled to 0, for rollback)
```

Change anything in the Deployment's pod `template`
— image, settings, labels — and a *new* ReplicaSet
appears and gradually takes over. The old one is
kept, emptied, as your way back.

## Scaling

Scaling is the desired-state idea at its plainest:
`replicas: 5` in the manifest, or, imperatively
(documented — needs a cluster):

```
kubectl scale deployment hello-web --replicas=5
```

Pods are created or removed until reality matches.
Which pods die on scale-down is the ReplicaSet's
choice (youngest first, roughly) — a reason the
workloads Deployments suit best are **stateless**
copies that share nothing and hold nothing unique.
Chapter 8 is about everything that breaks that
assumption.

## Rolling updates and rollbacks

Change the image and the Deployment doesn't swap
pods wholesale; it **rolls**: start some new pods,
and as they report ready, retire old ones, so the
service never drops to zero. Two knobs in
`spec.strategy` govern the overlap:

| Setting | Meaning |
|---|---|
| `maxSurge` | How many *extra* pods may exist during the roll (default 25%) |
| `maxUnavailable` | How many of the desired count may be down mid-roll (default 25%) |

The workflow (documented):

```
kubectl set image deployment/hello-web web=nginx:1.27-alpine
kubectl rollout status deployment/hello-web     # watch it complete
kubectl rollout history deployment/hello-web    # numbered revisions
kubectl rollout undo deployment/hello-web       # back one revision
```

If the new pods never become ready — bad image,
crashing app — the roll *stalls* rather than
completing, and the old pods keep serving. That
stall is a feature: `rollout undo` returns you to
a state the system has kept warm. What it doesn't
do is notice a new version that's healthy but
*wrong*; Kubernetes checks readiness, not sense.

## Honest limits

A Deployment guarantees **count and template**,
nothing more. It doesn't order pod starts, doesn't
give pods stable names or addresses, and treats
every copy as interchangeable — exactly right for
stateless web and API tiers, exactly wrong for a
database primary. Kubernetes has a separate object
(StatefulSet) for that world; know it exists, and
know this guide deliberately stops before it.

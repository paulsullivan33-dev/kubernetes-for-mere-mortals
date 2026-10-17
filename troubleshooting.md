# Troubleshooting

**Scope:** the one diagnostic loop that answers
most questions, and the half-dozen failure states
that account for nearly every stuck pod a beginner
meets — what each means and the first check for it.

## The loop

Almost every investigation is these four commands,
in this order, before anything cleverer:

```
kubectl get pods                       # what state, how many restarts?
kubectl describe pod <name>            # events, at the bottom: the cluster's own account
kubectl logs <name>                    # the app's account
kubectl logs <name> --previous         # the crashed container's account
kubectl get events --sort-by=.lastTimestamp   # the namespace's recent history
```

`get` says *what*, `describe` says *why the cluster
thinks so*, `logs` says *what the program said*.
Skipping `describe` is the classic beginner waste
of an hour: the Events section at its bottom is
where scheduling failures, pull failures, and
missing-config errors are simply written down.

## The states, decoded

| State | What it means | First checks |
|---|---|---|
| **Pending** | No node has taken the pod | Resources: did it ask for more CPU/RAM than any node has? `describe` shows `FailedScheduling` and why |
| **ImagePullBackOff** / **ErrImagePull** | The image can't be fetched; backoff = it's retrying, slower each time | Typo'd name or tag (the classic); private registry without credentials; no network to the registry |
| **CrashLoopBackOff** | The container starts, exits, restarts, repeat | `logs --previous`: the app is telling you. Usual causes: bad config, missing file, a dependency it can't reach |
| **CreateContainerConfigError** | A referenced ConfigMap or Secret (or key in one) doesn't exist | Did you apply the ConfigMap first? Does the key name match exactly? (The worked example, applied out of order, lands here on purpose-adjacent) |
| **Running, but 0/1 ready** | Alive, failing its readiness probe | Probe path/port wrong, or the app is up but not actually serving yet |
| **Service answers nothing** | No endpoints behind the Service | `kubectl get endpoints <svc>`: empty means the selector doesn't match any pod's labels — compare them character by character |
| **Completed** | Not an error: a run-to-finish container that finished | Only a problem if you expected a server |

Two habits worth more than any row: read the
*newest* event first (sort helps; stale events from
an old failure linger), and change **one** thing
between checks — the states are informative exactly
because they're specific.

## When it's the cluster, not you

Pods Pending on *every* node, DNS names failing
cluster-wide (`hello-web` unresolvable from other
pods), or images that pulled fine yesterday: suspect
the platform layer — a node out of disk (evictions
follow), the cluster DNS pods themselves unwell
(`kubectl get pods -n kube-system`), or, on a local
cluster, the host machine simply out of memory.
`kubectl describe node <name>` lists the pressure
conditions the node itself is reporting. Knowing
which layer you're standing in — your object, the
node, the control plane — is most of troubleshooting;
the loop above is how you find out.

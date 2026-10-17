# A Worked Example

**Scope:** one small web app, taken through the
whole mortal lifecycle — install, reach it, scale
it, update it, kill a pod and watch the system heal.
All cluster steps are **(documented)**: they are
written for your cluster, per the README's note.
The manifests themselves ship in `examples/` and
were schema-verified (kubeconform, strict, 1.31).

## The app

`hello-web`: the `nginx:alpine` web server, serving
a home page that lives in a ConfigMap instead of
the image (Chapter 7), run by a Deployment at 2
copies, exposed by a NodePort Service on 30080.
Four small files:

```
examples/configmap.yaml     the page itself, as data
examples/deployment.yaml    2 pods, the page mounted in
examples/service.yaml       NodePort 30080 → port 80
examples/secret.yaml        a specimen for Chapter 7; unused here
```

## Step 0 — a cluster to stand on

Any local cluster from Chapter 11 works. With kind,
one extra detail: to reach a NodePort from your
machine, the kind cluster must map the port when
created. Save this as `kind-config.yaml`
(documented — kind's documented config format):

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30080
        hostPort: 30080
```

Then: `kind create cluster --config kind-config.yaml`.
(On minikube, `minikube start` is the whole step,
and its NodePorts are reachable at `minikube ip`.)

## Step 1 — apply the three files

```
kubectl apply -f examples/configmap.yaml
kubectl apply -f examples/deployment.yaml
kubectl apply -f examples/service.yaml
kubectl get pods
```

Order matters exactly once: the ConfigMap must
exist before pods that mount it can start (apply
the Deployment first and its pods wait in an error
state until the ConfigMap appears — Troubleshooting
covers that state). `kubectl get pods` should
settle at two pods, both `Running`, readiness
`1/1` — names with the Deployment's suffix pattern
from Chapter 3.

**Check:** `kubectl describe deployment hello-web`
shows *desired* 2 and *available* 2.

## Step 2 — reach it

From inside the cluster's world, the Service is
just a name (Chapter 6). From your machine, with
the kind port mapping above:

```
curl http://localhost:30080/
```

Expected: the two lines from the ConfigMap —
`Hello from a ConfigMap.` and the sentence after
it. That page exists in **no image**: the stock
`nginx:alpine` is serving content the cluster
handed it at start. Repeat the curl; answers may
come from either pod, indistinguishably — the
Service is doing its one job.

## Step 3 — scale it

```
kubectl scale deployment hello-web --replicas=4
kubectl get pods -w        # watch two more arrive; Ctrl-C to stop
```

Four copies, no ceremony: you edited a count and
reconciliation closed the gap. (In file-first
working you'd change `replicas:` in the manifest
and re-apply — try that too, back to 2.)

## Step 4 — roll out an update

```
kubectl set image deployment/hello-web web=nginx:1.27-alpine
kubectl rollout status deployment/hello-web
```

New pods (new ReplicaSet) start, report ready, and
old pods retire, a few at a time per Chapter 5's
strategy. `kubectl get pods` mid-roll shows both
generations coexisting. `kubectl rollout undo
deployment/hello-web` walks it back — do it once,
just to have felt the safety net.

## Step 5 — kill a pod, watch it heal

```
kubectl delete pod <one of the pod names>
kubectl get pods
```

A replacement is already being created before you
look: the ReplicaSet counted 1 where 2 were
desired and closed the gap. The Service never
noticed — its endpoints simply point at the living.
This thirty-second experiment is the entire
self-healing claim of Chapter 1, made visible.

## Step 6 — clean up

```
kubectl delete -f examples/service.yaml \
            -f examples/deployment.yaml \
            -f examples/configmap.yaml
```

Deleting the files' objects removes everything the
example created; the cluster itself (kind's, say)
goes with `kind delete cluster`.

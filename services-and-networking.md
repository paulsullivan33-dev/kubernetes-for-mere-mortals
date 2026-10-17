# Services and Networking

**Scope:** how anything reaches your pods when pod
addresses are disposable — the Service object, its
three types, and the names things call each other
by inside a cluster.

## The problem: pods are cattle with random addresses

Every pod gets its own IP, and pods are replaced
constantly — crashes, updates, rescheduling. Any
system that hardcoded pod IPs would break hourly.
A **Service** is the fixed point: a stable virtual
IP and DNS name that forwards to whichever pods
currently match its **selector** (labels again —
Chapter 4's connective tissue at work).

```
clients ──▶ Service "hello-web" (one stable address)
                ├──▶ pod 10.42.0.11
                ├──▶ pod 10.42.0.12
                └──▶ pod 10.42.0.13   (today's copies;
                                       tomorrow's differ)
```

kube-proxy (Chapter 2) programs each node's
network rules so traffic to the Service address is
handed to a live pod behind it, spread roughly
evenly. The pods behind a Service are its
**endpoints**; a Service with zero endpoints is a
Service whose selector matches nothing — a classic
silent failure (Chapter 10).

## The three types you'll meet

| Type | Reachable from | Use it for |
|---|---|---|
| `ClusterIP` (default) | Inside the cluster only | The normal case: services calling services |
| `NodePort` | Outside, via `<any node IP>:<port 30000–32767>` | Quick exposure; what the worked example uses |
| `LoadBalancer` | Outside, via a cloud load balancer the provider creates | Production exposure on managed clouds; on local clusters it just stays "pending" |

This guide's example Service, verified against the
1.31 schema like the other manifests:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-web
spec:
  type: NodePort
  selector:
    app: hello-web          # ← must match the pods' labels
  ports:
    - name: http
      port: 80              # the Service's own port
      targetPort: 80        # the container's port
      nodePort: 30080       # the door on every node
```

Three ports, three meanings — most Service
confusion is mixing them up. `port` is what
clients of the Service use; `targetPort` is where
the container actually listens; `nodePort` is the
external door (only for NodePort/LoadBalancer).

## Names: DNS inside the cluster

The cluster runs its own DNS (CoreDNS), and every
Service gets a name: `hello-web` from anywhere in
its namespace, `hello-web.default` (adding the
namespace) from anywhere in the cluster — the full
form is `hello-web.default.svc.cluster.local`.
So one service reaches another with no addresses
anywhere: `http://hello-web/` in a config file is
a complete, durable instruction. (If you enjoyed
the DNS guide in this series: this is DNS doing
application plumbing — a name per Service, answers
maintained by the cluster itself.)

## Ingress: the front door, conceptually

NodePorts and per-service load balancers don't
scale as a way in. **Ingress** is a separate object
that routes outside HTTP traffic by hostname and
path — `shop.example.com` to one Service,
`api.example.com` to another — through one
**Ingress controller** (a program like
ingress-nginx or Traefik that you install; the
Ingress object alone does nothing without one).
That's all this guide says about it: know the
shape — one door, rules, many Services behind it —
and learn the details when you have real traffic
to route.

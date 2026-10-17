# The Pieces

**Scope:** the cast of a cluster — what each part
does, in one sentence each, and which parts you
can rent instead of running.

## The shape of a cluster

A **cluster** is a set of machines (physical or
virtual) called **nodes**, plus a **control plane**
that runs the cluster itself. Nodes come in two
roles, often the same machines in small setups:

- **Control-plane nodes** run the cluster's brain.
- **Worker nodes** run your containers.

Every action — yours, the system's, a container's —
goes through one component: the **API server**.
There is no side door. `kubectl`, the web dashboard,
and the control plane's own parts all talk to the
same API, which is why "Kubernetes" is less a
program than a *place where objects live*.

## The control plane

| Part | Job, in one sentence |
|---|---|
| API server | The front door: validates and stores every object you create |
| etcd | The database: the cluster's one source of truth (desired *and* recorded state) |
| Scheduler | Decides which node each new pod lands on |
| Controller manager | Runs the reconciliation loops: "Deployment says 3, I see 2, start 1" |
| Cloud controller manager | The seam where a cloud provider plugs in (load balancers, disks) — absent in local clusters |

Two of these deserve a second look. **etcd** is
why control-plane backups matter: lose etcd and
you lose the cluster's memory of everything,
including how to rebuild itself. And the
**controller manager** is where the desired-state
idea lives — there is no single "Kubernetes
process," just dozens of small loops, each keeping
one kind of object true.

## On every node

| Part | Job |
|---|---|
| kubelet | The agent: takes pod assignments for its node, starts them via the runtime, reports health |
| Container runtime | Actually runs containers: usually **containerd** (Docker's engine uses it too) |
| kube-proxy | Programs the node's network rules so Services reach pods (Chapter 6) |

## Objects, not processes

The things you create — Pods, Deployments, Services,
ConfigMaps — are **objects** stored in etcd through
the API. They persist until deleted, they're
described by manifests, and controllers reconcile
them. When something behaves oddly, the reflex to
build is: *look at the object* (`kubectl get`,
`kubectl describe`) before blaming the machinery.

## Managed vs. self-run

- **Managed** (GKE, EKS, AKS, and smaller hosts):
  the provider runs the control plane; you get an
  API endpoint and worker nodes. etcd, upgrades of
  the brain, and its availability are their problem.
  For most teams this is the sane default.
- **Self-run**: you install the control plane
  yourself — on your own machines with a tool like
  k3s or kubeadm, or on your laptop with kind or
  minikube for learning (Chapter 11). You own etcd
  backups and upgrades; in exchange, it runs
  anywhere and costs nothing but the machines.

This guide's example runs on a local, self-run
cluster precisely because that's the one a reader
can build for free in ten minutes.

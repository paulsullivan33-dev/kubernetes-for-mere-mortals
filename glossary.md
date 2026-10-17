# Glossary

**API server** — the cluster's front door. Every
read and change, from kubectl or the system itself,
goes through it.

**Cluster** — a set of nodes plus a control plane,
managed as one Kubernetes system.

**ClusterIP** — the default Service type: reachable
only from inside the cluster.

**ConfigMap** — an object holding plain
configuration (key/value pairs or files), delivered
to pods as environment variables or mounted files.

**Container runtime** — the software that actually
runs containers on a node; usually containerd.

**Control plane** — the cluster's brain: API server,
etcd, scheduler, and controller manager.

**Controller** — a loop that reconciles one kind of
object: compares desired with actual and closes
the gap, forever.

**CrashLoopBackOff** — a container that starts,
exits, and restarts repeatedly; the backoff is the
delay between attempts.

**Deployment** — the object that keeps a set of
identical pods at a desired count and rolls out
template changes gradually.

**Desired state** — what your manifest says should
exist. The cluster's job is making actual state
match it.

**Endpoints** — the live pod addresses currently
behind a Service. Empty endpoints, empty answers.

**etcd** — the control plane's database; the
cluster's single source of truth.

**Helm** — the package manager for bundles of
manifests ("charts") with fill-in values.

**ImagePullBackOff** — the kubelet cannot fetch the
container image and is retrying with growing delays.

**Ingress** — an object describing HTTP routing
rules (by host and path) into Services; needs an
Ingress controller installed to do anything.

**kind** — "Kubernetes in Docker": local clusters
whose nodes are containers.

**kube-proxy** — the per-node component that
programs network rules so Service addresses reach
pods.

**kubectl** — the command-line client for the API.
(`kube-control` and `kube-cuttle` are both heard;
nobody will correct you either way.)

**kubelet** — the agent on each node that starts
assigned pods and reports their health.

**Label** — a key/value tag on an object. Selectors
match labels to connect objects: Services to pods,
Deployments to the pods they count.

**LoadBalancer (Service type)** — asks the cloud
provider for a real external load balancer pointing
at the Service.

**Manifest** — a YAML document describing one
object: `apiVersion`, `kind`, `metadata`, `spec`.

**Namespace** — a named compartment inside a
cluster. Names must be unique only within one;
`default` is the default.

**Node** — one machine (physical or virtual) in
the cluster. Worker nodes run pods; control-plane
nodes run the brain.

**NodePort** — a Service type that also opens a
port (30000–32767) on every node, forwarding to
the Service.

**PersistentVolume (PV)** — a chunk of real storage
in the cluster. A **PersistentVolumeClaim (PVC)**
is a pod-side request that gets bound to one.

**Pod** — the smallest schedulable unit: one or
more containers sharing a network identity,
storage, and lifecycle. Almost always created by a
Deployment, not by hand.

**ReplicaSet** — the counting mechanism under a
Deployment: keeps N identical pods in existence.

**Rollout** — a Deployment's gradual replacement
of old pods with new ones; `kubectl rollout undo`
retreats to the previous revision.

**Scheduler** — the control-plane part that picks
a node for each new pod.

**Secret** — a ConfigMap's sibling for sensitive
values. Stored base64-encoded — an encoding, *not*
encryption.

**Selector** — a label matching rule. The way
Services find pods and Deployments count them.

**Service** — a stable virtual address and DNS
name fronting a shifting set of pods, chosen by
label selector.

**StatefulSet** — the controller for stateful
workloads: stable pod names, stable storage,
ordered starts. Named here so you know it exists.

**StorageClass** — the recipe a provisioner uses
to create PersistentVolumes on demand.

**Volume** — storage attached to a pod, outliving
its containers but not (except via PVs) the pod.

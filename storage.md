# Storage

**Scope:** how pods get files that outlive them —
the three layers between "the container has a
disk" and "the data survives," and why everything
with real data is harder than everything without.

## A container's disk is a scratch pad

Files a container writes to its own filesystem die
with the container — and containers die casually
here (updates, crashes, rescheduling). A **volume**
is storage whose lifecycle is tied to the *pod*
instead: it survives container restarts inside the
pod, and dies only when the pod does. The simplest
kind:

- `emptyDir` — a scratch directory created when the
  pod starts, shared by the pod's containers,
  deleted with the pod. A work area, not a home.
- Volumes can also *project* cluster objects into
  the filesystem — the worked example's home page
  arrives this way, a ConfigMap mounted as a file
  (Chapter 7).

Anything that must outlive the *pod* needs the
real machinery.

## PV, PVC, StorageClass: the three-layer ask

Persistent storage in Kubernetes is deliberately
indirect, so that pods can ask for storage without
knowing what provides it:

| Object | What it is | Who makes it |
|---|---|---|
| **PersistentVolume (PV)** | A chunk of real storage in the cluster: a cloud disk, an NFS export, a local path | The cluster operator, or a provisioner |
| **PersistentVolumeClaim (PVC)** | A pod's *request*: "10 GB, readable by one pod at a time" | You, in a manifest |
| **StorageClass** | The recipe a provisioner follows to create PVs on demand ("fast cloud SSD", "cheap disk") | The cluster / cloud provider |

The pod mounts the *claim*; the cluster **binds**
the claim to a volume that fits (or a provisioner
makes one to order, per the StorageClass named in
the claim). Your manifest says how much and what
kind; the same manifest then works on a cloud
(where the class mints a real disk) and on a local
cluster (where test tools mint a file-backed stand-in).
Access modes are part of the ask — notably
`ReadWriteOnce` (one node at a time, the disk-like
default) versus `ReadWriteMany` (shared, file-like,
rarer and slower).

## Why stateful is harder — the honest paragraph

A stateless pod is fungible: any copy serves any
request, kill any of them, the data lives elsewhere.
A stateful workload breaks every assumption this
guide has used so far. Its pods aren't
interchangeable (one is the primary), its data
can't move at container speed (a disk reattaches;
a database replicates), and "just restart it
somewhere" is a sentence with consequences. That
is why databases are the last thing teams move into
Kubernetes, why a separate object type (StatefulSet:
stable names, stable storage, ordered starts)
exists for them, and why "run the database outside
the cluster, or use a managed one" remains
respectable advice for mortals. This guide's
example is deliberately stateless; know that the
storage layer is *there* before the day you need a
volume that remembers.

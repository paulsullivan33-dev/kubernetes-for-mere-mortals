# What Kubernetes Is

**Scope:** the idea — a system that keeps a
*described* set of containers running across
several machines — and the honest question of
whether your problem is big enough to need it.

## Containers, then the problem after containers

A container runtime (Docker, containerd) runs
containers on **one** machine. That's enough until
it isn't. The problems that appear next are
logistical, not technical:

- The machine dies, and everything on it is down
  until a human rebuilds it.
- One service needs three copies behind some kind
  of load balancing, kept in step.
- A new version should replace the old one
  gradually, and retreat if it misbehaves.
- Ten services, three machines: which runs where,
  and what happens when a machine fills up?

Answering those questions by hand, repeatedly, at
3 a.m., is the job Kubernetes automates. It is an
**orchestrator**: given a description of what
should run, it decides where, starts it, watches
it, and repairs the difference between *described*
and *actual* forever. That loop — describe, compare,
repair — is called **reconciliation**, and nearly
everything in Kubernetes is an instance of it.

## Desired state: the one idea to keep

You don't tell Kubernetes "start two copies of the
web app on machine B." You hand it a **manifest**
(a YAML document, see the Manifests chapter) that
says "two copies of this container should exist,"
and the system makes it true and *keeps* it true:

- A copy crashes → a new one is started.
- A machine dies → its copies are recreated on the
  survivors.
- You edit the manifest from two copies to five →
  three more appear. Edit it to zero → all go away.

The manifest is the **desired state**; the running
containers are the **actual state**; the machinery
between them never stops reconciling. This is why
Kubernetes feels declarative and a little indirect:
you are not driving the car, you are filing the
destination with a very literal chauffeur.

## What it gives you, concretely

| Capability | What it means in practice |
|---|---|
| Scheduling | It picks which machine each container runs on, by available CPU/RAM |
| Self-healing | Dead containers and dead machines are replaced automatically |
| Scaling | "Run 12 copies" is an edit, not a project |
| Rolling updates | New versions replace old ones gradually, with rollback |
| Service discovery | A stable name and address for a shifting set of containers |
| Config management | Settings and secrets delivered to containers from outside the image |

Nothing here is magic, and all of it exists in
simpler tools. Kubernetes' claim is doing all of
it, uniformly, across a fleet, with one API.

## When you honestly don't need it

Most mortal workloads don't. Be suspicious of any
advice — including this guide's existence — that
skips this section.

- **One machine, a few containers:** Docker Compose
  describes the same desired state in one file, on
  one host, with a fraction of the machinery. It
  has no self-healing across machines because there
  are no other machines — which is the point.
- **One static service:** a VM, a binary, and a
  systemd unit (this series has both guides) has
  run production systems for decades.
- **"We might scale someday":** someday is not a
  requirement. Kubernetes can be adopted when the
  second machine actually arrives.

Kubernetes starts paying rent when several of
these are true at once: multiple machines, multiple
services, multiple people deploying, and a real
cost to downtime. Its price is real too — a control
plane to run or rent, a vocabulary to learn, and
failures that are more interesting than "the
process died." The rest of this guide teaches the
vocabulary. Chapter 11 helps you choose where — or
whether — to run it.

# Writing Criteria

Every guide in this repo is written to the same standard. If you're
adding or editing a guide — human or AI — hold it to these.

1. **Plain words, real ideas.** Simple language, never dumbed-down
   concepts. Explain jargon on first use; collect terms in the
   glossary.
2. **No hype.** No marketing adjectives, no "revolutionary," no
   promises about what something will do for the reader.
3. **Honest about limits.** Say what the thing *can't* do. State
   uncertainty instead of smoothing it over. Validation notes include
   what remains unverified.
4. **No invented numbers.** Benchmarks, defaults, and specs are either
   measured or qualified with "depends on your setup." If it wasn't
   tested, say so.
5. **Checkable.** Commands are meant to be run, with "Check:" blocks
   showing what success looks like. Sources are linked, not asserted.
6. **Respect the reader's setup.** Call out Windows/Mac/Linux
   differences. Don't assume a GPU, a budget, or prior experience.
7. **Safety isn't skipped.** Real risks get stated plainly, not buried.
8. **Scoped.** Each guide says what it covers and points elsewhere for
   the rest. No guide pretends to be the whole subject.
9. **Structure over prose.** Tables, before/after examples,
   symptom → fix. Scannable beats essay.

## Verification notes for this guide

Standard 3 and 4 applied to this guide, specifically:

**Verified by running, on the build machine:**
- Every manifest in `examples/` against the
  Kubernetes schemas with kubeconform v0.6.7
  (`-strict`, Kubernetes 1.31.0): 4 resources,
  4 valid, 0 errors.
- kubectl v1.31.14 client operations that need no
  cluster: the `kubectl create ... --dry-run=client
  -o yaml` outputs quoted in the Manifests and
  ConfigMaps chapters are verbatim from real runs,
  as is the base64 encode/decode demonstration.
  (File-based `apply`/`create` cannot validate
  offline — they need a server's API discovery —
  which is why kubeconform did the file checks.)

**Documented-only (not run), and why:**
- All operations against a live cluster: applying
  the example, scaling, rollouts, self-healing,
  Service access, log and event output. The build
  sandbox cannot host a cluster, for reasons
  measured while preparing this guide: its kernel
  has no bridge or VXLAN device support and lacks
  the `xt_addrtype` module Docker's networking
  needs (kind/Docker stop there); its cgroup v2
  hierarchy delegates no CPU or memory controllers
  (k3s stops at startup, and rootless k3s is
  blocked earlier still); and `/etc/hosts` is a
  read-only mount (minikube's none driver stops
  there). Available memory was also under ~1 GB.
  These are limits of that sandbox, not of the
  tools, and none of them apply to an ordinary
  laptop or spare machine.

# ConfigMaps and Secrets

**Scope:** getting configuration into containers
from *outside* the image — the ConfigMap for
ordinary settings, the Secret for sensitive ones,
and the warning about Secrets that matters most.

## Why config lives outside the image

An image should be one artifact you promote from
test to production unchanged. Settings differ per
environment — addresses, feature flags, log levels —
so baking them into the image means rebuilding to
change a setting, and a different image per
environment to test (defeating the point of testing
the artifact you ship). Kubernetes' answer is to
keep config in separate objects and hand it to the
pod at start.

## ConfigMaps

A **ConfigMap** is a named bag of key/value pairs —
plain strings, or whole files. Generated for real
with kubectl while writing this guide:

```
$ kubectl create configmap hello-config \
    --from-literal=greeting="Hello from a ConfigMap" \
    --dry-run=client -o yaml
apiVersion: v1
data:
  greeting: Hello from a ConfigMap
kind: ConfigMap
```

Two ways a pod consumes one:

- **As environment variables** — each key becomes
  an env var in the container. Simple, and frozen
  at pod start: change the ConfigMap and the pod
  sees nothing until it's restarted.
- **As mounted files** — each key becomes a file
  in a directory you mount. This guide's example
  mounts one key, `index.html`, as the web server's
  home page (see `examples/deployment.yaml`). A
  mounted ConfigMap *does* update in place when the
  ConfigMap changes — with one exception worth
  memorizing: files mounted with `subPath` (as the
  example does, to land one file in an existing
  directory) do **not** receive updates. The
  example trades live updates for a tidy mount;
  know which you're choosing.

## Secrets: base64 is not encryption

A **Secret** is a ConfigMap's sibling for sensitive
values — passwords, tokens, keys. It differs in
handling (the API is a little more careful about
who can read them, and kubectl avoids printing
them: in current versions a dry-run print shows
`api-key: <redacted>`), but its values are stored
**base64-encoded, not encrypted**. This is the
whole demonstration, run for real:

```
$ printf 's3cr3t' | base64
czNjcjN0
$ echo czNjcjN0 | base64 --decode
s3cr3t
```

Anyone who can read the Secret can decode it with
one command — base64 is a *transport encoding*, a
way to carry arbitrary bytes in text. The shipped
`examples/secret.yaml` holds exactly that fake
value, `czNjcjN0`, as a specimen. Consequences,
stated plainly:

- **Never commit a real Secret manifest to git.**
  The encoding protects it from nothing.
- Real protection is layered *around* Secrets:
  restricting who can read them (RBAC), encrypting
  etcd at rest, or keeping secrets in an external
  store and injecting them at run time. All three
  are beyond this guide — but "Secret in the repo"
  is the failure to avoid today.
- Treat a Secret's value as visible to anyone with
  cluster API access to its namespace, because it
  is.

Used with those eyes open, Secrets are still the
right mechanism: config-shaped delivery for
sensitive values, separated from ordinary config so
the careful handling has somewhere to attach.

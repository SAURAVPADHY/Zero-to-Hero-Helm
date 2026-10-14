# Day 2 — Chapter 5: Lab — Find the Helm Release Secret

**Difficulty:** Beginner  
**Estimated time:** 20–30 minutes  
**Goal:** Find the Secret for revision 1 of the `web` release in the `default` namespace, then save its name and actual type into the required files.

## What you will create

At the end, these two files must exist:

```text
/root/answers/
├── release-secret.txt
└── release-secret-type.txt
```

- `release-secret.txt` contains the full Secret name.
- `release-secret-type.txt` contains the Secret's actual `type` value read from Kubernetes.

## Step 0 — Check your tools and context

Run:

```bash
kubectl config current-context
kubectl get nodes
helm version
```

These commands confirm which cluster you are connected to and whether the tools are available. If they fail, resolve cluster access before continuing.

## Step 1 — Confirm the `web` release exists

Run:

```bash
helm list -n default
helm status web -n default
```

Look for a release named `web` in the `default` namespace.

If Helm says the release does not exist, stop and verify the release name and namespace with:

```bash
helm list -A
```

Do not create a replacement release unless your instructor specifically asks you to; this task is about inspecting the existing release.

## Step 2 — Create the answer directory

```bash
mkdir -p /root/answers
```

This creates the directory if it does not already exist.

## Step 3 — Find the revision 1 Secret

The default Helm 3 storage name for revision 1 of release `web` is expected to be:

```text
sh.helm.release.v1.web.v1
```

First list the Secrets labelled for the `web` release:

```bash
kubectl get secrets -n default -l owner=helm,name=web
```

Look for the Secret whose name ends in `.v1`.

Now check that exact object:

```bash
kubectl get secret sh.helm.release.v1.web.v1 -n default
```

If it exists, save its full name into the required file:

```bash
printf '%s\n' 'sh.helm.release.v1.web.v1' > /root/answers/release-secret.txt
```

Verify:

```bash
cat /root/answers/release-secret.txt
```

**Important:** If the expected name is not present, do not write a guessed name. Use the Secret list to identify the actual revision 1 Secret for `web`, then save that actual name.

## Step 4 — Read the Secret type from Kubernetes

Do not type the type from memory. Ask Kubernetes for the object's actual `type` field:

```bash
kubectl get secret sh.helm.release.v1.web.v1 \
  -n default \
  -o jsonpath='{.type}{"\n"}'
```

Expected for Helm's default Secret storage is typically:

```text
helm.sh/release.v1
```

Save the actual command output directly to the required file:

```bash
kubectl get secret sh.helm.release.v1.web.v1 \
  -n default \
  -o jsonpath='{.type}{"\n"}' \
  > /root/answers/release-secret-type.txt
```

Verify:

```bash
cat /root/answers/release-secret-type.txt
```

If you discovered a different actual Secret name in Step 3, substitute that name in both commands.

## Step 5 — Describe the Secret

Run:

```bash
kubectl describe secret sh.helm.release.v1.web.v1 -n default
```

Observe the Secret's name, namespace, type, and data-key metadata. The release payload is not displayed as a human-readable manifest.

**Safety note:** Do not dump, decode, or publish the Secret's data. Use `helm get manifest` and `helm get values` when you need to inspect release content.

## Step 6 — Verify both answer files

Run:

```bash
echo "Secret name:"
cat /root/answers/release-secret.txt

echo "Secret type:"
cat /root/answers/release-secret-type.txt
```

Check that:
- The first file contains the full name of the revision 1 Secret.
- The second file contains only the type string read from the Kubernetes object.
- Both files are in `/root/answers/`.

## Troubleshooting

### The Secret is not found

List the matching Secrets:

```bash
kubectl get secrets -n default -l owner=helm,name=web
```

Then check:
- Is the release named `web`?
- Is it installed in `default`?
- Does revision 1 still exist?
- Is Helm using its default Secret storage driver?

### Permission denied

Your Kubernetes identity may lack permission to read Secrets in `default`. Ask your cluster administrator for the minimum required read access; do not try to bypass RBAC.

### No Secrets match the labels

List Secrets in the namespace and inspect metadata:

```bash
kubectl get secrets -n default
```

Do not delete or modify any Secret as part of this lab.

## Acceptance criteria

- [ ] `/root/answers/release-secret.txt` contains the actual revision 1 Secret name.
- [ ] `/root/answers/release-secret-type.txt` contains the Secret type retrieved from Kubernetes.
- [ ] I ran `kubectl describe secret` and inspected its metadata.
- [ ] I can explain why `helm get` is more useful than reading the encoded Secret payload.

## Optional challenge

Compare the Secret names for revisions of `web`:

```bash
kubectl get secrets -n default -l owner=helm,name=web
helm history web -n default
```

Match the Secret revision suffix (for example, `.v1`, `.v2`) with the Helm revision history. Do not delete the Secrets.

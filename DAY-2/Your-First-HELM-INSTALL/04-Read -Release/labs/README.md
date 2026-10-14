# Day 2 — Chapter 4: Hands-on Labs for `helm get`

**Goal:** Inspect a release's stored manifest, computed values, and notes.

**Prerequisites:** Helm and `kubectl` configured, with a release named `web` in the `default` namespace. Replace the release name or namespace if your lab environment differs.

## Lab 1 — Required: Capture the manifest and merged values

Create the answer directory:

```bash
mkdir -p /root/answers
```

### Task A: Save the manifest

```bash
helm get manifest web -n default > /root/answers/web-manifest.yaml
```

Inspect it:

```bash
cat /root/answers/web-manifest.yaml
```

**Acceptance criteria:** The file contains the rendered Deployment and Service produced by the chart.

### Task B: Save merged values

```bash
helm get values web -n default --all > /root/answers/web-values.yaml
```

Inspect it:

```bash
cat /root/answers/web-values.yaml
```

**Acceptance criteria:** The file includes chart defaults and user overrides, including `replicaCount` if that is the chart's configured key.

**Important:** Plain `helm get values web -n default` shows user-supplied values only. Use `--all` to include computed defaults.

## Lab 2 — Compare user-supplied values with computed values

Run:

```bash
helm get values web -n default
helm get values web -n default --all
```

**Tasks**

1. Compare the outputs.
2. Identify one value present in the computed output that may not appear in the user-supplied output.
3. Explain why the two outputs can differ.

Save evidence if useful:

```bash
helm get values web -n default > /root/answers/web-user-values.yaml
helm get values web -n default --all > /root/answers/web-computed-values.yaml
```

## Lab 3 — Inspect notes and combined release data

Run:

```bash
helm get notes web -n default
helm get all web -n default
```

**Tasks**

1. Read any usage instructions printed by the chart.
2. Identify the manifest and values sections in the combined output.
3. Explain why you should review output before sharing it publicly.

Some charts have no meaningful notes. The combined output can contain sensitive values, so do not commit it to a public repository without reviewing and redacting it.

## Lab 4 — Compare the recorded manifest with live objects

Inspect the release manifest:

```bash
helm get manifest web -n default
```

Inspect live Kubernetes resources:

```bash
kubectl get deployments,services -n default
kubectl get deployment <deployment-name> -n default -o yaml
```

Replace `<deployment-name>` with the actual Deployment name.

**Tasks**

1. Find the Deployment in both outputs.
2. Compare the desired replica count and image.
3. Describe one reason live resources might differ from the Helm-stored manifest.

Possible reasons include manual changes after installation, controllers updating fields, or a newer release revision.

## Troubleshooting

- **Release not found:** Check `helm list -A` and verify the release name and namespace.
- **Missing `replicaCount`:** Confirm that the chart uses this values key and that `--all` was used.
- **Empty notes:** The chart may not define notes.
- **Sensitive output:** Do not publish secrets or sensitive configuration from values or manifests.

## Evidence checklist

- [ ] `/root/answers/web-manifest.yaml` contains the Deployment and Service.
- [ ] `/root/answers/web-values.yaml` contains computed values.
- [ ] I compared user-supplied values with computed values.
- [ ] I inspected chart notes or confirmed they are empty.
- [ ] I understand that live Kubernetes objects can drift from the stored release manifest.

## Reflection questions

1. What does `helm get manifest` return?
2. What is the difference between `helm get values` and `helm get values --all`?
3. What is the purpose of `helm get notes`?
4. Why should you be careful when sharing `helm get all` output?
5. Why might a live Deployment differ from the manifest stored by Helm?

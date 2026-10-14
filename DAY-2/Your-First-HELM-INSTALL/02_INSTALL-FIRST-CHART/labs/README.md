# Day 2 — Chapter 2: Hands-on Labs

**Topic:** Release names and namespaces  
**Goal:** Prove that release names must be unique within a namespace, but the same name can be reused in another namespace.

## Prerequisites

- Helm installed
- `kubectl` configured for a practice cluster
- A valid chart at `/root/charts/hello-web` (replace this path if your chart lives elsewhere)

Check access:

```bash
helm version
kubectl get nodes
```

## Lab 1 — Install a release in the default namespace

Install the chart only if a release named `web` does not already exist in `default`:

```bash
helm install web /root/charts/hello-web
```

Inspect the release:

```bash
helm status web -n default
helm list -n default
```

**Tasks**

1. Record the release name and namespace.
2. Identify the revision and status.
3. List the Kubernetes resources created by the chart.

```bash
kubectl get deployments,services,pods -n default
```

If `web` already exists, do not install it again. Inspect it and continue.

## Lab 2 — Try to reuse the name in the same namespace

If the `web` release is already installed in `default`, run:

```bash
helm install web /root/charts/hello-web
```

Expected: Helm refuses because that release name is already in use in the namespace.

**Tasks**

1. Read the error message.
2. Explain why Helm does not allow two releases named `web` in the same namespace.
3. Do not uninstall the existing release just to make the command succeed.

## Lab 3 — Reuse the name in another namespace

Install a release named `web` into `staging`:

```bash
helm install web /root/charts/hello-web \
  --namespace staging \
  --create-namespace
```

Now inspect each namespace:

```bash
helm list -n default
helm list -n staging
helm list -A
```

Inspect each release:

```bash
helm status web -n default
helm status web -n staging
helm history web -n default
helm history web -n staging
```

**Acceptance criteria**

- A release named `web` is present in `default` (if you started with one).
- A separate release named `web` is present in `staging`.
- `helm list -A` shows releases from both namespaces.
- You can explain why the two releases have independent histories.

## Lab 4 — Compare resources in each namespace

```bash
kubectl get deployments,services,pods -n default
kubectl get deployments,services,pods -n staging
```

**Tasks**

1. Compare the resources in each namespace.
2. Explain how namespaces help separate application environments.
3. Remember that namespaces separate namespaced resources, but they are not by themselves a complete security boundary.

## Optional challenge — Use a third namespace

Install the same chart as `web` in a `test` namespace:

```bash
helm install web /root/charts/hello-web \
  --namespace test \
  --create-namespace
```

Then run:

```bash
helm list -A
```

You should now see separate releases with the same name in different namespaces.

## Cleanup

Only remove releases and namespaces that you created for this lab and that contain no other work.

For example, if you created the `staging` release just for this exercise:

```bash
helm uninstall web -n staging
```

If `staging` was created only for this lab and is now empty, you can remove it:

```bash
kubectl delete namespace staging
```

Do not uninstall the pre-existing `web` release in `default` unless it belongs to you and you intend to remove it. Do not delete shared namespaces.

## Evidence to record

Create an `evidence.md` file with:

- Output of `helm list -A`
- Release names and namespaces
- The error from attempting to reuse a name in the same namespace (if tested)
- Answers to:
  1. Why must a release name be unique within a namespace?
  2. Why can `web` exist in both `default` and `staging`?
  3. What does `helm list -A` show?



## Lab 5 — Inspect resources and release labels

**Objective:** Prove that resources installed by Helm are ordinary Kubernetes objects, and use standard labels to identify the `web` release.

**Assumption:** The release `web` already exists in the `default` namespace, and the chart applies the labels `app.kubernetes.io/name` and `app.kubernetes.io/instance`.

### Task A — Find the Deployment and Service

```bash
kubectl get deployments,services -n default
kubectl get deployments,services \
  -n default \
  -l app.kubernetes.io/instance=web
```

Create the answer directory:

```bash
mkdir -p /root/answers
```

Save the matching Deployment and Service names, one per line:

```bash
{
  kubectl get deployments -n default \
    -l app.kubernetes.io/instance=web \
    -o name
  kubectl get services -n default \
    -l app.kubernetes.io/instance=web \
    -o name
} | sed 's#^[^/]*/##' > /root/answers/web-resources.txt
```

Inspect the file:

```bash
cat /root/answers/web-resources.txt
```

It should contain the actual Deployment name and Service name, one per line. If the chart doesn't apply the standard labels, identify the objects with `kubectl get deployments,services -n default` and write their names manually.

### Task B — Save the Deployment's labels

First identify the Deployment name from Task A, then run:

```bash
kubectl get deployment <deployment-name> \
  -n default \
  -o jsonpath='{.metadata.labels}' \
  > /root/answers/web-labels.txt
```

Replace `<deployment-name>` with the actual name. Verify the saved file:

```bash
cat /root/answers/web-labels.txt
```

Confirm that `app.kubernetes.io/instance` is set to `web`.

If you need to print only the application and instance label values:

```bash
kubectl get deployment <deployment-name> \
  -n default \
  -o jsonpath='{.metadata.labels.app\.kubernetes\.io/name}{" "}{.metadata.labels.app\.kubernetes\.io/instance}{"\n"}' \
  > /root/answers/web-labels.txt
```

Use the full-labels command when the requirement is to save the Deployment's metadata labels.

### Acceptance criteria

- [ ] `/root/answers/web-resources.txt` contains the Deployment and Service names, one per line.
- [ ] `/root/answers/web-labels.txt` contains the Deployment's labels.
- [ ] The labels show `app.kubernetes.io/instance` set to `web`.
- [ ] You can explain how labels help identify resources belonging to a Helm release.

**Troubleshooting:** If the label selector returns no resources, list all Deployments and Services in `default`, inspect their labels with `kubectl get deployment <name> -o yaml`, and confirm that the release is installed in the expected namespace.

# Day 2 — Chapter 1: Hands-on Labs

These labs practise the three stages of `helm install`: rendering, submitting resources to Kubernetes, and recording the release.

**Prerequisites:** Helm installed, `kubectl` configured, and a valid local chart at `/root/charts/hello-web` (or update the path to your chart).

## Lab 1 — Render without installing

**Objective:** See how Helm converts templates into normal Kubernetes YAML.

```bash
helm lint /root/charts/hello-web
helm template web /root/charts/hello-web
```

### Tasks

1. Find the Deployment in the rendered output.
2. Identify where the replica count and container image came from.
3. Confirm the output contains no unresolved Helm placeholders such as `{{ .Values... }}`.
4. Explain why `helm template` does not create resources in the cluster.

**Acceptance criteria:** Linting succeeds, you can identify the rendered Deployment and Service, and you can explain rendering versus installing.

## Lab 2 — Install and inspect a release

**Objective:** Install a chart and inspect the resulting Kubernetes objects and Helm release.

```bash
helm install web /root/charts/hello-web \
  --namespace helm-day2 \
  --create-namespace
```

Check the release and resources:

```bash
helm status web -n helm-day2
helm list -n helm-day2
helm history web -n helm-day2
kubectl get deployments,services,pods -n helm-day2
```

### Tasks

1. Record the release name, namespace, status, and revision.
2. Identify the Deployment and Service created by the chart.
3. Compare `helm status` with `kubectl get`.
4. Explain why the release is more than just the Deployment.

If `web` already exists in the namespace, inspect it first or use a unique release name such as `web-lab2`.

## Lab 3 — Prove release history is recorded

**Objective:** Observe how Helm records a later change as a new revision.

Check the chart's configurable values:

```bash
helm show values /root/charts/hello-web
```

If the chart supports `replicaCount`, install a fresh practice release:

```bash
helm install web-history /root/charts/hello-web \
  --namespace helm-day2 \
  --create-namespace \
  --set replicaCount=1
```

Upgrade it:

```bash
helm upgrade web-history /root/charts/hello-web \
  --namespace helm-day2 \
  --set replicaCount=2
```

Inspect history and the workload:

```bash
helm history web-history -n helm-day2
helm get values web-history -n helm-day2
kubectl get deployment -n helm-day2
```

### Tasks

1. Record the revision after installation and after upgrade.
2. Verify the Deployment's desired replica count.
3. Explain why Helm needs a release record for upgrades and rollback.

If your chart does not use `replicaCount`, choose a value supported by that chart.

## Acceptance criteria

- [ ] I can explain the three steps of `helm install`.
- [ ] I can render a chart without installing it.
- [ ] I can install a chart into a dedicated namespace.
- [ ] I can inspect a release with `helm status` and `helm history`.
- [ ] I can verify resources with `kubectl`.
- [ ] I can explain how an upgrade adds a revision.

## Cleanup

Uninstall only the releases you created for these labs:

```bash
helm uninstall web -n helm-day2
helm uninstall web-history -n helm-day2
```

If `helm-day2` was created only for this exercise and contains no other resources, remove it:

```bash
kubectl delete namespace helm-day2
```

Do not delete a shared namespace or a release that belongs to another user.

## Evidence to record

Create an `evidence.md` file with notes or outputs showing:

- The output of `helm template`
- Release status, namespace, and revision
- Kubernetes objects created by the chart
- Revision history before and after the upgrade
- Answers to these questions:
  1. What does rendering do?
  2. Which component receives the manifests?
  3. What does Helm record as a release?
  4. Why is `helm template` useful before installation?

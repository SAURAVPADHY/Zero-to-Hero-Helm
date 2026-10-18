# Day 3 — Chapter 1: Hands-on Labs — Helm Uninstall

**Difficulty:** Beginner  
**Estimated time:** 30–45 minutes  
**Goal:** Practise normal uninstall and uninstall with retained history.

## Prerequisites

- Helm installed and configured
- `kubectl` connected to a practice cluster
- A valid chart, for example `/root/charts/hello-web`
- Permission to create resources in a lab namespace

> **Safety:** Use a dedicated practice namespace. Do not uninstall shared or production workloads.

## Lab 1 — Install a test release

```bash
helm install web-delete-lab /root/charts/hello-web \
  --namespace helm-uninstall-lab \
  --create-namespace
```

Verify:

```bash
helm status web-delete-lab -n helm-uninstall-lab
helm list -n helm-uninstall-lab
kubectl get deployments,services,pods -n helm-uninstall-lab
```

**Acceptance criteria:** The release is deployed and you can identify its Deployment and Service.

## Lab 2 — Normal uninstall: remove the app and history

```bash
helm uninstall web-delete-lab -n helm-uninstall-lab
```

Verify:

```bash
helm list -n helm-uninstall-lab
kubectl get deployments,services,pods -n helm-uninstall-lab
```

Expected: the release is no longer deployed, its chart-managed resources are removed, and its history is removed by default.

**Tasks**
1. Record the output from `helm list`.
2. Confirm the Deployment and Service are gone.
3. Explain why Helm can remove resources without you specifying each manifest.

## Lab 3 — Keep history while removing the app

Install a second test release:

```bash
helm install web-keep-lab /root/charts/hello-web \
  --namespace helm-uninstall-lab \
  --create-namespace
```

Uninstall but retain history:

```bash
helm uninstall web-keep-lab \
  --namespace helm-uninstall-lab \
  --keep-history
```

Inspect the retained release:

```bash
helm list --uninstalled -n helm-uninstall-lab
helm history web-keep-lab -n helm-uninstall-lab
kubectl get deployments,services,pods -n helm-uninstall-lab
```

Expected:
- `web-keep-lab` appears as uninstalled in the uninstalled listing.
- Release history remains available.
- Its chart-managed Deployment and Service have been removed.

## Lab 4 — Compare both outcomes

| Check | Normal uninstall | `--keep-history` |
|---|---|---|
| Workload resources removed? | Yes | Yes |
| Release history retained? | No, by default | Yes |
| Can inspect retained history? | Usually no | Yes |

Run:

```bash
helm list -n helm-uninstall-lab
helm list --uninstalled -n helm-uninstall-lab
helm history web-keep-lab -n helm-uninstall-lab
```

## Lab 5 — Optional: reinstall after normal uninstall

Because the normal uninstall removed the history, the release name is generally available again:

```bash
helm install web-delete-lab /root/charts/hello-web \
  --namespace helm-uninstall-lab
helm status web-delete-lab -n helm-uninstall-lab
helm history web-delete-lab -n helm-uninstall-lab
```

This is a new installation and starts a new release history. Do not assume a name whose history was retained can be reused in the same way.

## Cleanup

Uninstall any remaining test releases:

```bash
helm uninstall web-delete-lab -n helm-uninstall-lab
helm uninstall web-keep-lab -n helm-uninstall-lab
```

If the namespace was created only for this lab and contains no other work, remove it:

```bash
kubectl delete namespace helm-uninstall-lab
```

Verify the namespace contents before deleting it. Do not delete a shared namespace.

## Troubleshooting

- **Release not found:** Check `helm list -A` and verify name and namespace.
- **Resources remain:** Check whether they belong to another release or were created manually.
- **Name still in use:** Check `helm list --uninstalled -n <namespace>` and `helm history <name> -n <namespace>`.
- **Permission error:** Check your Kubernetes context and RBAC.

## Evidence checklist

- [ ] Installed a test release in a dedicated namespace.
- [ ] Performed a normal uninstall and checked resources.
- [ ] Used `--keep-history` on a separate test release.
- [ ] Verified retained history using `helm list --uninstalled` and `helm history`.
- [ ] Cleaned up only resources created for this lab.

## Reflection questions

1. What two things does normal uninstall remove?
2. What changes when using `--keep-history`?
3. Why might an SRE retain release history?
4. Why should you avoid deleting Helm release Secrets manually?
5. What checks should you run before uninstalling a production release?

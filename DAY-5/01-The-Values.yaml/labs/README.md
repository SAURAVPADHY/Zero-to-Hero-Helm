# Day 5 — Chapter 1: Lab — Change a Default Value and Roll It Out

**Difficulty:** Beginner  
**Estimated time:** 25–40 minutes  
**Goal:** Change the chart's default replica count to `3`, upgrade the `webapp` release, and verify that all three replicas are ready.

## Prerequisites

- Helm and `kubectl` installed
- Chart at `/root/work/webapp`
- Release `webapp` installed in namespace `default`
- Permission to update the release and scale its Deployment

> **Safety:** This lab changes a running release. Use a practice cluster or confirm the change is approved before proceeding.

## Step 1 — Confirm the release and Deployment

```bash
helm status webapp -n default
kubectl get deployments -n default
kubectl get pods -n default
```

Identify the Deployment managed by the `webapp` release. The default scaffold commonly names it `webapp`, but check the actual output.

## Step 2 — Inspect the current defaults

```bash
grep -n -A4 '^replicaCount:' /root/work/webapp/values.yaml
grep -n -A2 '^color:' /root/work/webapp/values.yaml
```

The current default should be `replicaCount: 1`, and the `color` value should be `blue`. **Do not change the color value in this lab.**

## Step 3 — Change `replicaCount` to 3

```bash
vi /root/work/webapp/values.yaml
```

Change only the replica count:

```yaml
replicaCount: 3
```

Leave the rest of the file intact, including `color: blue`. Save and exit with `:wq`.

Verify:

```bash
grep -n '^replicaCount:' /root/work/webapp/values.yaml
grep -n '^color:' /root/work/webapp/values.yaml
```

Expected values:

```text
replicaCount: 3
color: blue
```

## Step 4 — Render locally before applying

```bash
helm template webapp /root/work/webapp
```

Inspect the rendered Deployment and confirm it contains `replicas: 3`. If it still shows one replica, check that the template references `.Values.replicaCount` and that you edited the correct values file.

## Step 5 — Upgrade the existing release

```bash
helm upgrade webapp /root/work/webapp -n default
```

This applies the changed chart configuration to the installed release.

## Step 6 — Wait for all three replicas

Find the Deployment name:

```bash
kubectl get deployments -n default
```

If it is named `webapp`, run:

```bash
kubectl rollout status deployment/webapp -n default --timeout=180s
kubectl get deployment webapp -n default
kubectl get pods -n default
```

The Deployment should show `3/3` ready replicas. If it has another name, substitute the actual name.

## Step 7 — Verify Helm's release state

```bash
helm status webapp -n default
helm list -n default
helm get values webapp -n default --all
```

Confirm the release is deployed and the effective replica count is `3`.

## Acceptance criteria

- [ ] `values.yaml` has `replicaCount: 3`.
- [ ] The `color` value remains `blue`.
- [ ] `helm template` shows three replicas.
- [ ] The existing `webapp` release was upgraded.
- [ ] The Deployment reports three ready replicas.

## Troubleshooting

- **Helm says release not found:** Check `helm list -A` and confirm the release namespace.
- **Deployment remains at one replica:** Confirm the template reads `.Values.replicaCount`, then inspect `helm get values webapp -n default --all`.
- **Rollout is stuck:** Run `kubectl describe deployment <deployment-name> -n default` and inspect pod events with `kubectl describe pod <pod-name> -n default`.
- **Pods do not become Ready:** Check pod events and container logs.
- **Color changed unexpectedly:** Restore `color: blue` before submitting.

## Reflection questions

1. Where do the default values for `.Values` come from?
2. Why is changing `values.yaml` considered a chart change?
3. Does editing `values.yaml` update a running release automatically?
4. Why must you run `helm upgrade`?
5. How would you configure different replica counts for staging and production without editing chart defaults each time?

# Day 5 — Chapter 2: Labs — Override Values with `--set` and `-f`

**Difficulty:** Beginner  
**Estimated time:** 40–60 minutes  
**Goal:** Override a value for one release, then use separate values files to install development and production releases from the same chart.

## Prerequisites

- Helm and `kubectl` installed and configured
- Chart directory at `/root/work/webapp`
- `webapp` release installed in namespace `default`
- Permission to create namespaces and deploy resources

> **Safety:** These labs change a running release and create resources in `dev` and `prod`. Use a practice cluster.

## Lab 1 — Confirm the chart default

```bash
grep -n '^color:' /root/work/webapp/values.yaml
grep -n '^replicaCount:' /root/work/webapp/values.yaml
```

Expected starting point for this task: `color: blue`. The replica count may already be `3` after the previous chapter.

## Lab 2 — Override `color` with `--set`

First confirm the release:

```bash
helm status webapp -n default
```

Upgrade with a one-time override:

```bash
helm upgrade webapp /root/work/webapp \
  -n default \
  --set color=green
```

Verify the release's supplied values:

```bash
helm get values webapp -n default
```

Verify the ConfigMap:

```bash
kubectl get configmap webapp-info -n default -o yaml
```

**Acceptance criteria**
- Helm records `color: green` as a release override.
- The ConfigMap has `color: green`.
- `/root/work/webapp/values.yaml` still contains `color: blue`.

If your chart does not yet include the `color` field in `webapp-info`, check that the ConfigMap template publishes `.Values.color` and that the release was upgraded after that template was added.

## Lab 3 — Create development and production values files

Create the development file:

```bash
cat > /root/work/values-dev.yaml <<'EOF'
replicaCount: 1
color: blue
EOF
```

Create the production file:

```bash
cat > /root/work/values-prod.yaml <<'EOF'
replicaCount: 3
color: green
EOF
```

Read both files back:

```bash
cat /root/work/values-dev.yaml
cat /root/work/values-prod.yaml
```

## Lab 4 — Install the development release

```bash
helm upgrade --install webapp-dev /root/work/webapp \
  -n dev \
  --create-namespace \
  -f /root/work/values-dev.yaml
```

Verify:

```bash
helm status webapp-dev -n dev
helm get values webapp-dev -n dev
kubectl get deployments -n dev
kubectl get pods -n dev
```

Expected configuration: one replica and blue.

## Lab 5 — Install the production release

```bash
helm upgrade --install webapp-prod /root/work/webapp \
  -n prod \
  --create-namespace \
  -f /root/work/values-prod.yaml
```

Verify:

```bash
helm status webapp-prod -n prod
helm get values webapp-prod -n prod
kubectl get deployments -n prod
kubectl get pods -n prod
```

Expected configuration: three replicas and green. Wait for readiness if needed:

```bash
kubectl get deployments -n prod --watch
```

Press `Ctrl+C` when the Deployment is ready.

## Lab 6 — Compare the releases

```bash
helm list -A
helm get values webapp-dev -n dev
helm get values webapp-prod -n prod
```

If the chart includes the info ConfigMap, compare its values:

```bash
kubectl get configmap -n dev
kubectl get configmap -n prod
kubectl get configmap webapp-info -n dev -o yaml
kubectl get configmap webapp-info -n prod -o yaml
```

Note: The ConfigMap name can differ if your chart's helper templates generate another name. Use `kubectl get configmaps -n <namespace>` to discover it.

## Lab 7 — Optional: prove `--set` wins over `-f`

Render the production chart with the file and a conflicting command-line value:

```bash
helm template webapp-prod /root/work/webapp \
  -n prod \
  -f /root/work/values-prod.yaml \
  --set color=purple
```

If the templates expose `color`, the rendered output should use `purple`, because the command-line override takes precedence over the file.

## Troubleshooting

- **Namespace not found:** Include `--create-namespace` when installing into a new namespace.
- **Release not found:** `helm upgrade` requires an existing release; use `helm upgrade --install` for install-or-upgrade behavior.
- **Wrong color:** Check `helm get values <release> -n <namespace>` and confirm the intended values file or `--set` argument was used.
- **Wrong replica count:** Confirm the Deployment template uses `.Values.replicaCount` and check the release's effective values.
- **ConfigMap does not show color:** Ensure the template contains `color: {{ .Values.color | quote }}` under `data`, then upgrade the release.
- **Pods not ready:** Inspect `kubectl describe deployment <name> -n <namespace>` and pod events before changing configuration further.

## Acceptance checklist

- [ ] `webapp` was upgraded with `--set color=green`.
- [ ] The chart's default `color` remains `blue`.
- [ ] `/root/work/values-dev.yaml` has one replica and blue.
- [ ] `/root/work/values-prod.yaml` has three replicas and green.
- [ ] `webapp-dev` is installed in `dev`.
- [ ] `webapp-prod` is installed in `prod`.
- [ ] `helm list -A` shows both environment releases.
- [ ] Dev and prod have the expected effective values.

## Reflection questions

1. When would you choose `--set` instead of `-f`?
2. Why should environment-specific values live outside the chart defaults?
3. If both `-f` and `--set` define `color`, which wins?
4. Why can one chart safely be installed as two different releases?
5. How can you check which overrides a running release received?

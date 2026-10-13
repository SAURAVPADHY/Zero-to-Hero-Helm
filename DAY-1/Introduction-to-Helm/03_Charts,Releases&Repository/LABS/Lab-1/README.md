# Lab 3: One Chart, Two Releases — Staging and Production

**Chapter:** 3 — Charts, Releases, and Repositories  
**Difficulty:** Beginner–Intermediate  
**Estimated time:** 30–45 minutes  
**Goal:** Install the same Helm chart twice with different release names, namespaces, and configuration values.

## Scenario

Your team maintains one web application, but it runs in two environments:

- **Staging:** 1 replica, used to test changes.
- **Production:** 3 replicas, used by customers.

Previously, a team might copy the Kubernetes manifests into separate folders and edit each copy. In this lab, you will reuse the same chart and provide different values files instead.

## Prerequisites

Complete **Lab 2: Build Your First Helm Chart** first. You need a working chart named `hello-web` with a configurable `replicaCount`, Deployment, and Service.

Check that Helm and Kubernetes are available:

```bash
helm version
kubectl get nodes
```

## 1. Locate your chart

Go to the directory containing the `hello-web` chart from Lab 2.

Example:

```bash
cd path/to/hello-web
```

The directory should contain:

```text
hello-web/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    └── service.yaml
```

Run a quick validation:

```bash
helm lint .
```

## 2. Create staging values

Create `values-staging.yaml` beside `Chart.yaml`:

```yaml
replicaCount: 1
```

This file overrides the chart's default replica count for staging.

## 3. Create production values

Create `values-production.yaml` beside `Chart.yaml`:

```yaml
replicaCount: 3
```

Both environments use the same chart templates. Only the configuration differs.

## 4. Preview each environment before deployment

Render the staging manifests:

```bash
helm template web-staging . -f values-staging.yaml
```

Render the production manifests:

```bash
helm template web . -f values-production.yaml
```

Inspect each output and confirm that staging renders one replica and production renders three.

`helm template` renders manifests locally; it does not install resources into the cluster.

## 5. Install the staging release

```bash
helm upgrade --install web-staging . \
  --namespace staging \
  --create-namespace \
  -f values-staging.yaml
```

In this command:

- `web-staging` is the release name.
- `.` points to the chart in the current directory.
- `staging` is the Kubernetes namespace.
- `values-staging.yaml` supplies the staging configuration.

## 6. Install the production release

```bash
helm upgrade --install web . \
  --namespace production \
  --create-namespace \
  -f values-production.yaml
```

This creates a separate release called `web` in the `production` namespace.

> These commands assume the chart's Deployment and Service names are based on `.Release.Name`, as in Lab 2. If your chart uses fixed resource names, review its templates before installing multiple releases into the same namespace.

## 7. Verify both releases

List all Helm releases:

```bash
helm list -A
```

Check staging:

```bash
helm status web-staging -n staging
kubectl get deployments,services,pods -n staging
```

Check production:

```bash
helm status web -n production
kubectl get deployments,services,pods -n production
```

Expected result:

- `web-staging` exists in namespace `staging`.
- `web` exists in namespace `production`.
- Staging Deployment has 1 desired replica.
- Production Deployment has 3 desired replicas.
- Each release has its own Service and Pods.

Wait for each Deployment to become ready:

```bash
kubectl rollout status deployment/web-staging -n staging
kubectl rollout status deployment/web -n production
```

## 8. Change staging without changing production

Update `values-staging.yaml` to:

```yaml
replicaCount: 2
```

Upgrade only the staging release:

```bash
helm upgrade web-staging . \
  --namespace staging \
  -f values-staging.yaml
```

Verify both environments:

```bash
kubectl get deployment web-staging -n staging
kubectl get deployment web -n production
```

Expected: staging now desires 2 replicas, while production remains at 3.

Inspect the release histories:

```bash
helm history web-staging -n staging
helm history web -n production
```

This shows that the releases can be upgraded independently even though they originate from the same chart.

## 9. Acceptance criteria

| Requirement | Expected result |
|---|---|
| Shared chart | Both environments use the same chart directory |
| Separate releases | `web-staging` and `web` exist |
| Namespace isolation | Releases are in `staging` and `production` |
| Initial configuration | Staging has 1 replica; production has 3 |
| Independent upgrade | Staging changes to 2 replicas; production stays at 3 |
| Release history | Each release has its own revision history |

## 10. Reflection questions

1. What is the difference between the chart name and a release name?
2. Can two releases use the same chart? Explain how.
3. Why are separate values files useful?
4. What happens if you upgrade `web-staging` but do not upgrade `web`?
5. What does a namespace help isolate?
6. Why should you render the chart before installing it?

## 11. Clean up

Remove the two releases:

```bash
helm uninstall web-staging -n staging
helm uninstall web -n production
```

If these namespaces were created only for this lab and contain no other resources, you may delete them:

```bash
kubectl delete namespace staging
kubectl delete namespace production
```

**Warning:** Deleting a namespace removes all resources inside it. Skip namespace deletion if it contains other work.

## Optional challenge

Add a different resource limit to staging and production through the values files. Upgrade both releases and verify the rendered configuration with:

```bash
helm get values web-staging -n staging
helm get values web -n production
```

## Key takeaway

**A chart is the package; a release is one installed instance of that chart.** One chart can power multiple environments with different release names and values, avoiding duplicated chart templates.

**Next lab:** Upgrade a release, inspect its revisions, and practise rolling back to a previous revision.

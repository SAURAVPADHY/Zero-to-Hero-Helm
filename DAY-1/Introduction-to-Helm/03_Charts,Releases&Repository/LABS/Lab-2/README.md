# Lab 4: Helm Upgrade, Release History, and Rollback

**Chapter:** 3 — Charts, Releases, and Repositories  
**Difficulty:** Beginner–Intermediate  
**Estimated time:** 30–45 minutes  
**Goal:** Upgrade a Helm release, inspect its revision history, and roll back to a previous revision.

## Scenario

You manage the `web` application in production. The team changes the desired replica count from 3 to 4. After the upgrade, you need to know which release revision was installed and practise restoring the earlier configuration.

You will use the chart from Lab 2 and the production values from Lab 3.

## Prerequisites

- A working Kubernetes cluster
- Helm installed and configured
- The `hello-web` chart from Lab 2
- The production values file from Lab 3

Verify access:

```bash
helm version
kubectl get nodes
```

Go to the directory containing your `hello-web` chart:

```bash
cd path/to/hello-web
```

Confirm the chart passes validation:

```bash
helm lint .
```

## 1. Install or confirm the production release

If you still have the `web` release from Lab 3, inspect it:

```bash
helm status web -n production
helm history web -n production
kubectl get deployment web -n production
```

If the release does not exist, install it using your production values file. Adjust the file path if it is stored elsewhere:

```bash
helm upgrade --install web . \
  --namespace production \
  --create-namespace \
  -f values-production.yaml
```

For a clean exercise, make sure the initial desired replica count is **3**.

## 2. Record the starting revision

Run:

```bash
helm history web -n production
helm get values web -n production
kubectl get deployment web -n production
```

Write down the latest revision number. Do not assume it is `1`; earlier installs and upgrades may already have created revisions.

Expected starting state:

- Release: `web`
- Namespace: `production`
- Desired replicas: `3`

## 3. Prepare the new configuration

Create a copy of the production values file named `values-production-v2.yaml`:

```yaml
replicaCount: 4
```

This file is a minimal override. Helm combines it with the chart's default values, so other chart settings continue to use their defaults unless overridden.

Preview the rendered Deployment before applying the change:

```bash
helm template web . \
  --namespace production \
  -f values-production-v2.yaml
```

Confirm that the rendered Deployment specifies four replicas.

## 4. Upgrade the release

Apply the new values:

```bash
helm upgrade web . \
  --namespace production \
  -f values-production-v2.yaml
```

Check the rollout:

```bash
kubectl rollout status deployment/web -n production
kubectl get deployment web -n production
```

Inspect the release history:

```bash
helm history web -n production
```

Expected result: Helm records a new release revision and the Deployment's desired replica count becomes 4.

## 5. Inspect the current release

```bash
helm status web -n production
helm get values web -n production
helm get manifest web -n production
```

These commands help answer different questions:

- `helm status`: What is the release's current status?
- `helm get values`: Which supplied values are stored for the release?
- `helm get manifest`: Which Kubernetes manifests were rendered for the release?

By default, `helm get values` shows user-supplied values. Use `helm get values web -n production --all` to include computed values.

## 6. Roll back to the earlier revision

First, read the history:

```bash
helm history web -n production
```

Identify the revision from before the upgrade. If it is revision `1`, the command is:

```bash
helm rollback web 1 -n production
```

**Important:** `1` is only an example. Use the actual revision number recorded before your upgrade.

Wait for the Deployment and verify the state:

```bash
kubectl rollout status deployment/web -n production
kubectl get deployment web -n production
helm history web -n production
helm status web -n production
```

Expected result: the desired replica count returns to 3, assuming the selected earlier revision had that configuration. Helm typically records the rollback as a new revision; it does not erase the intervening history.

## 7. Acceptance criteria

| Requirement | Expected result |
|---|---|
| Starting state | Production release has 3 desired replicas |
| Upgrade | Desired replicas change to 4 |
| History | A new revision appears after the upgrade |
| Inspection | You can inspect release values and manifests |
| Rollback | Earlier configuration is restored |
| Final verification | Release history shows the upgrade and rollback |

## 8. Troubleshooting

Check release status and history:

```bash
helm status web -n production
helm history web -n production
```

Inspect the Deployment and Pods:

```bash
kubectl describe deployment web -n production
kubectl get pods -n production
kubectl describe pod <pod-name> -n production
kubectl logs <pod-name> -n production
```

Replace `<pod-name>` with a real Pod name.

If the rollback does not show the expected replica count, confirm that you selected the correct prior revision and inspect its stored values and manifest.

## 9. Reflection questions

1. What is a Helm release revision?
2. How is `helm history` useful during an incident?
3. Why should you preview a change before upgrading?
4. Does rollback delete the release history? Why or why not?
5. Does Helm rollback automatically reverse database migrations or external side effects?
6. Why should a team test rollback procedures before relying on them in production?

## 10. Clean up

When you have finished collecting evidence, uninstall the release if it was created only for this lab:

```bash
helm uninstall web -n production
```

Delete the namespace only if it was created for this exercise and contains no other resources:

```bash
kubectl delete namespace production
```

**Warning:** Namespace deletion removes every resource inside it. Do not delete a shared production or training namespace.

## Evidence to save in GitHub

Create an `evidence.md` file containing:

- The output of `helm history web -n production` before the upgrade
- The replica count before and after the upgrade
- The revision selected for rollback and why
- The final replica count after rollback
- Answers to the reflection questions

Avoid committing secrets or sensitive cluster information into the repository.

## Key takeaway

Helm records release revisions so you can inspect deployment history and roll back to a previous release configuration. Rollback is valuable, but it is not a universal undo button: data changes, database migrations, and external side effects may need separate recovery plans.

**Next step:** Practise these steps with a second chart or a non-production application, and document the exact verification commands you would use during an incident.

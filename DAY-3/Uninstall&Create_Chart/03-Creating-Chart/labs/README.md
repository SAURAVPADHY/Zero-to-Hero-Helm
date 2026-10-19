# Day 3 — Chapter 3: Lab — Edit `Chart.yaml` and Install Your Chart

**Difficulty:** Beginner  
**Estimated time:** 25–40 minutes  
**Goal:** Update the metadata and NGINX image tag in your chart, install it as the `webapp` release, and confirm its pod is Ready.

## Prerequisites

- Helm installed and connected to your practice cluster
- The chart created in the previous chapter at `/root/work/webapp`
- Permission to create Deployments and Services in the `default` namespace

> **Safety:** Use your practice cluster. If a release named `webapp` already exists in `default`, inspect it before changing or uninstalling anything.

## Step 1 — Confirm your chart exists

```bash
ls -la /root/work/webapp
```

Check that these files exist:

```bash
test -f /root/work/webapp/Chart.yaml && echo "Chart.yaml exists"
test -f /root/work/webapp/values.yaml && echo "values.yaml exists"
```

If the chart is missing, complete the previous chapter's `helm create webapp` lab first.

## Step 2 — Inspect `Chart.yaml`

```bash
cat /root/work/webapp/Chart.yaml
```

Open it in an editor:

```bash
vi /root/work/webapp/Chart.yaml
```

Update the required fields so they match this example. Keep any other valid scaffold fields that are already present:

```yaml
apiVersion: v2
name: webapp
description: Deploys an NGINX web server on Kubernetes
type: application
version: 0.2.0
appVersion: "1.1"
```

Save and exit in `vi` with `:wq`.

**Important:** The description must be a real sentence, `version` must be `0.2.0`, and `appVersion` must include quotes around `"1.1"`.

Verify:

```bash
grep -E '^(apiVersion|name|description|type|version|appVersion):' \
  /root/work/webapp/Chart.yaml
```

## Step 3 — Update the image values

Inspect the image section:

```bash
grep -n -A5 '^image:' /root/work/webapp/values.yaml
```

Open the values file:

```bash
vi /root/work/webapp/values.yaml
```

Under `image:`, make sure the repository is `nginx` and the tag is explicitly `alpine`. The section should look similar to:

```yaml
image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: alpine
```

Keep other existing values and indentation intact. The `tag` should be a sibling of `repository` and `pullPolicy`, nested under `image`.

Verify:

```bash
grep -n -A5 '^image:' /root/work/webapp/values.yaml
```

## Step 4 — Validate the chart before installing

Run:

```bash
helm lint /root/work/webapp
helm show chart /root/work/webapp
```

Check that the chart metadata reports version `0.2.0`.

Preview the rendered manifests:

```bash
helm template webapp /root/work/webapp
```

In the rendered Deployment, confirm the container image uses the `nginx:alpine` tag. If it does not, inspect `templates/deployment.yaml` to see how the image is assembled from values.

## Step 5 — Check whether the release name is already in use

```bash
helm list -n default
helm list --uninstalled -n default
```

If `webapp` is already deployed, do not blindly run `helm install` again. Inspect the existing release and follow your instructor's instructions. If a retained uninstalled release named `webapp` exists, it may also affect reuse of that name.

## Step 6 — Install the chart

If no release named `webapp` is already in use in `default`, run:

```bash
helm install webapp /root/work/webapp -n default
```

Check the result:

```bash
helm list -n default
helm status webapp -n default
```

The chart column in `helm list` should show `webapp-0.2.0`.

## Step 7 — Wait for the pod to become Ready

List the resources:

```bash
kubectl get deployments,services,pods -n default
```

Find the Deployment created by this chart, then wait for it to roll out. The scaffold's default Deployment is usually named `webapp`, but verify the actual name from the output.

```bash
kubectl rollout status deployment/webapp -n default --timeout=120s
kubectl get pods -n default
```

The pod should show `Running` and `READY` should indicate that its container is ready, commonly `1/1`.

If the Deployment has a different name, substitute the actual name.

## Step 8 — Final verification

```bash
helm list -n default
helm status webapp -n default
helm get values webapp -n default --all
kubectl get deployment,pods -n default
```

### Acceptance criteria

- [ ] `/root/work/webapp/Chart.yaml` has `version: 0.2.0`.
- [ ] `appVersion` is written as `"1.1"` with quotes.
- [ ] The description clearly states what the chart deploys.
- [ ] `values.yaml` has `image.repository: nginx` and `image.tag: alpine`.
- [ ] Helm lists release `webapp` with chart `webapp-0.2.0`.
- [ ] The Deployment rollout succeeds and its pod is Ready.

## Troubleshooting

- **`helm install` says the name is already in use:** Check both deployed and uninstalled releases. Do not uninstall an existing workload unless you know it is safe.
- **Image pull errors:** Inspect the pod with `kubectl describe pod <pod-name> -n default`. Confirm the node can reach the image registry.
- **Pod is not Ready:** Check `kubectl get pods -n default` and `kubectl describe pod <pod-name> -n default`.
- **Rendered image tag is wrong:** Check the `image:` block in `values.yaml` and how `templates/deployment.yaml` references `.Values.image`.
- **Chart version is still old:** Confirm you edited `/root/work/webapp/Chart.yaml`, then rerun `helm show chart /root/work/webapp`.

## Reflection questions

1. What is the difference between `version` and `appVersion`?
2. Does changing `appVersion` automatically change the image tag?
3. Why is `"1.1"` quoted in YAML?
4. What does `helm lint` check?
5. Why should you inspect the rendered image before installing?

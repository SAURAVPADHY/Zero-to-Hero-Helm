# Day 2 — Chapter 3: Hands-on Labs

**Topic:** Helm release records and tracked state  
**Prerequisites:** Helm installed, `kubectl` configured, and a release named `web` in the `default` namespace for the required task.

Create the answer directory before starting:

```bash
mkdir -p /root/answers
```

## Lab 1 — Required: List releases and check status

**Scenario:** A teammate wants to know what is deployed across the cluster and whether `web` is healthy.

### Task A: Save the full status output

```bash
helm status web -n default > /root/answers/web-status.txt
```

Check the file:

```bash
cat /root/answers/web-status.txt
```

**Acceptance criteria:**
- `/root/answers/web-status.txt` exists.
- It contains the line `STATUS: deployed`.

If the command reports that the release does not exist, confirm the release name and namespace. Do not fabricate or manually add the expected status.

### Task B: Save all releases across namespaces

```bash
helm list -A > /root/answers/all-releases.txt
```

Read the output:

```bash
cat /root/answers/all-releases.txt
```

**Acceptance criteria:**
- `/root/answers/all-releases.txt` exists.
- It lists releases from all namespaces visible to your Kubernetes credentials.
- You can identify the name, namespace, revision, status, and chart columns.

## Lab 2 — Inspect release history

**Objective:** See how Helm tracks revisions.

```bash
helm history web -n default
```

Save the history as evidence:

```bash
helm history web -n default > /root/answers/web-history.txt
cat /root/answers/web-history.txt
```

**Tasks:**
1. Identify the latest revision.
2. Read the description and status for each revision.
3. Explain why a rollback creates a new revision rather than deleting history.

**Acceptance criteria:** You can identify the current revision and describe what the history tells you.

## Lab 3 — Inspect the values and manifest

**Objective:** Compare Helm's recorded configuration with the Kubernetes resources.

```bash
helm get values web -n default
helm get manifest web -n default
```

Save the outputs:

```bash
helm get values web -n default > /root/answers/web-values.txt
helm get manifest web -n default > /root/answers/web-manifest.yaml
```

**Tasks:**
1. Find a value used by the chart, such as replica count or image tag.
2. Identify a Deployment and Service in the rendered manifest.
3. Compare the manifest with the actual resources:

```bash
kubectl get deployments,services -n default
```

**Acceptance criteria:** You can explain the difference between Helm's recorded release configuration and the current live objects.

## Lab 4 — Find the default Helm release Secrets (challenge)

**Objective:** Observe where Helm 3 normally stores release records.

By default, Helm 3 uses Kubernetes Secrets as its release storage backend. List Secrets that commonly match Helm's release naming pattern:

```bash
kubectl get secrets -n default \
  -l owner=helm,name=web
```

You may also inspect Secret names:

```bash
kubectl get secrets -n default
```

Names often resemble `sh.helm.release.v1.web.v1`, with the final number indicating a revision. Exact names and labels can vary with Helm version and storage configuration.

**Important:** Do not print or share decoded Secret data. Release records may contain configuration values, including sensitive information if a chart was supplied with secrets.

**Acceptance criteria:** You can explain that release storage lives in the cluster and is namespace-scoped, without exposing Secret contents.

## Troubleshooting

- **Release not found:** Verify the name and namespace with `helm list -A`.
- **Permission denied:** Check your Kubernetes context and permissions.
- **Status is not `deployed`:** Read `helm status` and investigate the release instead of changing the output file.
- **No matching Secret:** Confirm the namespace and storage driver; Helm may be configured to use a different backend.

## Evidence checklist

- [ ] `/root/answers/web-status.txt` contains `STATUS: deployed`.
- [ ] `/root/answers/all-releases.txt` contains the all-namespace release listing.
- [ ] `/root/answers/web-history.txt` contains release history.
- [ ] `/root/answers/web-values.txt` contains the recorded values output.
- [ ] `/root/answers/web-manifest.yaml` contains the release manifest.
- [ ] I can explain result versus memory.

## Reflection questions

1. What three pieces of information are associated with a Helm release record?
2. Where is the release record stored by default in Helm 3?
3. What is the difference between `kubectl get` and `helm history`?
4. Why does Helm need release history for rollback?
5. Why should Helm release Secrets be handled carefully?

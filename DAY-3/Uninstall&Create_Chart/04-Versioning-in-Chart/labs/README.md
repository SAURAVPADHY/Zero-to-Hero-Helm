# Day 3 — Chapter 4: Hands-on Labs — Chart Version vs Application Version

**Difficulty:** Beginner  
**Estimated time:** 25–40 minutes  
**Goal:** Practise deciding which version field to change and verify what Helm reports.

## Prerequisites

- Chart directory at `/root/work/webapp`
- A deployed release named `webapp` in the `default` namespace for the inspection steps
- Helm CLI configured for your practice cluster

> **Safety:** These first exercises inspect files only. Do not upgrade or uninstall shared workloads unless you have permission.

## Lab 1 — Read both versions

Run:

```bash
helm list -n default
helm show chart /root/work/webapp
```

Inspect the metadata directly:

```bash
grep -E '^(name|version|appVersion):' /root/work/webapp/Chart.yaml
```

**Tasks**
1. Record the chart `version`.
2. Record the `appVersion`.
3. Explain what each value represents.

## Lab 2 — Simulate a chart-only change

Imagine you fixed a typo in `templates/deployment.yaml`, but you did not change the application image.

**Task:** Decide which field should change.

A common patch-release example is:

```yaml
version: 0.2.1
appVersion: "1.1"
```

Do not edit your working chart unless you want to make this change for real. If you do edit it, validate the chart:

```bash
helm lint /root/work/webapp
helm show chart /root/work/webapp
```

**Expected reasoning:** Raise `version` because the chart changed. Keep `appVersion` the same because the application did not change.

## Lab 3 — Simulate an application image change

Imagine the chart currently deploys NGINX 1.27 and you are updating it to NGINX 1.28.

**Tasks**
1. Identify the image tag setting in `/root/work/webapp/values.yaml`.
2. Explain why changing only `appVersion` is not enough.
3. Explain why the chart version should also increase when the chart's default image changes.

For a real change, update the image tag in `values.yaml`, set `appVersion` to the accurate app version, and increment the chart `version` according to your team's versioning policy. Do not assume an image tag such as `1.28` is valid for every image; verify the tag used by your project.

## Lab 4 — Compare local metadata with installed release metadata

Run:

```bash
helm show chart /root/work/webapp
helm list -n default
helm status webapp -n default
```

**Tasks**
1. Compare the chart version in the local directory with the chart version reported for the installed release.
2. Explain why editing `Chart.yaml` alone does not update the release already installed in the cluster.
3. Identify which Helm operation is used to apply a new chart version to an existing release: `helm upgrade`.

Do not run an upgrade on a shared environment just for this exercise.

## Lab 5 — Optional: validate a new chart package version

If you changed the chart metadata in a practice copy, run:

```bash
helm lint /root/work/webapp
helm template webapp /root/work/webapp
helm show chart /root/work/webapp
```

These commands validate the chart, render the manifests locally, and show the chart metadata. They do not update the installed release.

## Troubleshooting

- **`helm list` shows an older chart version:** The installed release still uses the version that was installed or upgraded. Local edits alone do not change it.
- **`appVersion` changed but the image did not:** `appVersion` is informational; inspect `values.yaml` and `templates/deployment.yaml`.
- **Not sure which field to bump:** Ask whether the chart packaging/configuration changed, the application changed, or both.
- **SemVer policy is unclear:** Follow the versioning policy used by your team or chart repository.

## Evidence checklist

- [ ] Read the local chart metadata.
- [ ] Compared `CHART` and `APP VERSION` in `helm list`.
- [ ] Explained a chart-only change.
- [ ] Explained an application image change.
- [ ] Understand that `helm upgrade` is needed to apply chart changes to an installed release.

## Reflection questions

1. What does the chart `version` represent?
2. What does `appVersion` represent?
3. Does changing `appVersion` change the running container image automatically?
4. Why do application image changes normally require a chart version bump too?
5. Why might local chart metadata differ from the installed release?

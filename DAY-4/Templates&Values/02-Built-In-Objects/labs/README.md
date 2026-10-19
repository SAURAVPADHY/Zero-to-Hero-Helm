# Day 4 — Chapter 2: Hands-on Labs — Helm Built-in Objects

**Difficulty:** Beginner  
**Estimated time:** 35–50 minutes  
**Goal:** Use `.Release`, `.Chart`, and `.Values` in a template, install a ConfigMap, and render the chart locally with a different release name.

## Prerequisites

- Helm and `kubectl` installed
- Chart directory: `/root/work/webapp`
- Practice cluster access
- Existing `webapp` release in `default`, or permission to install it

> **Safety:** Work in your practice environment. `helm upgrade --install` can modify an existing release named `webapp`.

## Lab 1 — Inspect the chart and release context

Run:

```bash
cat /root/work/webapp/Chart.yaml
cat /root/work/webapp/values.yaml
helm list -n default
```

Record the chart name, chart version, appVersion, and release name.

## Lab 2 — Create the ConfigMap template

Create or edit the file:

```bash
vi /root/work/webapp/templates/configmap.yaml
```

Enter:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-info
data:
  chart: {{ .Chart.Name | quote }}
  appVersion: {{ .Chart.AppVersion | quote }}
```

Save and exit with `:wq`.

**What each expression does**
- `.Release.Name` uses the release name.
- `.Chart.Name` reads the chart name from chart metadata.
- `.Chart.AppVersion` reads the app version.
- `quote` ensures the rendered values are strings.

## Lab 3 — Render before applying

Run:

```bash
helm template webapp /root/work/webapp
```

Inspect the rendered ConfigMap. It should have the name `webapp-info`, and its `data` should contain the chart name and app version.

To save the output:

```bash
mkdir -p /root/answers
helm template webapp /root/work/webapp \
  > /root/answers/webapp-rendered.yaml
```

Inspect the relevant section:

```bash
grep -n -A8 'name: webapp-info' /root/answers/webapp-rendered.yaml
```

Do not proceed until the rendered YAML looks correct.

## Lab 4 — Install or upgrade the release

Run:

```bash
helm upgrade --install webapp /root/work/webapp -n default
```

Wait for the chart's Deployment to roll out if needed, then verify the ConfigMap:

```bash
kubectl get configmap webapp-info -n default -o yaml
```

Expected data should include:

```yaml
data:
  chart: "webapp"
  appVersion: "1.1"
```

The exact appVersion depends on `/root/work/webapp/Chart.yaml`.

## Lab 5 — Render with `preview`, without installing

Render the chart using a different release name:

```bash
helm template preview /root/work/webapp \
  > /root/answers/rendered.yaml
```

Check for the generated ConfigMap name:

```bash
grep -n -A8 'name: preview-info' /root/answers/rendered.yaml
```

Now verify that no `preview` release was installed:

```bash
helm list -A
```

`helm template` only renders locally. It does not create a Helm release.

## Lab 6 — Optional: test values and chart metadata

Render the chart again:

```bash
helm template preview /root/work/webapp \
  --set replicaCount=3
```

Inspect the Deployment replicas. If the template uses `.Values.replicaCount`, the rendered value should be `3`.

## Troubleshooting

- **ConfigMap not found:** Confirm the template file is under `/root/work/webapp/templates/`, then rerun the upgrade command.
- **Wrong name:** Check that the metadata name uses `{{ .Release.Name }}-info`.
- **App version appears unquoted:** Ensure the template uses `| quote`.
- **`helm template` fails:** Inspect YAML indentation and template syntax.
- **Release already exists:** `helm upgrade --install` is designed to upgrade an existing release or install it if absent; verify the target namespace before running it.

## Evidence checklist

- [ ] Created `templates/configmap.yaml`.
- [ ] Used `.Release.Name` for the resource name.
- [ ] Used `.Chart.Name` and `.Chart.AppVersion` for ConfigMap data.
- [ ] Quoted string values.
- [ ] Rendered the chart and saved `/root/answers/rendered.yaml`.
- [ ] Installed/upgraded `webapp` and verified `webapp-info`.
- [ ] Confirmed the local `preview` render did not install a release.

## Reflection questions

1. What does the root dot (`.`) represent in a Helm template?
2. What is the difference between `.Chart.AppVersion` and `.Values.image.tag`?
3. Why should ConfigMap data values be strings?
4. How does `.Release.Name` make a chart reusable?
5. Why should you render a chart before upgrading a release?

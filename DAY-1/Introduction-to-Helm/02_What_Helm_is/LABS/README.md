# Lab 2: Build Your First Helm Chart

**Chapter:** 2 — What Is Helm?  
**Difficulty:** Beginner  
**Estimated time:** 45–60 minutes  
**Goal:** Convert a simple Kubernetes application into a Helm chart and install it as a named release.

## Scenario

In Lab 1, you created Kubernetes resources by applying separate YAML files. Now your team wants to package the web application so it can be installed, configured, and managed using Helm.

In this lab, you will build a small chart for NGINX with a Deployment and a Service.

## Prerequisites

- A working Kubernetes cluster
- `kubectl` configured to access the cluster
- Helm 3 or later installed

Check the tools:

```bash
kubectl get nodes
helm version
```

## 1. Create the project

```bash
mkdir -p lab-02-first-helm-chart
cd lab-02-first-helm-chart
helm create hello-web
cd hello-web
```

Helm creates a starter chart with sample templates. Explore it:

```bash
find . -maxdepth 2 -type f
```

Expected layout (the generated chart may contain additional example files):

```text
hello-web/
├── Chart.yaml
├── values.yaml
├── charts/
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── serviceaccount.yaml
    ├── ingress.yaml
    ├── hpa.yaml
    └── ...
```

## 2. Understand the key files

- `Chart.yaml`: chart name, version, and metadata.
- `values.yaml`: default settings used by templates.
- `templates/`: Kubernetes manifest templates that Helm renders.

The chart is a package description on disk. It does not create cluster resources until you install it.

## 3. Simplify the chart for this lab

To keep this first exercise focused, replace the contents of `values.yaml` with:

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.27"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  requests:
    cpu: 50m
    memory: 64Mi
  limits:
    cpu: 250m
    memory: 128Mi
```

Replace `templates/deployment.yaml` with:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  labels:
    app.kubernetes.io/name: {{ .Chart.Name }}
    app.kubernetes.io/instance: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app.kubernetes.io/instance: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app.kubernetes.io/instance: {{ .Release.Name }}
    spec:
      containers:
        - name: nginx
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

Replace `templates/service.yaml` with:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
  labels:
    app.kubernetes.io/name: {{ .Chart.Name }}
    app.kubernetes.io/instance: {{ .Release.Name }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app.kubernetes.io/instance: {{ .Release.Name }}
  ports:
    - name: http
      port: {{ .Values.service.port }}
      targetPort: http
```

For this exercise, remove the other generated files inside `templates/` so they do not introduce resources you are not studying yet. Keep `NOTES.txt` if you want, or remove it.

Your chart should now have these essential files:

```text
hello-web/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    └── service.yaml
```

## 4. Validate the chart

Run:

```bash
helm lint .
```

Expected: Helm reports that the chart passed linting.

Render the templates without installing anything:

```bash
helm template hello-web .
```

Check the rendered YAML. Confirm that:

- The Deployment has two replicas.
- The container image is `nginx:1.27`.
- The Service uses `ClusterIP`.
- The Deployment's Pod labels match the Service selector.

`helm template` renders manifests locally; it does not install resources in the cluster.

## 5. Install the chart

Install the chart into a dedicated namespace:

```bash
helm upgrade --install hello-web . \
  --namespace helm-lab \
  --create-namespace
```

Here, `hello-web` is the release name and `.` means the chart in the current directory.

Verify the release:

```bash
helm list -n helm-lab
helm status hello-web -n helm-lab
kubectl get deployments,services,pods -n helm-lab
```

Wait for the Deployment:

```bash
kubectl rollout status deployment/hello-web -n helm-lab
```

Expected: two replicas become available and the Service exists.

## 6. Test the application

Forward local port 8080 to the Service's port 80:

```bash
kubectl port-forward service/hello-web 8080:80 -n helm-lab
```

Keep that terminal running. In another terminal, run:

```bash
curl http://localhost:8080
```

Expected: the response contains the default NGINX welcome page HTML. Stop the port-forward with `Ctrl+C`.

## 7. Change configuration using Helm values

Create `values-small.yaml` in the chart directory:

```yaml
replicaCount: 1
```

Render with the override:

```bash
helm template hello-web . -f values-small.yaml
```

Confirm the rendered Deployment has one replica. Then apply the change:

```bash
helm upgrade hello-web . \
  --namespace helm-lab \
  -f values-small.yaml
```

Verify:

```bash
kubectl get deployment hello-web -n helm-lab
helm history hello-web -n helm-lab
```

This demonstrates how the same chart can use different configuration values without copying the templates.

## 8. Acceptance criteria

| Requirement | Expected result |
|---|---|
| Chart structure | `Chart.yaml`, `values.yaml`, and templates exist |
| Linting | `helm lint .` succeeds |
| Rendering | `helm template` shows valid Kubernetes manifests |
| Installation | Release `hello-web` exists in `helm-lab` |
| Workload | Two replicas are available before the values override |
| Service | Service routes to the chart's Pods |
| Values override | Upgrade changes the desired replica count to one |
| Release history | `helm history` shows the release revisions |

## 9. Troubleshooting

```bash
helm status hello-web -n helm-lab
kubectl get pods -n helm-lab
kubectl describe deployment hello-web -n helm-lab
kubectl logs deployment/hello-web -n helm-lab
```

If `helm lint` fails, inspect indentation and template syntax. If Pods do not become ready, inspect Pod events and logs.

## 10. Clean up

Uninstall the release:

```bash
helm uninstall hello-web -n helm-lab
```

If the namespace was created only for this lab and contains no other work, remove it:

```bash
kubectl delete namespace helm-lab
```

Deleting a namespace removes every resource in it, so verify that it is safe to delete first.

## Reflection questions

1. What is the difference between `helm template` and `helm install`?
2. Which file defines the chart's default values?
3. What does the release name identify?
4. How did the values override change the replica count without editing the Deployment template?
5. What does Helm add on top of raw Kubernetes YAML?

## Optional challenge

Add a configurable `service.port` value and verify that changing it updates the rendered Service. Then run `helm lint .` and inspect the output of `helm template`.

**Next lab:** Install the same chart as two releases—staging and production—with different values.

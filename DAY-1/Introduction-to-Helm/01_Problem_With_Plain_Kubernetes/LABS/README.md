# Lab 1: Managing Kubernetes with Raw YAML

**Chapter:** 1 — Why Helm?  
**Difficulty:** Beginner  
**Estimated time:** 30–45 minutes  
**Goal:** Experience how Kubernetes resources are managed individually before learning how Helm groups them into one reusable package.

## Scenario

You are an SRE deploying a small web application. The app needs:

- A ConfigMap containing a welcome message
- A Deployment running two NGINX replicas
- A Service exposing the Deployment inside the cluster

For this lab, use plain Kubernetes YAML files and `kubectl`. Do **not** use Helm yet.

## Prerequisites

- A working Kubernetes cluster (for example, kind, minikube, or a training cluster)
- `kubectl` installed and configured
- Permission to create Deployments, Services, ConfigMaps, and Namespaces

Check your connection:

```bash
kubectl config current-context
kubectl get nodes
```

If these commands fail, fix your cluster/context before continuing.

## 1. Create the lab directory

```bash
mkdir -p lab-01-raw-yaml
cd lab-01-raw-yaml
```

Create these three files in the directory:

```text
lab-01-raw-yaml/
├── README.md
├── configmap.yaml
├── deployment.yaml
└── service.yaml
```

## 2. Create `configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-web-config
data:
  WELCOME_MESSAGE: "Hello from raw Kubernetes YAML!"
```

A ConfigMap stores non-sensitive configuration. Do not store passwords, tokens, or other secrets in a ConfigMap.

## 3. Create `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-web
  template:
    metadata:
      labels:
        app: hello-web
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - name: http
              containerPort: 80
          env:
            - name: WELCOME_MESSAGE
              valueFrom:
                configMapKeyRef:
                  name: hello-web-config
                  key: WELCOME_MESSAGE
          resources:
            requests:
              cpu: 50m
              memory: 64Mi
            limits:
              cpu: 250m
              memory: 128Mi
```

The Deployment asks Kubernetes to keep two replicas running. The environment variable demonstrates how a Pod can consume the ConfigMap value; the default NGINX page will not automatically display that message.

## 4. Create `service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-web
spec:
  type: ClusterIP
  selector:
    app: hello-web
  ports:
    - name: http
      port: 80
      targetPort: http
```

The Service selects Pods with the label `app: hello-web` and provides a stable in-cluster endpoint.

## 5. Validate the YAML before applying it

From the directory containing the three manifests, run:

```bash
kubectl apply --dry-run=client -f configmap.yaml
kubectl apply --dry-run=client -f deployment.yaml
kubectl apply --dry-run=client -f service.yaml
```

This is a client-side validation check, not a guarantee that every resource will be accepted by the cluster. You can also use server-side dry run if your cluster is reachable and your permissions allow it:

```bash
kubectl apply --dry-run=server -f configmap.yaml
kubectl apply --dry-run=server -f deployment.yaml
kubectl apply --dry-run=server -f service.yaml
```

## 6. Deploy each file individually

Create a dedicated namespace for the lab:

```bash
kubectl create namespace helm-lab
```

Apply each file separately:

```bash
kubectl apply -f configmap.yaml -n helm-lab
kubectl apply -f deployment.yaml -n helm-lab
kubectl apply -f service.yaml -n helm-lab
```

Notice that you need to remember all three files and apply each one.

Wait for the Deployment:

```bash
kubectl rollout status deployment/hello-web -n helm-lab
```

## 7. Verify the application

```bash
kubectl get configmap,deployment,service -n helm-lab
kubectl get pods -n helm-lab -o wide
kubectl describe deployment hello-web -n helm-lab
kubectl describe service hello-web -n helm-lab
```

Expected result:

- ConfigMap `hello-web-config` exists.
- Deployment `hello-web` has **2/2 available replicas**.
- Service `hello-web` exists on port 80.
- Two Pods have the label `app=hello-web`.

If the Pods are not ready, inspect them:

```bash
kubectl get pods -n helm-lab
kubectl describe pod <pod-name> -n helm-lab
kubectl logs <pod-name> -n helm-lab
```

Replace `<pod-name>` with an actual Pod name from `kubectl get pods`.

## 8. Test the Service

Start a temporary port-forward:

```bash
kubectl port-forward service/hello-web 8080:80 -n helm-lab
```

Keep that terminal running. In another terminal, test the endpoint:

```bash
curl http://localhost:8080
```

You should receive the default NGINX welcome page HTML. Stop port-forwarding with `Ctrl+C` when finished.

## 9. Observe the raw-YAML management problem

Answer these questions in your own words:

1. How many manifest files did you need to create?
2. How many separate `kubectl apply` commands did you run?
3. What would happen if you forgot to apply `configmap.yaml` before creating the Deployment?
4. If another application had ten more resources, how would you keep track of every file?
5. How would you make staging use one replica and production use five without duplicating or manually editing manifests?
6. How would you identify and remove every resource belonging to this application?

The point is not that YAML is bad. YAML is still how Kubernetes resources are described. The difficulty is managing many related resources and environment-specific settings as one application.

## 10. Clean up

Delete the resources individually, as you would have to do in this raw-YAML workflow:

```bash
kubectl delete -f service.yaml -n helm-lab
kubectl delete -f deployment.yaml -n helm-lab
kubectl delete -f configmap.yaml -n helm-lab
```

Confirm they are gone:

```bash
kubectl get configmap,deployment,service -n helm-lab
```

If `helm-lab` was created only for this lab and contains no other work, remove the namespace:

```bash
kubectl delete namespace helm-lab
```

Deleting the namespace deletes all resources inside it. Do not run that command if you are using the namespace for other exercises.

## Acceptance criteria

| Requirement | Expected result |
|---|---|
| ConfigMap | `hello-web-config` exists during the lab |
| Deployment | Two NGINX replicas become available |
| Service | `ClusterIP` Service selects the application Pods |
| Connectivity | Port-forward plus `curl` returns the NGINX welcome page |
| Understanding | You can explain why managing many manifests manually gets cumbersome |
| Cleanup | Lab resources are removed |

## Optional challenge

Add a second Deployment called `hello-worker` with one replica and a separate label. Apply it with another YAML file, verify it, and then delete it.

Then consider: if this application grew to 15–20 Kubernetes resources, what would make installation, configuration, and cleanup easier?

**Next chapter:** Learn how Helm packages related Kubernetes templates into a chart and installs them as a named release.

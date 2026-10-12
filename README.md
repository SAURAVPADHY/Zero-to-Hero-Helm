# Zero-to-Hero-Helm
Learn Helm From Scratch

## Helm Installation

Helm is the package manager for Kubernetes. It helps you define, install, and manage applications using charts.

The official Helm installation guide is available at https://helm.sh/docs/intro/install/. The project recommends either:

- installing a prebuilt binary release, or
- using the official install script

Package managers like Homebrew, Chocolatey, and winget are community-contributed options and are useful, but the official Helm project documents the binary and script methods as the primary installation paths.

### Prerequisites

Before installing Helm, ensure that:

- `kubectl` is installed and configured
- you have a Kubernetes cluster available
- you have internet access to download Helm
- if you are on WSL, install Helm inside your Linux environment

### Official installation methods

#### Option 1: Install from a binary release

This is the recommended method when you want a direct install from the official Helm project.

1. Go to the Helm GitHub releases page:
   https://github.com/helm/helm/releases
2. Download the appropriate tarball for your OS and architecture.
3. Unpack the archive:

```bash
tar -zxvf helm-v4.0.0-linux-amd64.tar.gz
```

4. Move the Helm binary to a location in your `PATH`:

```bash
sudo mv linux-amd64/helm /usr/local/bin/helm
```

5. Confirm the binary is available:

```bash
helm version
```

> The official Helm docs also recommend verifying the downloaded binary and checking the checksum/signature before use.

#### Option 2: Install using the official script

This is the easiest way to install Helm on Linux or macOS.

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh
```

You can also run the script directly:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4 | bash
```

This script downloads the latest Helm release and installs it locally.

### Community package manager installs

These are supported by the wider Helm community, not the Helm project itself.

#### macOS (Homebrew)

```bash
brew install helm
```

#### Windows (winget)

```powershell
winget install Helm.Helm
```

#### Windows (Chocolatey)

```powershell
choco install kubernetes-helm
```

#### Windows (Scoop)

```powershell
scoop install helm
```

#### Debian / Ubuntu (APT)

The Helm project does not maintain native OS packages itself, but community packages are available.

```bash
curl -fsSL https://packages.buildkite.com/helm-linux/helm-debian/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/helm.gpg
sudo apt-get install -y apt-transport-https
sudo sh -c 'echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://packages.buildkite.com/helm-linux/helm-debian/any/ any main" > /etc/apt/sources.list.d/helm-stable-debian.list'
sudo apt-get update
sudo apt-get install helm
```

#### Fedora / RPM-based systems

```bash
sudo dnf install helm
```

### Verify the installation

After installation, run:

```bash
helm version
```

You should see output similar to:

```bash
version.BuildInfo{Version:"v4.x.x", GitCommit:"...", GitTreeState:"clean", GoVersion:"..."}
```

If Helm is installed correctly, you can also check:

```bash
helm help
```

### Add a chart repository

Helm needs a chart repository to discover software packages.

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Then search for charts:

```bash
helm search repo bitnami
```

### Quick cluster validation

Before installing apps with Helm, confirm your Kubernetes cluster is reachable:

```bash
kubectl get nodes
```

If this returns cluster information, Helm is ready to use.

### Common Helm commands

```bash
helm create mychart
helm install myapp bitnami/nginx
helm list
helm status myapp
helm uninstall myapp
```

### Troubleshooting

- If `helm` is not found, make sure the binary is in your `PATH`.
- If `helm version` fails, reinstall Helm or reopen your shell.
- If `kubectl` cannot connect, verify the cluster is running and your kubeconfig is valid.
- If a repo command fails, run `helm repo update` and check your network connection.

### Next step

Once Helm is installed and the CLI works, the next step is to begin deploying charts and managing Kubernetes applications in this repository.

---

This repository is part of a hands-on learning path for Helm and Kubernetes.

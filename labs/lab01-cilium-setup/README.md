
# Lab1 - Cilium setup with a single kubernetes cluster

## Objective

Setup a single kubernetes cluster using kind and install cilium as CNI.

## System requirements

- Host with either AMD64 or AArch64 architecture
- Linux kernel >= 4.19.57 or equivalent

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

## Step1: Create kubernetes cluster

Kind is a tool for running local Kubernetes clusters using Docker container “nodes”. It was primarily designed for testing Kubernetes itself. We will use it to setup a kubernetes cluster.

Check if docker, kind and kubectl are installed correctly
```bash
docker version --format '{{.Server.Version}}'
kind version
kubectl version --short
```
<details>
<summary>The output is similar to:</summary>

```console
24.0.2
kind v0.20.0 go1.20.4 linux/amd64
Client Version: v1.26.1
Kustomize Version: v4.5.7
Server Version: v1.27.3
```
</details>

Create a configuration file `kind-config.yaml`
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
networking:
  disableDefaultCNI: true
  kubeProxyMode: none
```

Create a kubernetes cluster using kind
```bash
kind create cluster --config kind-config.yaml
```

<details>
<summary>The output is similar to:</summary>

```console
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
 ✓ Preparing nodes 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Thanks for using kind!
```
</details>

Check if the docker containers are created correctly
```bash
docker ps
```

<details>
<summary>The output is similar to:</summary>

```console
CONTAINER ID   IMAGE                  COMMAND                  CREATED         STATUS         PORTS                       NAMES
f5436c13657c   kindest/node:v1.27.3   "/usr/local/bin/entr…"   6 minutes ago   Up 6 minutes   127.0.0.1:41191->6443/tcp   kind-control-plane
1ccae43eca31   kindest/node:v1.27.3   "/usr/local/bin/entr…"   6 minutes ago   Up 6 minutes                               kind-worker
```
</details>

Check if the cluster is created correctly
```bash
kubectl get nodes
```

<details>
<summary>The output is similar to:</summary>

```console
NAME                 STATUS     ROLES           AGE     VERSION
kind-control-plane   NotReady   control-plane   2m46s   v1.27.3
kind-worker          NotReady   <none>          2m22s   v1.27.3
```
</details>

> Note: The nodes are not ready because we haven't installed any CNI yet. We will install cilium as CNI in the next step.

## Step2: Install cilium

Cilium is a CNI plugin that supports dynamic insertion of BPF bytecode into the Linux kernel. It can be used with Kubernetes in order to provide networking and security services.

First we need to install the cilium CLI, you can find the related documentation [Install the Cilium CLI](https://docs.cilium.io/en/v1.14/gettingstarted/k8s-install-default/#install-the-cilium-cli). Please follow the instructions for your operating system.

Check if the cilium CLI is installed correctly
```bash
cilium version --client
```

<details>
<summary>The output is similar to:</summary>

```console
cilium-cli: v0.15.0 compiled with go1.20.4 on linux/amd64
cilium image (default): v1.13.4
cilium image (stable): v1.14.0
```
</details>

Install Cilium into the Kubernetes cluster
```bash
cilium install --version 1.14.0
```

<details>
<summary>The output is similar to:</summary>

```console
🔮 Auto-detected Kubernetes kind: kind
✨ Running "kind" validation checks
✅ Detected kind version "0.20.0"
ℹ️  Using Cilium version 1.14.0
🔮 Auto-detected cluster name: kind-kind
ℹ️  kube-proxy-replacement disabled
🔮 Auto-detected datapath mode: tunnel
ℹ️  Detecting real Kubernetes API server addr and port on Kind
🔮 Auto-detected kube-proxy has not been installed
ℹ️  Cilium will fully replace all functionalities of kube-proxy
```
</details>

Validate the installation
```bash
cilium status --wait
```
> Note: This process may take a few minutes.

<details>
<summary>The output is similar to:</summary>

```console
    /¯¯\
 /¯¯\__/¯¯\    Cilium:             OK
 \__/¯¯\__/    Operator:           OK
 /¯¯\__/¯¯\    Envoy DaemonSet:    disabled (using embedded mode)
 \__/¯¯\__/    Hubble Relay:       disabled
    \__/       ClusterMesh:        disabled

Deployment             cilium-operator    Desired: 1, Ready: 1/1, Available: 1/1
DaemonSet              cilium             Desired: 2, Ready: 2/2, Available: 2/2
Containers:            cilium-operator    Running: 1
                       cilium             Running: 2
Cluster Pods:          3/3 managed by Cilium
Helm chart version:    1.14.0
Image versions         cilium-operator    quay.io/cilium/operator-generic:v1.14.0@sha256:3014d4bcb8352f0ddef90fa3b5eb1bbf179b91024813a90a0066eb4517ba93c9: 1
                       cilium             quay.io/cilium/cilium:v1.14.0@sha256:5a94b561f4651fcfd85970a50bc78b201cfbd6e2ab1a03848eab25a82832653a: 2
```
</details>

Check if the nodes are ready
```bash
kubectl get nodes
```

<details>
<summary>The output is similar to:</summary>

```console
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   21m   v1.27.3
kind-worker          Ready    <none>          20m   v1.27.3
```
</details>


## Conclusion

In this tutorial we have installed a Kubernetes cluster using kind and installed cilium as CNI plugin.

## References

- [Cilium Quick Installation](https://docs.cilium.io/en/v1.14/gettingstarted/k8s-install-default)
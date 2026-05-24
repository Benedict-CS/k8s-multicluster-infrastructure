# Lab8 - Karmada Setup

## Objectives

- Create the management cluster
- Deploy Karmada control plane
- Setup the workload clusters

## Prerequisites

- Environment from [Lab 7](../lab07-multi-cluster-setup/README.md)


## Overview

[karmada](https://karmada.io/) is a Kubernetes-based multi-cloud and multi-cluster orchestration platform. It provides central management of cloud-native workloads and applications, and enables workload and application portability across clouds and clusters.

![Karmada concept](./../assets/lab8-karmada-concept.png)


In this lab, we will create the control cluster and deploy Karmada control plane.


## Step1: Create the control cluster and prepare enviroment

You can use any kubernetes cluster as the control cluster (e.g. EKS, GKE, AKS, etc.). In our lab, we will use [kind](https://kind.sigs.k8s.io/) to create the control cluster.

> Note: we will use this cluster after the lab. Use kind is just for convenience. If you want to prepare the production environment, please use the cloud provider or other tools to create the cluster.

Create a Kubernetes cluster with kind
```bash
kind create cluster --name control-cluster
```

<details>
<summary>The output is similar to:</summary>

```bash
Creating cluster "control-cluster" ...
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-control-cluster"
You can now use your cluster with:

kubectl cluster-info --context kind-control-cluster

Thanks for using kind!
```
</details>


Next, we need to prepare the kubeconfig for the other cluster we want to register to the Karmada control plane. In previous lab, we have created the two clusters. Please copy the kubeconfig to the `~/.kube/` folder.

```bash
# tree ~/.kube
.kube
├── cluster1.config
├── cluster2.config
└── config
```
> Note: cluster1.config and cluster2.config are the kubeconfig of the member clusters.

## Step2: Deploy Karmada control plane

We can use karmadactl to deploy the Karmada control plane. Please follow the [Install Karmadactl](https://karmada.io/docs/installation/install-cli-tools#install-karmadactl) to install the Karmada CLI.

Check the Karmada version
```bash
karmadactl version
```

<details>
<summary>The output is similar to:</summary>

```console
karmadactl version: version.Info{GitVersion:"v1.6.2", GitCommit:"1cfe7250e90592f46c17cf96955d4d55a55853c4", GitTreeState:"clean", BuildDate:"2023-08-03T08:11:50Z", GoVersion:"go1.20.4", Compiler:"gc", Platform:"linux/amd64"}
```
</details>

Check the kubectl current context
```bash
kubectl config get-contexts
```

<details>
<summary>The output is similar to:</summary>

```console
CURRENT   NAME                   CLUSTER                AUTHINFO               NAMESPACE
          cluster1               cluster1               cluster1
          cluster2               cluster2               cluster2
*         kind-control-cluster   kind-control-cluster   kind-control-cluster
```
</details>

> Note: The current context should be the control cluster which we just use kind to create.

Deploy Karmada control plane
```bash
sudo karmadactl init --kubeconfig ~/.kube/config
```

<details>
<summary>The output is similar to:</summary>

```console
Karmada is installed successfully.

Register Kubernetes cluster to Karmada control plane.

Register cluster with 'Push' mode

Step 1: Use "karmadactl join" command to register the cluster to Karmada control plane. --cluster-kubeconfig is kubeconfig of the member cluster.
(In karmada)~# MEMBER_CLUSTER_NAME=$(cat ~/.kube/config  | grep current-context | sed 's/: /\n/g'| sed '1d')
(In karmada)~# karmadactl --kubeconfig /etc/karmada/karmada-apiserver.config  join ${MEMBER_CLUSTER_NAME} --cluster-kubeconfig=$HOME/.kube/config

Step 2: Show members of karmada
(In karmada)~# kubectl --kubeconfig /etc/karmada/karmada-apiserver.config get clusters


Register cluster with 'Pull' mode

Step 1: Use "karmadactl register" command to register the cluster to Karmada control plane. "--cluster-name" is set to cluster of current-context by default.
(In member cluster)~# karmadactl register 172.20.0.2:32443 --token jttbb2.v4a73ptkad764o3s --discovery-token-ca-cert-hash sha256:4390cb1345a7748b746aa6b7eb2a7dabf973f379e16360e0a6a5dd052c6991a2

Step 2: Show members of karmada
(In karmada)~# kubectl --kubeconfig /etc/karmada/karmada-apiserver.
```
</details>

Now you can use kubeconfig stored in `/etc/karmada/karmada-apiserver.config` to access the Karmada control plane.

> Note: Karmada will create the apiserver in the control cluster. If you want to interact with the Karmada control plane, you need to use the kubeconfig of the control cluster.

## Step3: Register the member clusters

Now we can register the member clusters to the Karmada control plane. We can use `karmadactl join <cluster>` to register the member clusters.

Copy your member clusters kubeconfig to /tmp
```bash
cp ~/.kube/cluster1.config /tmp
cp ~/.kube/cluster2.config /tmp
```

Register the member clusters
```bash
karmadactl --kubeconfig /etc/karmada/karmada-apiserver.config join cluster1 --cluster-kubeconfig=/tmp/cluster1.config
karmadactl --kubeconfig /etc/karmada/karmada-apiserver.config join cluster2 --cluster-kubeconfig=/tmp/cluster2.config
```

<details>
<summary>The output is similar to:</summary>

```console
cluster(cluster1) is joined successfully
cluster(cluster2) is joined successfully
```
</details>

Check the status of member clusters
```bash
kubectl --kubeconfig /etc/karmada/karmada-apiserver.config get clusters
```

<details>
<summary>The output is similar to:</summary>

```console
NAME       VERSION        MODE   READY   AGE
cluster1   v1.27.4+k3s1   Push   True    34s
cluster2   v1.27.4+k3s1   Push   True    30s
```
</details>

> If the status is `Ready`, it means the member cluster is registered successfully.


## Conclusion

In this lab, we have created the control cluster and deployed Karmada control plane. We also registered the member clusters to the Karmada control plane.

## References

- [Karmada Installation](https://karmada.io/docs/installation/)
- [Installation of CLI Tools](https://karmada.io/docs/installation/install-cli-tools#install-karmadactl)
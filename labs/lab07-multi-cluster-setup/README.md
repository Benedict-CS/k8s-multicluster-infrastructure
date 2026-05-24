# Lab7 - Multi Cluster Setup

## Objectives

- Learn how to create production ready clusters
- Learn how to install cilium on the production ready clusters
- Use K3s to create the clusters

## Overview

In this lab, we will learn how to create production ready clusters, and use K3s to create the clusters. Install cilium on the clusters.

## Requirements of the Cluster

In the previous labs we learned how to install cilium in kind cluster, but in the production environment, we usually use cloud provider (AWS, GCP, Azure, etc.) or bare metal server to create the cluster.

To install Cilium with the gateway API and Cluster Mesh enabled, the cluster must meet the following requirements:

- Cilium requirments
  - Create a cluster without default CNI
- Cilium Gateway requirments
  - kube-proxy replacement mode must be true
  - L7 proxy enabled using the `--enable-l7-proxy` flag (enabled by default)
  - CRDs from Gateway API v0.7.0 `must` be pre-installed
> Ref: [Cilium Gateway API Support](https://docs.cilium.io/en/v1.14/network/servicemesh/gateway-api/gateway-api/#prerequisites)
- Cluster Mesh Requirement
  - Worker nodes in all clusters must be able to communicate with each other
  - PodCIDRs must not conflict across clusters
  - Firewall and networking security rules must allow for client access to ClusterMesh API Services across all clusters
> Ref: [Setting up Cluster Mesh](https://docs.cilium.io/en/latest/network/clustermesh/clustermesh/#cluster-addressing-requirements)

- Here is the example of how to install cilium on various cloud provider.
  - Ref: [Cilium Quick Installation](https://docs.cilium.io/en/v1.14/gettingstarted/k8s-install-default/)
- Here is the example of how to get the gateway API support
  - Ref: [Gateway API Support](https://docs.cilium.io/en/v1.14/network/servicemesh/gateway-api/gateway-api/)
- Here is the example of how to prepare AKS to meet the requirements of the Cluster Mesh
  - Ref: [AKS-to-AKS Clustermesh Preparation](https://docs.cilium.io/en/latest/network/clustermesh/aks-clustermesh-prep/)

You need to mix the above information to create your own cluster. I prepare the checklist for you to follow.

- Create the clusters
  - [ ] Create a cluster without default CNI
  - [ ] PodCIDRS and ServiceCIDRs must not conflict across clusters
  - [ ] Check the node connection between clusters
  - [ ] Check the firewall and networking security rules
- Install Cilium
  - [ ] CRDs from Gateway API v0.7.0 `must` be pre-installed
  - [ ] kube-proxy replacement mode must be true
  - [ ] L7 proxy enabled using the `--enable-l7-proxy` flag (enabled by default)
  - [ ] Set unique cluster.id and cluster.name for each cluster

Now we are go through the installation steps with K3s

## Step1 - Create new clusters by K3s

K3s is a lightweight Kubernetes distribution by Rancher. It is a fully compliant Kubernetes distribution. We can use it to create new clusters.

First, we need to create two VMs, and ssh on one of the VM. In VM1, Set the environment variable for the node1 ip address.

```bash
export NODE1_IP=<YOUR_VM1_IP>
```

Now you can install K3s on the VM1 using the following command.

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC='--flannel-backend=none' sh -s - \
            --node-ip $NODE1_IP \
            --disable-network-policy \
            --disable-kube-proxy \
            --disable traefik \
            --disable servicelb \
            --disable metrics-server
```
> Note: We need to disable the default CNI, and kube-proxy to match the requirements of Cilium.


Check if the cluster is created correctly
```bash
sudo kubectl get nodes
```
<details>
<summary>The output is similar to:</summary>

```console
NAME       STATUS     ROLES                  AGE     VERSION
cluster1   NotReady   control-plane,master   3m33s   v1.27.4+k3s1
```
</details>

> Note: The node are not ready because we haven't installed any CNI yet.

When the installation is done, The kubeconfig file will be generated in `/etc/rancher/k3s/k3s.yaml`. You can check the content of the file


```bash
sudo cat /etc/rancher/k3s/k3s.yaml
```

Copy the content of the file to your local machine, and save it as `cluster1.config`.


Now ssh on the VM2, and go through the same steps to install K3s on the VM2. and save the kubeconfig file as `cluster2.config`.


## Step2: Merge kubeconfig files

Prepare a new bastion server, and put the cluster1 and cluster2 kubeconfig files in the ~/.kube directory. and change some values in the kubeconfig file.

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: <base64 encoded data>
    server: https://192.168.50.101:6443 # Change the IP address to the VM1 IP address (Default: 127.0.0.1:6443)
  name: cluster1 # Change the cluster name to the cluster1
contexts:
- context:
    cluster: cluster1 # Change the cluster name to the cluster1
    user: cluster1 # Change the user name to the cluster1
  name: cluster1 # Change the context name to the cluster1
current-context: cluster1 # Change the current context name to the cluster1
kind: Config
preferences: {}
users:
- name: cluster1 # Change the user name to the cluster1
  user:
    client-certificate-data: <base64 encoded data>
    client-key-data: <base64 encoded data>
```
> Note: Need to change the cluster name from default to cluster1 or cluster to avoid conflict.

> Note: Change the IP address in the kubeconfig file to the VM1 or VM2 IP address.

The directory structure is similar to:

```bash
# tree ~/.kube
.kube
├── cluster1.config
├── cluster2.config
```

Merge the kubeconfig files into one.

```bash
export KUBECONFIG=~/.kube/config:$(find . -type f | tr '\n' ':')
kubectl config view --flatten > ~/.kube/config
```

Check the context.

```bash
kubectl config get-contexts
```

<details>
<summary>The output is similar to:</summary>

```bash
CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
*         cluster1   cluster1   cluster1
          cluster2   cluster2   cluster2
```
</details>

Set the environment variable for the cluster name for the next step.

```bash
export CLUSTER1=cluster1
export CLUSTER2=cluster2
```

## Step3: Install cilium

Before we install cilium, we need to install the CRDs from Gateway API v0.7.0 to match the requirements of Cilium.

```bash
# Install CRDs from Gateway API v0.7.0 on cluster1
kubectl apply --context $CLUSTER1 -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v0.7.0/config/crd/standard/gateway.networking.k8s.io_gatewayclasses.yaml
kubectl apply --context $CLUSTER1 -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v0.7.0/config/crd/standard/gateway.networking.k8s.io_gateways.yaml
kubectl apply --context $CLUSTER1 -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v0.7.0/config/crd/standard/gateway.networking.k8s.io_httproutes.yaml
kubectl apply --context $CLUSTER1 -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v0.7.0/config/crd/standard/gateway.networking.k8s.io_referencegrants.yaml
kubectl apply --context $CLUSTER1 -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v0.7.0/config/crd/experimental/gateway.networking.k8s.io_tlsroutes.yaml
# Install CRDs from Gateway API v0.7.0 on cluster2
kubectl apply --context $CLUSTER2 -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v0.7.0/config/crd/standard/gateway.networking.k8s.io_gatewayclasses.yaml
kubectl apply --context $CLUSTER2 -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v0.7.0/config/crd/standard/gateway.networking.k8s.io_gateways.yaml
kubectl apply --context $CLUSTER2 -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v0.7.0/config/crd/standard/gateway.networking.k8s.io_httproutes.yaml
kubectl apply --context $CLUSTER2 -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v0.7.0/config/crd/standard/gateway.networking.k8s.io_referencegrants.yaml
kubectl apply --context $CLUSTER2 -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v0.7.0/config/crd/experimental/gateway.networking.k8s.io_tlsroutes.yaml
```

Set the environment variable for the node1 and node2 ip address.

```bash
export NODE1_IP=<YOUR_VM1_IP>
export NODE2_IP=<YOUR_VM2_IP>
```

Install cilium on the cluster1 by cilium CLI.

```bash
cilium install --version 1.14.0 --context $CLUSTER1 \
--set kubeProxyReplacement=true \
--set gatewayAPI.enabled=true \
--set k8sServiceHost=$NODE1_IP \
--set k8sServicePort=6443 \
--set ipam.operator.clusterPoolIPv4PodCIDRList="10.45.0.0/16" \
--set cluster.id=1 \
--set cluster.name=cluster1
```
> Note: We need to set the unique cluster.id, cluster.name and ipam.operator.clusterPoolIPv4PodCIDRListfor each cluster.

Before install cilium on cluster2, we need to copy the cilium-ca secret from cluster1 to cluster2.

```bash
kubectl --context=$CLUSTER1 get secret -n kube-system cilium-ca -o yaml | \
  kubectl --context $CLUSTER2 create -f -
```
> Ref: [Setting up Cluster Mesh](https://docs.cilium.io/en/v1.14/network/clustermesh/clustermesh/#shared-certificate-authority)

Install cilium on the cluster2 by cilium CLI.

```bash
cilium install --version 1.14.0 --context $CLUSTER2 \
--set kubeProxyReplacement=true \
--set gatewayAPI.enabled=true \
--set k8sServiceHost=$NODE2_IP \
--set k8sServicePort=6443 \
--set ipam.operator.clusterPoolIPv4PodCIDRList="10.46.0.0/16" \
--set cluster.id=2 \
--set cluster.name=cluster2
```
> Note: We need to set the unique cluster.id, cluster.name and ipam.operator.clusterPoolIPv4PodCIDRListfor each cluster.


Check the cilium status on the cluster1 and cluster2.

```bash
cilium status --context $CLUSTER1 --wait
cilium status --context $CLUSTER2 --wait
```

If the cilium status is `HEALTHY`, then the cilium is installed successfully.

## Step4: Enable Cluster Mesh

To enable the cluster mesh. Please follow the step3 in [Lab 6](../lab06-cluster-mesh/README.md) to do it.

## Step5: Create Cilium Gateway

To Create Cilium Gatway. Please follow the step2 and step3 in [Lab 2](../lab02-cilium-gateway-api/README.md) to do it.
> Note: In Step2. The allocate IP address range are determined by the IPPool where VM1 and VM2 are located. 
 
> Note: If you use cloud provider, you can skip the step2 in lab2

Finally, check the gateway status on the cluster1 and cluster2.

```bash
kubectl --context $CLUSTER1 get gateway
kubectl --context $CLUSTER2 get gateway
```

<details>
<summary>The output is similar to:</summary>

```bash
# Cluster1 gateway status
NAME             CLASS    ADDRESS          PROGRAMMED   AGE
cilium-gateway   cilium   192.168.50.200   True         32s
# Cluster2 gateway status
NAME             CLASS    ADDRESS          PROGRAMMED   AGE
cilium-gateway   cilium   192.168.50.210   True         31s
```
</details>

> Note: We will use the gateway IP address to access the service in the future.

## Conclusion

In this lab, we learned how to create production ready clusters, and use K3s to create the clusters. Install cilium on the clusters.

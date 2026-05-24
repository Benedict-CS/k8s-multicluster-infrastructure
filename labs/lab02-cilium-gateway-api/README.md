# Lab2 - Cilium Gateway API Setup

## Objectives

- Setup Cilium Gateway API
- Setup MetalLB

## Prerequisites

- Environment from [Lab 1](../lab01-cilium-setup/README.md)
- [helm](https://helm.sh/docs/intro/install/)

## Overview

In lab1 we have setup a single kubernetes cluster with Cilium as CNI. In this lab, we will setup Cilium Gateway API.

Cilium Gateway API is a new API that allows users to configure L3/L4 routing and load balancing for services. To learn more about Cilium Gateway API, please refer to [A Deep Dive into Cilium Gateway API: The Future of Ingress Traffic Routing](https://isovalent.com/blog/post/cilium-gateway-api/).

![Cilium Gateway API](./../assets/lab2-cilium-gateway-API.png)
> Reference: [Cilium release 1.13](https://isovalent.com/blog/post/cilium-release-113/)

## Step1: Upgrade Cilium with Gateway API support

To use gateway API, you need to meet the requirements from [Cilium Gateway API Prerequisites](https://docs.cilium.io/en/v1.14/network/servicemesh/gateway-api/gateway-api/#prerequisites)

Install CRDs for Cilium Gateway API
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v0.7.0/config/crd/standard/gateway.networking.k8s.io_gatewayclasses.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v0.7.0/config/crd/standard/gateway.networking.k8s.io_gateways.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v0.7.0/config/crd/standard/gateway.networking.k8s.io_httproutes.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v0.7.0/config/crd/standard/gateway.networking.k8s.io_referencegrants.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v0.7.0/config/crd/experimental/gateway.networking.k8s.io_tlsroutes.yaml
```

Next, we need to upgrade to Cilium with setting `gatewayAPI.enabled=true` and `kubeProxyReplacement=true`.

Helm is a package manager for Kubernetes. We can use helm to upgrade Cilium, check if helm is installed correctly
```bash
helm version
```
<details>
<summary>The output is similar to:</summary>

```console
version.BuildInfo{Version:"v3.11.1", GitCommit:"293b50c65d4d56187cd4e2f390f0ada46b4c4737", GitTreeState:"clean", GoVersion:"go1.18.10"}
```
</details>

Add the helm repo and update it.
```bash
helm repo add cilium https://helm.cilium.io
helm repo update
```

Upgrade Cilium with helm
```bash
helm upgrade cilium cilium/cilium --version 1.14.0 \
    --namespace kube-system \
    --reuse-values \
    --set kubeProxyReplacement=true \
    --set gatewayAPI.enabled=true
```

<details>
<summary>The output is similar to:</summary>

```console
Release "cilium" has been upgraded. Happy Helming!
NAME: cilium
LAST DEPLOYED: Sat Aug  5 15:58:50 2023
NAMESPACE: kube-system
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
You have successfully installed Cilium with Hubble.

Your release version is 1.14.0.

For any further help, visit https://docs.cilium.io/en/v1.14/gettinghelp
```
</details>

Restart the Cilium agent and operator pods
```bash
kubectl -n kube-system rollout restart deployment/cilium-operator
kubectl -n kube-system rollout restart ds/cilium
```

Validate the upgrade
```bash
cilium status --wait
```

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
Containers:            cilium             Running: 2
                       cilium-operator    Running: 1
Cluster Pods:          3/3 managed by Cilium
Helm chart version:    1.14.0
Image versions         cilium             quay.io/cilium/cilium:v1.14.0@sha256:5a94b561f4651fcfd85970a50bc78b201cfbd6e2ab1a03848eab25a82832653a: 2
                       cilium-operator    quay.io/cilium/operator-generic:v1.14.0@sha256:3014d4bcb8352f0ddef90fa3b5eb1bbf179b91024813a90a0066eb4517ba93c9: 1
```
</details>

Double check it was successfully set up
```bash
cilium config view | grep "enable-gateway-api"
```

<details>
<summary>The output is similar to:</summary>

```console
enable-gateway-api                                true
enable-gateway-api-secrets-sync                   true
```
</details>


## Step2: Install MetalLB

To access the service that will be exposed via the Gateway API, we need to allocate an external IP address. We will use MetalLB to allocate the IP address.

> Note: If you are using a cloud provider like AKS, GKE, EKS, etc. They usually has a load balancer service that can be used to allocate an external IP address. You can skip this step.

Install MetalLB with helm
```bash
helm repo add metallb https://metallb.github.io/metallb
helm repo update
helm install metallb metallb/metallb --version 0.13.10 \
  --namespace metallb-system --create-namespace
```

<details>
<summary>The output is similar to:</summary>

```console
NAME: metallb
LAST DEPLOYED: Sat Aug  5 16:36:30 2023
NAMESPACE: metallb-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
MetalLB is now running in the cluster.

Now you can configure it via its CRs. Please refer to the metallb official docs
on how to use the CRs.
```
</details>

Now we need to configure MetalLB to allocate IP addresses from the same subnet as the Kubernetes cluster.

Set parmeters for MetalLB
```bash
KIND_NET_CIDR=$(docker network inspect kind -f '{{(index .IPAM.Config 0).Subnet}}')
METALLB_IP_START=$(echo ${KIND_NET_CIDR} | sed "s@0.0/16@255.200@")
METALLB_IP_END=$(echo ${KIND_NET_CIDR} | sed "s@0.0/16@255.250@")
METALLB_IP_RANGE="${METALLB_IP_START}-${METALLB_IP_END}"
echo $METALLB_IP_RANGE
```

<details>
<summary>The output is similar to:</summary>

```console
172.21.255.200-172.21.255.250
```
</details>


Create a `metallb-setting.yaml` file with MetalLB configuration
```bash
cat <<EOF > metallb-setting.yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: example
  namespace: metallb-system
spec:
  addresses:
  - ${METALLB_IP_RANGE}
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: empty
  namespace: metallb-system
EOF
```
> Note: you can get more information about MetalLB configuration from [MetalLB Configuration](https://metallb.universe.tf/configuration/)

apply the configuration
```bash
kubectl apply -f metallb-setting.yaml
```

check if the configuration is applied correctly
```bash
kubectl get IPAddressPool -n metallb-system
kubectl get L2Advertisement -n metallb-system
```

<details>
<summary>The output is similar to:</summary>

```console
NAME      AGE
example   64s
NAME    AGE
empty   64s
```
</details>


## Step3: Install Cilium Gateway

Verify that a GatewayClass has been deployed and accepted
```bash
kubectl get gatewayclasses
```

<details>
<summary>The output is similar to:</summary>

```console
NAME     CONTROLLER                     ACCEPTED   AGE
cilium   io.cilium/gateway-controller   True       5m
```
</details>

Create a Gateway resource `cilium-gateway.yaml`
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: cilium-gateway
spec:
  gatewayClassName: cilium
  listeners:
  - protocol: HTTP
    port: 80
    name: web-gw
    allowedRoutes:
      namespaces:
        from: Same
```

Apply the Gateway resource
```bash
kubectl apply -f cilium-gateway.yaml
```

Check the service created by the Gateway
```bash
kubectl get svc
```

<details>
<summary>The output is similar to:</summary>

```console
NAME                            TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
cilium-gateway-cilium-gateway   LoadBalancer   10.96.166.32   172.21.255.200   80:32123/TCP   8m33s
kubernetes                      ClusterIP      10.96.0.1      <none>           443/TCP        3h4m
```
</details>

> Note: The external IP address is allocated by MetalLB

Check the Gateway resource
```bash
kubectl get gateway
```

<details>
<summary>The output is similar to:</summary>

```console
NAME             CLASS    ADDRESS          PROGRAMMED   AGE
cilium-gateway   cilium   172.21.255.200   True         11m
```
</details>

> Note: if the `PROGRAMMED` field is `False`, it means the Gateway is not programmed correctly. Check the logs of the `cilium-operator` pod for more information.

Now we can use Cilium Gateway API to expose a service. We will deploy a application in the next lab.

## Conclusion

In this tutorial, we have learned what is Cilium Gateway API and how to install it on a Kubernetes clusters.

## References

- [Cilium Gateway API](https://docs.cilium.io/en/v1.14/network/servicemesh/gateway-api/gateway-api)
- [A Deep Dive into Cilium Gateway API: The Future of Ingress Traffic Routing](https://isovalent.com/blog/post/cilium-gateway-api/)
- [Tutorial: Getting Started with the Cilium Gateway API](https://isovalent.com/blog/post/tutorial-getting-started-with-the-cilium-gateway-api/)
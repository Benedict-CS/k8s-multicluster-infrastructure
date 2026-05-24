# Lab12 - Network Policy

## Objectives

- Setup karmada to propagate Cilium network policy
- Deploy a network policy to set the only allow IP address to access the service
- Deploy a network policy to allow specific HTTP header to access the service

## Prerequisites

- Environment from [Lab 10](../lab10-haproxy-setup/README.md)

## Overview

If tou want to control traffic flow at the IP address or HTTP header level, you can use network policies. In this lab, we will setup karmada to propagate cilium network policy to allow the IP address and HTTP header to access the service.

## Step1: Environment Setup

In [lab8](../lab08-karmada-setup/README.md) we have setup the karmada cluster. In this lab, we will use it to propagate the network policy to the workload clusters.


First we have to set the environment variables for the karmada kubeconfig
```bash
export KARMADA_CONFIG=~/.kube/karmada-apiserver.config
```

Set the environment variables for workload clusters kubeconfig

```bash
export CLUSTER1=cluster1
export CLUSTER2=cluster2
```

> Note: cluster1 and cluster2 are the context name of the workload cluster stored in the `~/.kube/kubeconfig`.

To propagate the network policy to each workload clusters, we have to install the Cilium CRDs in the karmada cluster. Follow the [Propagate a CRD application with Karmada](https://karmada.io/docs/tutorials/crd-application/) to get more information.

Apply the CiliumNetworkPolicy CRD resource to the karmada cluster  
```bash
kubectl apply --kubeconfig $KARMADA_CONFIG -f https://raw.githubusercontent.com/cilium/cilium/main/pkg/k8s/apis/cilium.io/client/crds/v2/ciliumnetworkpolicies.yaml
```
> Note: Cilium CRDs files are stored in [Cilium crds](https://github.com/cilium/cilium/tree/main/pkg/k8s/apis/cilium.io/client/crds)

Then create a file named `network-policy-propagationpolicy.yaml`

```yaml
apiVersion: policy.karmada.io/v1alpha1
kind: PropagationPolicy
metadata:
  name: network-policy-propagation
spec:
  resourceSelectors:
    - apiVersion: cilium.io/v2
      kind: CiliumNetworkPolicy
      name: whoami-network-policy
  placement:
    clusterAffinity:
      clusterNames:
        - cluster1
        - cluster2
```

Apply the propagation policy to the karmada cluster

```bash
kubectl apply --kubeconfig $KARMADA_CONFIG -f network-policy-propagationpolicy.yaml
```

Check the propagation policy

```bash
kubectl get --kubeconfig $KARMADA_CONFIG propagationpolicy
```

<details>
<summary>The output is similar to:</summary>

```console
NAME                         AGE
network-policy-propagation   37s
whoami-propagation           17m
```
</details>

> Note: whoami-propagation is the propagation policy from [lab9](../lab09-simple-app-propagating/README.md)


## Step2: Deploy the Network Policy to allow the IP address

Cilium network policy is a Kubernetes network policy implementation of Cilium. To get more information, please refer to [Overview of Network Policy](https://docs.cilium.io/en/latest/security/policy/)

X-forward-for is a HTTP header used to identify the originating IP address of a client connecting to a web server through an HTTP proxy or a load balancer. In this lab, we will use it to identify the IP address of the client.


In [lab10](../lab10-haproxy-setup/README.md), we have setup the haproxy service. We will use it as the application endpoint

Set the environment variables for haproxy endpoint
```bash
export HAPROXY_ENDPOINT=<YOUR_HAPROXY_ENDPOINT>
echo $HAPROXY_ENDPOINT
```

<details>
<summary>The output is similar to:</summary>

```console
192.168.50.100:80
```
</details>


Now we can test the haproxy service. I use the server which ip is `192.168.50.100` to test the haproxy service. You can use any server to test it.

Use curl to send a request to the haproxy endpoint
```bash
curl $HAPROXY_ENDPOINT
```

<details>
<summary>The output is similar to:</summary>

```console
Hostname: whoami-deployment-6d54cbf86f-shj4z
IP: 127.0.0.1
IP: ::1
IP: 10.45.0.172
IP: fe80::20e4:8eff:fe77:e8cb
RemoteAddr: 10.45.0.58:35677
GET / HTTP/1.1
Host: 192.168.50.100
User-Agent: curl/7.81.0
Accept: */*
X-Envoy-External-Address: 192.168.50.100
X-Forwarded-For: 192.168.50.100,192.168.50.100
X-Forwarded-Proto: http
X-Request-Id: 69336f94-9bce-4bb7-ba1a-098753ccd20f
```
</details>

> Note: This information is from the whoami service. The first IP in X-Forwarded-For is the client IP address. The second IP is the IP address of the HAProxy. Because we use the same server to curl, so the two IP addresses are the same.

We provide another server to curl the haproxy endpoint. The IP address of my the server is `192.168.50.102`. You can use any server to test it.

```bash
export HAPROXY_ENDPOINT=<YOUR_HAPROXY_ENDPOINT>
curl -s $HAPROXY_ENDPOINT | grep X-Forwarded-For
```

<details>
<summary>The output is similar to:</summary>

```console
X-Forwarded-For: 192.168.50.102,192.168.50.100
```
</details>

> Note: You can see the first IP in X-Forwarded-For is different from the previous result. It is the IP address of the client.

Now let's propagate the network policy to the workload clusters. We set the x-forwarded-for header to only allow the IP `192.168.50.100` to access the service.

Create a file named `whoami-network-policy.yaml`

```yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: "whoami-network-policy"
spec:
  endpointSelector:
    matchLabels:
      app: whoami
  ingress:
  - toPorts:
    - ports:
      - port: "80"
        protocol: TCP
      - port: "443"
        protocol: TCP
      rules:
        http:
        - headers:
          - "X-Forwarded-For: 192.168.50.100,192.168.50.100"
```

Apply the network policy to the karmada cluster

```bash
kubectl apply --kubeconfig $KARMADA_CONFIG -f whoami-network-policy.yaml
```

Check the network policy in the workload clusters

```bash
kubectl get ciliumnetworkpolicy --context $CLUSTER1
kubectl get ciliumnetworkpolicy --context $CLUSTER2
```

<details>
<summary>The output is similar to:</summary>

```console
NAME                    AGE
whoami-network-policy   39s
NAME                    AGE
whoami-network-policy   40s
```
</details>

> Note: Karmada propagate the network policy to the workload clusters


Try to curl in the the server which ip is `192.168.50.100`

```bash
curl $HAPROXY_ENDPOINT
```

<details>
<summary>The output is similar to:</summary>

```console
...
...
Host: 192.168.50.100
User-Agent: curl/7.81.0
Accept: */*
X-Envoy-Expected-Rq-Timeout-Ms: 3600000
X-Envoy-External-Address: 10.46.0.135
X-Forwarded-For: 192.168.50.100,192.168.50.100
X-Forwarded-Proto: http
X-Request-Id: 81e74a1d-5d11-4276-a8b2-cf7dbf9882b2
```
</details>

> Note: This server is allowed to access the service

Use another server which ip is `192.168.50.102` to curl

```bash
curl $HAPROXY_ENDPOINT
```

<details>
<summary>The output is similar to:</summary>

```console
Access denied
```
</details>

> Note: This server is not allowed to access the service

## Step3: Allow specific HTTP header to access the service

We can also use the network policy to allow specific HTTP header to access the service. In this lab, we will use the user header to allow the user to access the service.

Update the file `whoami-network-policy.yaml` with new header rule
```yaml
'''
'''
      rules:
        http:
        - headers:
          - "X-Forwarded-For: 192.168.50.100.192,168,50,100"
          - "user: admin" # Add this line
```
> Note: Set the header `user: admin` to allow the user to access the service


Update the network policy in the karmada cluster

```bash
kubectl apply --kubeconfig $KARMADA_CONFIG -f whoami-network-policy.yaml
```



Try to curl the haproxy endpoint without user header in the server which ip is `192.168.50.100`
```bash
curl $HAPROXY_ENDPOINT
```

<details>
<summary>The output is similar to:</summary>

```console
Access denied
```
</details>

> Note: Server is not allowed to access the service

Try to curl the haproxy endpoint with user header

```bash
curl --header "user: admin" $HAPROXY_ENDPOINT
```

<details>
<summary>The output is similar to:</summary>

```console
...
...
Host: 192.168.50.100
User-Agent: curl/7.81.0
Accept: */*
User: admin
X-Envoy-Expected-Rq-Timeout-Ms: 3600000
X-Envoy-External-Address: 10.45.0.58
X-Forwarded-For: 192.168.50.100,192.168.50.100
X-Forwarded-Proto: http
X-Request-Id: ad1e169e-d371-4c8c-b915-ec7529d27143
```
</details>

> Note: Now server is allowed to access the service

## Conclusion

In this lab, we have setup karmada to propagate cilium network policy to the workload clusters. We also deploy a network policy to set the only allow IP address to access the service and allow specific HTTP header to access the service.

## References

- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Overview of Network Policy](https://docs.cilium.io/en/latest/security/policy/)
- [Network Policy Editor](https://editor.networkpolicy.io/)
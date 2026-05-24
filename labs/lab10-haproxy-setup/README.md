# Lab10 - HAProxy Setup

## Objectives

- Deploy HAProxy using Docker Compose
- Configure HAProxy to load balance between two web apps

## Prerequisites

- Environment from [Lab 9](../lab09-simple-app-propagating/README.md)
- [Docker](https://docs.docker.com/install/)

## Overview

In this lab, we will deploy HAProxy using Docker Compose. We will then configure HAProxy to load balance between two web apps.


## Step1: Deploy HAProxy

[HAProxy](https://www.haproxy.org/) is a free, very fast and reliable solution offering high availability, load balancing, and proxying for TCP and HTTP-based applications.

We can deploy HAProxy anywhere. Just make sure that it can access the web apps. In this lab, we will deploy HAProxy on the same machine as the control cluster.

[Docker compose](https://docs.docker.com/compose/) is a tool for defining and running Docker applications. With Compose, we use a YAML file to configure our application's services. Then, with a single command, we create and start all the services from our configuration.

To use Docker Compose to deploy HAProxy, we need to create a file named `docker-compose.yml` with the following content:
```yaml
version: "3"
services:
  haproxy:
      container_name: haproxy
      image: haproxy:latest
      volumes:
          - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg
      ports:
          - "80:80"
      expose:
          - 80
```


Set the Application endpoints in environment variables:
```bash
export CLUSTER1_APP_ENDPOINT=<YOUR_CLUSTER1_CILIUM_GATEWAY_IP>:<PORT>
export CLUSTER2_APP_ENDPOINT=<YOUR_CLUSTER2_CILIUM_GATEWAY_IP>:<PORT>
echo $CLUSTER1_APP_ENDPOINT
echo $CLUSTER2_APP_ENDPOINT
```
> Note: Replace `<YOUR_CLUSTER1_CILIUM_GATEWAY_IP>` and `<YOUR_CLUSTER2_CILIUM_GATEWAY_IP>` with the IP addresses of your Cilium gateway. Replace `<PORT>` with the port number of your web apps.

> Note: Follow the step5 in [Lab 7](../lab07-multi-cluster-setup/README.md) to get the IP addresses of your Cilium gateway.

<details>
<summary>The output is similar to:</summary>

```console
192.168.50.200:80
192.168.50.210:80
```
</details>



Now we can create the HAProxy configuration file. Create a file named `haproxy.cfg` with the following content
```bash
cat <<EOF > haproxy.cfg
defaults
    mode http
    timeout connect 5000ms
    timeout client 3000ms
    timeout server 3000ms

frontend http
    bind *:80
    mode http
    use_backend app_http

backend app_http
    balance roundrobin
    server cluster1 ${CLUSTER1_APP_ENDPOINT} check
    server cluster2 ${CLUSTER2_APP_ENDPOINT} check

EOF
```
> Note: This configuration file tells HAProxy to load balance between two clusters.


Deploy HAProxy by docker-compose:
```bash
docker compose up -d
```
> Note: The `-d` flag tells Docker Compose to run the containers in the background.


Verify that HAProxy is running
```bash
docker ps
```

<details>
<summary>The output is similar to:</summary>

```console
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                               NAMES
f847be9f3545   haproxy:latest   "docker-entrypoint.s…"   7 seconds ago   Up 6 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   haproxy
```
</details>


## Step2: Test HAProxy

Now we can test HAProxy. We will use `curl` to send HTTP requests to HAProxy.

Send HTTP requests to HAProxy:
```bash
for i in {1..10}; do curl -s 127.0.0.1:80 | grep Hostname ; done
```
> Note: This command will send 10 HTTP requests to HAProxy.

<details>
<summary>The output is similar to:</summary>

```console
Hostname: whoami-deployment-6d54cbf86f-mxxdc
Hostname: whoami-deployment-6d54cbf86f-vkmcc
Hostname: whoami-deployment-6d54cbf86f-mxxdc
Hostname: whoami-deployment-6d54cbf86f-vkmcc
Hostname: whoami-deployment-6d54cbf86f-lvt67
Hostname: whoami-deployment-6d54cbf86f-mxxdc
Hostname: whoami-deployment-6d54cbf86f-lvt67
Hostname: whoami-deployment-6d54cbf86f-mxxdc
Hostname: whoami-deployment-6d54cbf86f-lvt67
Hostname: whoami-deployment-6d54cbf86f-mxxdc
```
</details>

The output shows that HAProxy is load balancing between two clusters.

## Conclusion

In this lab, we deployed HAProxy using Docker Compose. We then configured HAProxy to load balance between two web apps.

## References

- [HAProxy](https://www.haproxy.org/)
- [Docker Compose overview](https://docs.docker.com/compose/overview/)
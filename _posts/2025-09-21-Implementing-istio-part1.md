---
layout: single
title: Implementing Istio - Part 1
excerpt: "How Istio secures pod-to-pod communication with tls. "
date: 2025-09-21
classes: wide
header:
  teaser: /assets/images/istio-servicemesh-part1/istio-intro.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - kubernetes
  - istio
tags:  
  - service mesh
  - microservices
  - traffic management
  - security
  - observability
---

![](/assets/images/istio-servicemesh-part1/istio-intro.png)



## Introduction
- Context
Nowadays, in a world dominated by microservices and platforms like Kubernetes, and with the rapid growth of these applications, it is challenging to secure, manage, and monitor them effectiverly across thousands services.

### Why this matters

By default communications betweens pods are in plain text and unencrypted. Any service can connect with each other without controls and this opens the door to security risks.

- Intercepting internal traffic
- Lateral movement in case of compromise
- Lack of auditability and visibility of communications

## What is Istio?

Is a service mesh that is installed in Kubernetes and helps to secure, manage, and monitor microservices.
It provides a uniform way to enforce security, control service-to-service communication, and collect observability data.
Istio introduces a control plane that allows you to manage traffic flow, enforce security policies, and gather telemetry without modifying your application code

### Key benefits
- mTLS: Encrypts the traffic pod to pod in-cluster, preventing sniffing attacks or man in the middles.
- Zero trust inside the mesh: each service authenticates itself with a certificate
- Granular authorization: control who can access what with AuthorizationPolicies
- Integration with identity providers (OIDC/JWT): allows you to integrate with external identity providers for user authentication and authorization.
- Auditability: provides detailed logs and metrics for all traffic within the mesh, making it easier to detect anomalies.


## Architecture in my homelab 
The following diagram illustrates the architecture of Istio i will be creating in my homelab.

![](/assets/images/istio-servicemesh-part1/istio-system.png)

## Step 1: Istiod instalation

First, add and update the Istio Helm repository:

```bash
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
```
Then install the base chart, which contains the cluster-wide CRDs, and deploy the control plane in the istio-system namespace.

```bash
helm install istio-base istio/base -n istio-system --set defaultRevision=default --create-namespace
```

### Step 2: Installing add-ons

**Kiali:** It’s an observability and operational console for Istio. It shows the topology of the mesh, the health of workloads, and lets you inspect and validate Istio configuration (VirtualServices, DestinationRules, Gateways, AuthorizationPolicies, etc.).
**Prometheus:** It provides monitoring and alerting capabilities for the service mesh. It also provides the metrics that Kiali uses to graph the health and performance of services.

```bash
#Istio dashaboard
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.27/samples/addons/kiali.yaml
#Metrics
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.27/samples/addons/prometheus.yaml
```

## Step 3 - Exposing kiali and grafana internally

I use MetalLB to create internal LoadBalancer IPs and expose the Istio Ingress Gateway on ports 80/443 (mapped to Envoy 8080/8443). This allows external access to services (e.g., Kiali, Prometheus) through Gateway/VirtualService routing.

```bash
#Istio dashaboard
kubectl create namespace istio-ingress
helm install istio-ingress istio/gateway -n istio-ingress --set service.type=LoadBalancer  --set service.ports[0].name=http  --set service.ports[0].port=80  --set service.ports[0].targetPort=8080  --set service.ports[1].name=https --set service.ports[1].port=443 --set service.ports[1].targetPort=8443
```


# In progress....



## Summary
- What we achieved
- Next steps

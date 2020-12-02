# Lab 4 - Linkerd Dashboard

## 4.1 Introduction to Linkerd Dashboard

The Dashboard provides a clickable user interface for administration of Linkerd. The Dashboard provides measurements of success rate, requests/second and latency for services on the mesh. Run the Linkerd Dashboard, by executing:

```sh
linkerd dashboard &
```

This command port-forwards from your local system to the `linkerd-web` service. You can also expose the dashboard using Kubernetes `ingress`, which we will see later in this section.

Since Linkerd's control plane components have the Linkerd proxy sidecarred, you can examine statistics of the traffic you are generating by looking at the dashboard. Execute:

```sh
linkerd -n linkerd top deploy/linkerd-web
```

_NOTE: `linkerd tap` provides the ability to listern to a traffic stream for a resource_

## 4.2 Exposing the dashboard

Instead of using `linkerd dashboard &` every time you'd like to see what's going on, you can expose the dashboard via an ingress. We will use the Nginx ingress, which we had deployed and used in Lab 3.

We will be applying Nginx ingress-traffic rule with basic authentication protocol

```yaml
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: web-ingress-auth
  namespace: linkerd
data:
  auth: YWRtaW46JGFwcjEkbjdDdTZnSGwkRTQ3b2dmN0NPOE5SWWpFakJPa1dNLgoK
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: web-ingress
  namespace: linkerd
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/upstream-vhost: $service_name.$namespace.svc.cluster.local:8084
    nginx.ingress.kubernetes.io/configuration-snippet: |
      proxy_set_header Origin "";
      proxy_hide_header l5d-remote-ip;
      proxy_hide_header l5d-server-id;
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: web-ingress-auth
    nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"
spec:
  rules:
    - host: dashboard.example.com
      http:
        paths:
          - backend:
              serviceName: linkerd-web
              servicePort: 8084
```

This exposes the dashboard at `dashboard.example.com` and protects it with basic auth with credentials admin,admin.

From here you may need to modify your resolv.conf to add `dashboard.example.com` to localhost or use an alternative approach in order to see the dashboard deployment. We will not cover this in the workshop.

## 4.2 Tools exposed by dashboard

Linkerd dashboard exposes various CLI tools which may come handy while you debug your application running on mesh.

Mainly there are three tools which Linkerd exposes as an extension to it's CLI commands

- stat : This will show the “golden” metrics for each deployment:
  - Success rates
  - Request rates
  - Latency distribution percentiles
- top (([see here](img/top.png)).) : to get a real-time view of which paths are being called.
- tap ([see here](img/tap.png)). : shows the stream of requests across a single pod, deployment, or even everything in the emojivoto namespace.

<h2>
  <a href="../lab-6/README.md">
  <img src="../img/go.svg" width="32" height="32" align="left" />
  Continue to Lab 6</a>: Linkerd Observability
  </h2>

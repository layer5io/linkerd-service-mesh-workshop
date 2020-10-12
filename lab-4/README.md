# Lab 4 - Linkerd Web

## 4.1 Introduction to Linkerd Web

The Linkerd dashboard gives a significant level perspective on what's going on with your administrations continuously. It very well may be utilized to see the "brilliant" measurements (success rate, requests/second and latency), visualize administration conditions and comprehend the strength of explicit assistance courses. One approach to pull it up is by running linkerd dashboard from the command line([see here](img/stat.png)).

With control plane start and running, we can access the linkerd web by

```sh
linkerd web &
```

This command uses a port-forward from the local system to linkerd-web pod, we can also expose the dashboard using ingress which we will see later in this section.

As the control-plane components have linkerd proxy injected in itself, we can see the traffic we are generating by looking at the dashboard itself by

```sh
linkerd -n linkerd top deploy/linkerd-web
```

_NOTE: `linkerd tap` gives us the ability to listern to a traffic stream for a resource_

## 4.2 Exposing the dashboard

Instead of using linkerd dashboard every time you'd like to see what's going on, you can expose the dashboard via an ingress. We will use the Nginx ingress, which we had deployed and used in Lab-3.

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

This exposes the dashboard at dashboard.example.com and protects it with basic auth with credentials admin,admin.

Now you can go to `dashboard.example.com` to see the dashboard deployment

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
  <a href="../lab-5/README.md">
  <img src="../img/go.svg" width="32" height="32" align="left" />
  Continue to Lab 5</a>: Debugging your application with dashboard
  </h2>

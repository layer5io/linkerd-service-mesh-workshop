# Lab 3 - Access Bookinfo site through Istio Ingress Gateway

The components deployed on the service mesh by default are not exposed outside the cluster.

For reasons of effortlessness, Linkerd doesn't give its own ingress controller. Rather, Linkerd is intended to work close by your ingress controller of decision.

## 3.1 How to use Ingress with Linkerd

If you're planning on injecting Linkerd into your ingress controller's pods there is some configuration required. Linkerd discovers services based on the :authority or Host header. This allows Linkerd to understand what service a request is destined for without being dependent on DNS or IPs.

When it comes to ingress, most controllers do not rewrite the incoming header (example.com) to the internal service name (example.default.svc.cluster.local) by default. In this case, when Linkerd receives the outgoing request it thinks the request is destined for example.com and not example.default.svc.cluster.local. This creates an infinite loop that can be pretty frustrating!

We will be using Nginx ingress gateway with Linkerd in this workshop.

## 3.2 Installing Nginx ingress controller/gateway

**Pre-requisites**
- Kubernetes (already set-up)
- Helm

**Installation**
- Clone the Ingress controller repo (repo is required to install the CRDs)
```sh
git clone https://github.com/nginxinc/kubernetes-ingress/
cd kubernetes-ingress/deployments/helm-chart
git checkout v1.8.1
```
- Apply the CRDs to your cluster (only for helm v2.x)
```sh
kubectl create -f crds/
``` 
- Install the controller using helm-chart
```sh
helm install my-release nginx-stable/nginx-ingress
```

## 3.3 Setting up ingress controller with the sample application deployed

Apply the following ingress definition to your cluster 
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: web-ingress
  namespace: emojivoto
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/configuration-snippet: |
      proxy_set_header l5d-dst-override $service_name.$namespace.svc.cluster.local:$service_port;
      grpc_set_header l5d-dst-override $service_name.$namespace.svc.cluster.local:$service_port;

spec:
  rules:
  - host: example.com
    http:
      paths:
      - backend:
          serviceName: web-svc
          servicePort: 80
```

The important definition in the above written definition is 
```sh
    nginx.ingress.kubernetes.io/configuration-snippet: |
      proxy_set_header l5d-dst-override $service_name.$namespace.svc.cluster.local:$service_port;
      grpc_set_header l5d-dst-override $service_name.$namespace.svc.cluster.local:$service_port;
```

This example combines the two directives that NGINX uses for proxying HTTP and gRPC traffic. In practice, it is only necessary to set either the proxy_set_header or grpc_set_header directive, depending on the protocol used by the service, however NGINX will ignore any directives that it doesn't need.
Nginx will add a l5d-dst-override header to instruct Linkerd what service the request is destined for. You'll want to include both the Kubernetes service FQDN (web-svc.emojivoto.svc.cluster.local) and the destination servicePort.

To test this, you need to get the external IP of your controller which you can get by running 
```sh
kubectl get svc --all-namespaces \
  -l app=nginx-ingress,component=controller \
  -o=custom-columns=EXTERNAL-IP:.status.loadBalancer.ingress[0].ip
```

You can now curl to your service without using port-forward
```sh
curl -H "Host: example.com" http://{external-ip}
```
## [Continue to lab 4 - Exploring Linkerd Web](../lab-4/README.md)

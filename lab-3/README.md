# Lab 3 - Ingressing and Egressing with Linkerd

The components deployed on the service mesh by default are not exposed outside the cluster.

For reasons of effortlessness, Linkerd doesn't give its own ingress controller. Rather, Linkerd is intended to work close by your ingress controller of decision.

## 3.1 How to use Ingress with Linkerd

In case you're anticipating infusing Linkerd into your ingress controller's pods there is some setup required. Linkerd discovers services dependent on the :authority or Host header. This permits Linkerd to comprehend what service a request is bound for without being subject to DNS or IPs.

We will be using Nginx ingress gateway with Linkerd in this workshop.

## 3.2 Installing Nginx ingress controller/gateway

**Pre-requisites**

- Kubernetes (already set-up)

**Installation**

- Install Ingress Controller using (For Docker-Desktop)

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.40.2/deploy/static/provider/cloud/deploy.yaml
```

- Install Ingress Controller using (For Minikube)
```sh
minikube addons enable ingress
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

This model joins the two mandates that NGINX utilizes for proxying HTTP and gRPC traffic. Practically speaking, it is just important to set either the proxy_set_header or grpc_set_header mandate, contingent upon the protocol utilized by the administration, anyway NGINX will overlook any orders that it needn't bother with. 

Nginx will include a l5d-dst-abrogate header to train Linkerd what administration the solicitation is bound for. You'll need to incorporate both the Kubernetes administration FQDN (web-svc.emojivoto.svc.cluster.local) and the objective servicePort.

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

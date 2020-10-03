# Lab 2 - Deploy sample application
To play with Linkerd and demonstrate some of it's capabilities, you will deploy the example EmojiVoto application, which is included the Linkerd package.

## What is the EmojiVoto Application?

A microservice application that allows users to vote for their favorite emoji, and tracks votes received on a leaderboard. May the best emoji win.

The end-to-end architecture of the application is shown [here](hhttps://github.com/BuoyantIO/emojivoto/blob/main/assets/emojivoto-topology.png).

It’s worth noting that these services have no dependencies on Linkerd, but make an interesting service mesh example, particularly because of the multitude of services, languages and versions for the reviews service.

Sidecars proxy can be either manually or automatically injected into your pods.

The injector is an admission controller, which receives a webhook request every time a pod is created. This injector inspects resources for a Linkerd-specific annotation (linkerd.io/inject: enabled). When that annotation exists, the injector mutates the pod's specification and adds both an initContainer as well as a sidecar containing the proxy itself.

As part of Linkerd deployment in [Lab 1](../lab-1/README.md), we have deployed the sidecar injector.

### <a name="auto"></a> Deploying Sample App

Linkerd, deployed as part of this workshop, will also deploy the sidecar injector. Let us now verify sidecar injector deployment.


```sh
kubectl -n linkerd get deployment -l linkerd=sidecar-injector
```
Output:

```sh
NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
linkerd-sidecar-injector   1         1         1            1           1d
```

NamespaceSelector decides whether to run the webhook on an object based on whether the namespace for that object matches the [selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors).


```sh
kubectl get namespace -L linkerd
```
<!-- 
Output:
```sh
NAME           STATUS    AGE       ISTIO-INJECTION
default        Active    1h        
istio-system   Active    1h        disabled
kube-public    Active    1h        
kube-system    Active    1h
``` -->

Now in Meshery in the browser, navigate to the Linkerd adapter's management page from the left nav menu.

On the Istio adapter's management page, please enter `default` in the `Namespace` field.
Then, click the (+) icon on the `Sample Application` card and select `Emojivoto Application` from the list.

This will do 3 things: 
1. Label `default` namespace for sidecar injection
1. Deploys all the Book info services in the `default` namespace
1. Deploys the virtual service and gateway needed to expose the Book info's productpage in the `default` namespace.

<small>Manual step for can be found [here](#appendix)</small>


### Verify the namespace is labelled

```sh
kubectl get namespace -L linkerd-injection
```

Output:
<!-- ```sh
NAME           STATUS    AGE       ISTIO-INJECTION
default        Active    1h        enabled
istio-system   Active    1h        disabled
kube-public    Active    1h        
kube-system    Active    1h
``` -->

### <a name="verify"></a> Verify Emojivoto deployment

1. Verify that previous deployments are all in a state of AVAILABLE before continuing. **Do not proceed until they are up and running.**

    ```sh
    watch kubectl get deployment
    ```

2. Inspect the details of the pods

    Let us look at the details of the pods:
    ```sh
    watch kubectl get po
    ```

    Let us look at the details of the services:
    ```sh
    watch kubectl get svc
    ```

    Now let us pick a service, for instance productpage service, and view it's sidecar configuration:
    ```sh
    kubectl get po

    kubectl describe pod productpage-v1-.....
    ```

## [Continue to Lab 3 - Access Linkerd via Linkerd Web](../lab-3/README.md)

<hr />
Alternative, manual installation steps below. No need to execute, if you have performed the steps above.
<hr />

## <a name="appendix"></a> Appendix - Alternative Manual Steps

### Label namespace for injection
Install emojivoto into the emojivoto namespace by running:
```sh
curl -sL https://run.linkerd.io/emojivoto.yml \
  | kubectl apply -f -
  ```

Before we mesh it, let's take a look at the app. If you're using Docker Desktop at this point you can visit http://localhost directly. If you're not using Docker Desktop, we'll need to forward the web-svc service. To forward web-svc locally to port 8080, you can run:

```sh
kubectl -n emojivoto port-forward svc/web-svc 8080:80
```

### Deploy Emojivoto
Applying this yaml file included in the Istio package you collected in https://github.com/layer5io/istio-service-mesh-workshop/tree/master/lab-1#1 will deploy the Book info app in you cluster.


```sh
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

<!-- Already deployed by linkerd using tap components -->

<!-- ### Deploy Gateway and Virtual Service for Book info app -->
<!-- 
```sh
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
``` -->


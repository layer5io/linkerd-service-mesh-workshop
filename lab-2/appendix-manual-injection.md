### <a name="manual"></a> Lab 3 Appendix: Deploying Sample App with manual sidecar injection

To do a manual sidecar injection we will be using `linkerd` command:

```sh
kubectl get -n emojivoto deploy -o yaml \
  | linkerd inject - > newEmojivoto.yaml
```

Observing the new yaml file reveals that additional container Linkerd Proxy has been added to the Pods with necessary configurations:

```
    template:
      metadata:
        annotations:
          linkerd.io/inject: enabled
```

We need to now deploy the new yaml using `kubectl`

```sh
kubectl apply -f newEmojivoto.yaml
```

To do both in a single command:

```sh
kubectl get -n emojivoto deploy -o yaml \
  | linkerd inject - \
  | kubectl apply -f -
```

Continue to [Lab 3](../lab-3/README.md).

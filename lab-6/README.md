# Lab 6 - Linkerd Telemetry & Distributed Tracing

## 6.1 - Linkerd Telemetry

Linkerd's telemetry and monitoring features function automatically, without requiring any work on the part of the developer.

We have already looked at Linkerd Telemetry, not by name but by concept. Using stat, top and tap is one of the major and characterstic feature which Linkerd offers out of the box. All of this metrics are also accessible through Grafana Dashboard which Linkerd provides.

Accessing Grafana Dashboard for each service in Linkerd :
- Start `linkerd dashboard` & navigate over the service you would want to see Grafana Dashboard for.
- In the last column, click of the `Grafana Icon` to access the metrics dashboard for the following deployment.

## 6.2 Distributed tracing with Linkerd

The first step of getting distributed tracing setup is installing a collector onto your cluster. This component consists of “receivers” that consume spans emitted from the mesh and your applications as well as “exporters” that convert spans and forward them to a backend.
Linkerd now has add-ons which enables users to install extra components that integrate well with Linkerd. Tracing is such one add-on which includes OpenCensus Collector and Jaeger.

To enable tracing onto your cluster :
```sh
cat >> config.yaml << EOF
tracing:
  enabled: true
EOF
```

This configuration file can also be used to apply Add-On configuration (not just specific to tracing Add-On).

Now, the above configuration can be applied using the --addon-config file with CLI :
```sh
linkerd upgrade --addon-config config.yaml | kubectl apply -f -
```

You will now have a linker-collector and linkerd-jaeger deployments in the linkerd namespace that are running as part of the mesh. Collector has been configured to:
- Receive spans from OpenCensus clients
- Export spans to a Jaeger backend

Before moving onto the next step, make sure everything is up and running with kubectl:
```sh
kubectl -n linkerd rollout status deploy/linkerd-collector
kubectl -n linkerd rollout status deploy/linkerd-jaeger
```

## 6.2.1 Configure your sample application

Apply the tracing configuration to the `emojivoto application`:
```sh
kubectl -n emojivoto patch -f https://run.linkerd.io/emojivoto.yml -p '
spec:
  template:
    metadata:
      annotations:
        config.linkerd.io/trace-collector: linkerd-collector.linkerd:55678
        config.alpha.linkerd.io/trace-collector-service-account: linkerd-collector
'
```

Before moving onto the next step, make sure everything is up and running with kubectl:
```sh
kubectl -n emojivoto rollout status deploy/web
```

Unlike most features of a service mesh, distributed tracing requires modifying the source of your application. Tracing needs some way to tie incoming requests to your application together with outgoing requests to dependent services. To do this, some headers are added to each request that contain a unique ID for the trace. Linkerd uses the b3 propagation format to tie these things together. Emojivoto application already incorporates all of such changes.

To enable tracing in emojivoto, run:
```sh
kubectl -n emojivoto set env --all deploy OC_AGENT_HOST=linkerd-collector.linkerd:55678
```

## 6.3 Explore Jaeger

With `vote-bot` starting traces for every request, spans should now be showing up in Jaeger. To get to the UI, start a port forward and send your browser to http://localhost:16686.
```sh
kubectl -n linkerd port-forward svc/linkerd-jaeger 16686
```
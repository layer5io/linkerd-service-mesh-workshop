# Lab 1 - Deploy Linkerd

Now that we have a Kubernetes cluster and Meshery, we are ready to download and deploy Linkerd resources.

## Steps

- [1. Install Linkerd](#1)
- [2. Verify install](#2)
- [3. Confirm add-ons](#3)

## <a name="1"></a> 1 - Install Linkerd

In Meshery, select the deployed Linkerd adapter in the left nav menu under the `Management` section ([see screenshot](img/linkerd.png)).

On the Linkerd adapter's management page, on the `Install` card, you can click on the (+) icon and select `Latest Linkerd` to install the latest version of Linkerd ([see screenshot](img/linkerd.png)).

<small>For manual steps go [here](#appendix)</small>

## <a name="2"></a> 2 - Verify install

Linkerd is deployed in a separate Kubernetes namespace `Linkerd`. To check if Linkerd is deployed, and also, to see all the pieces that are deployed, execute the following:

```sh
linkerd check
```

## <a name="3"></a> 3 - Enforce mTLS strict mode

By default, Linkerd automatically enables mutual Transport Layer Security (mTLS) for most HTTP-based communication between meshed pods, by establishing and authenticating secure, private TLS connections between Linkerd proxies.

## <a name="4"></a> 4 - Confirming Add-ons

Linkerd, as part of this workshop, is installed with several optional addons like:

1. [Prometheus](https://prometheus.io/)
2. [Grafana](https://grafana.com/)
3. [Jaeger](https://www.jaegertracing.io/)
4. [Dashboard](https://linkerd.io/2/reference/architecture/#dashboard)

You will use Prometheus and Grafana for collecting and viewing metrics, while for viewing distributed traces, you can choose between [Jaeger](https://www.jaegertracing.io/). In this training, we will use Jaeger.

## [Continue to Lab 2 - Deploy Sample Bookinfo app](../lab-2/README.md)

<hr />
Alternative, manual installation steps below. No need to execute, if you have performed the steps above.
<hr />

## <a name="appendix"></a> Appendix - Alternative Manual Install

### <a name="1.1"></a> 1.1 - Download Linkerd

You will download and deploy the latest Linkerd resources on your Kubernetes cluster.

**_Note to Docker Desktop users:_** please ensure your Docker VM has atleast 4GiB of Memory, which is required for all services to run.

### <a name="1.2"></a> 1.2 - Setting up `linkerd` CLI

On a \*nix system, you can setup `linkerd` by doing the following:

The above command will get the latest Linkerd package and untar it in the same folder.

Change into the Linkerd package directory and add the `linkerd` client to your PATH environment variable.

```sh
curl -sL https://run.linkerd.io/install | sh
export PATH=$PATH:$HOME/.linkerd2/bin
```

Alternatively, on MacOS you can sue `HomeBrew` to install `linkerd`

```sh
brew install linkerd
```

To verify `linkerd` is setup lets try to print out the command help

```sh
linkerd version
```

We can use a new feature in linkerd to check if the cluster is ready for install:

```sh
linkerd check --pre
```

### Install Linkerd:

Deploy Linkerd custom resources:

```sh
linkerd install | kubectl apply -f -
```

Use the following command to see the progress on the installation of Linkerd custom CRDs and components

```sh
linkerd check
```

# Lab 5 - Debugging application using Linkerd

We will be using Linkerd to debug the sample application, we had deployed earlier in the workshop.
The demo application emojivoto has some issues. Let's use that and Linkerd to diagnose an application that fails in ways which are a little more subtle than the entire service crashing. 

Let's jump into debugging the Emojivoto application right away.

## 5.1 Exploring the Linkerd Web

If you glance at the Linkerd dashboard (by running the linkerd dashboard command), you should see all the resources in the emojivoto namespace, including the deployments. Each deployment running Linkerd shows success rate, requests per second and latency percentiles.
You might notice is that the success rate is well below 100%! Let's debug this further & click on the web component.

<img align="center" style="margin-bottom:20px;" src="img/octopus.png"  width="70%" />

The first thing you'll see here is that the web deployment is taking traffic from vote-bot (a deployment included with emojivoto to continually generate a low level of live traffic). The web deployment also has two outgoing dependencies, emoji and voting.

While the emoji deployment is handling every request from web successfully, it looks like the voting deployment is failing some requests! A failure in a dependent deployment may be exactly what is causing the errors that web is returning.

## 5.2 Debugging the application

Scrolling down a little from the deployment page, we'll see a live list of all traffic that is incoming to and outgoing from web.

<img align="center" style="margin-bottom:20px;" src="img/web-top.png"  width="70%" />

There are two calls that are not at 100%: the first is vote-bot's call to the `/api/vote` endpoint. The second is the `VoteDoughnut` call from the web deployment to its dependent deployment, voting.
Since `/api/vote` is an incoming call, and `VoteDoughnut` is an outgoing call, this is a good clue that this endpoint is what's causing the problem!


To dig a little deeper, we can click on the tap icon in the far right column. This will take us to the live list of requests that match only this endpoint. You'll see `Unknown under the GRPC status column`. 

<img align="center" style="margin-bottom:20px;" src="img/web-tap.png"  width="70%" />

This is because the requests are failing with a [gRPC status code 2](https://godoc.org/google.golang.org/grpc/codes#Code), which is a common error response. Linkerd is aware of gRPC's response classification without any other configuration.

Now at this point we have all the necessary debugging information which can help us to restore the application to stable/working state.
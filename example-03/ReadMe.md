# Example 03

## What we'll do

This example sets up a simple web app, similar to the welcome page in the first two exercises, but running via a local Kubernetes cluster instead of just a Docker container.

The local Kubernetes cluster will be powered by [Minikube](https://minikube.sigs.k8s.io/docs/) and we'll use a custom [Wordle image](https://hub.docker.com/r/joevgreathead/wordle) created by the author.

# Setup

Install minikube

On macOS:

```bash
brew install minikube
```

Start minikube's cluster, verify everything is running, and start the dashboard in a separate terminal window.

```bash
$ minikube start
...
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

$ kubectl get po -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS       AGE
kube-system   coredns-xxxxxxxxx-wrhpx            1/1     Running   0              2m32s
kube-system   etcd-minikube                      1/1     Running   0              2m45s
kube-system   kube-apiserver-minikube            1/1     Running   0              2m47s
kube-system   kube-controller-manager-minikube   1/1     Running   0              2m44s
kube-system   kube-proxy-jzlhz                   1/1     Running   0              2m32s
kube-system   kube-scheduler-minikube            1/1     Running   0              2m45s
kube-system   storage-provisioner                1/1     Running   1 (2m2s ago)   2m43s

$ minikube dashboard
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:52765/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```

# One container, one port

We're gonna play some Wordle, but first let's act as if we're first rolling it out for ourselves. Create a single instance of Wordle we can use from our local machine.

## Create a deployment with a single container

```
kubectl create deployment wordle-port --image=joevgreathead/wordle:latest
```

## Expose the port on the deployment

```
kubectl expose deployment wordle-port --type=NodePort --port=9673
```

## Forward the port from your machine

```
kubectl port-forward service/wordle-port 8000:9673
```

Open the app in your [browser](http://localhost:8000)

## Delete the deployment

```
kubectl delete deployment wordle-port
```

# One Container, Scaled to meet demand

Let's now pretend other folks have discovered Wordle and we need to meet the high demand of the public. Scale up the service and apply a Load Balancer so traffic is sent to all 3 pods.

## Create a new deployment with a load balancer

Run this in a separate terminal window to give our LoadBalancer an external IP to which we can connect. Keep this terminal open and the process running.

```
minikube tunnel
```

Create the deployment and the load balancer.

```
kubectl create deployment lets-play-wordle --image=joevgreathead/wordle:latest
```

```
kubectl expose deployment lets-play-wordle --type=LoadBalancer --port=9673
```

This provides us with a single pod (one instance of the application) on our node (our local machine). That's not enough!

```
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
lets-play-wordle-xyzxyzxyzxyzxyz    1/1     Running   0          4m45s
```

## Scale up the pods

```
kubectl scale deployment lets-play-wordle --replicas=3
```

# Sell out

Now that we've sold out to a big mega corp, we can scale back down and remove the deployment.

```
kubectl scale deployment lets-play-wordle --replicas=1
```

```
kubectl delete deployment lets-play-wordle
```

# Stop the cluster

Nice work. You've now run a set of pods, which house containers, behind a service, on your node (aka local machine).

```
minikube stop
```

Be sure to kill the processes for the dashboard and tunnel in your other terminal tabs as well.

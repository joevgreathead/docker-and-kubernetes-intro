# Example 05

## What we'll do

This example mimics the lifecycle of a real app being updated and pushed to a production instance of Kubernetes. It exhibits the value Docker and Kubernetes brings us in managing a live app by allowing us to update an app with zero downtime and utterly easy rollback, if necessary.

# Setup

Start minikube, then start the minikube tunnel and dashboard in separate terminal windows. Also, make sure your terminal is in the example-05 folder.

```bash
$ minikube start
# In other windows:
$ minikube dashboard
$ minikube tunnel
```

```bash
~/code/docker-and-kubernetes-intro $ cd example-05
~/code/docker-and-kubernetes-intro/example-05 $
```

# Build the Rails app Docker image

Run the first script to build the Docker image.

```bash
sh ./01-build-rails.sh
```

Make sure the image creation completed without error.

# Push the Rails app image to Docker hub

In order for Kubernetes to use our image, we have to push the image to Dockerhub. Run the 02 script to do this.

```bash
sh 02-push-rails.sh
```

# Apply the deployment and service

The Kubernetes deployment and service yml files have been preconfigured. Run them with the following:

```bash
kubectl apply -f deployment.yml
```

and

```bash
kubectl apply -f service.yml
```

Kubernetes will pull the image, start a single pod with the Rails app, and front it with a load balancer. The app should now be accessible at:

http://localhost:8080/

# Update the app and image

Ack! We have this running in Kubernetes, but our own app is telling us it needs to be updated!

Navigate to:
`./rails-app/app/views/basic/index.html.erb`

Comment out the top line and uncomment out the second line to replace it. We need to create the image again and push it, but with an updated version since we changed the code.

Go into the two scripts and change the `0.1` value to `0.2` at the end of each command in the scripts.

Run both scripts to create the new image and push it to dockerhub.

# Re-apply the deployment and service to see the update

Once pushed, we can deploy the new version of the app with zero downtime. Update the `deployment.yml` file to use the newer `0.2` value instead of `0.1`.

Re-apply the deployment and Kubernetes will handle the rest.

```bash
kubectl apply -f deployment.yml
```

This deployment will happen fast as Kubernetes creates the new pod, gets it ready, transitions traffic to it, and then removes the old pod. You can refresh the minikube dashboard to see the pods update, run `kubectl get po` repeatedly to see the same, or try and detect any downtime by refreshing the app page at http://localhost:8080/.

Once complete, you should see the new message we uncommented earlier.

# Stop everything

Stop all processes in terminal tabs and make sure to run the following to stop minkube.

```bash
minikube stop
```
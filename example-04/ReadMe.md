# Example 04

## What we'll do

This example replicatest the results of example-03, but using kubernetes configuration yaml files. This is a more typical way of leveraging Kubernetes and has a similar set of values to using Docker compose because your configurations are documented, can be version controlled, and the commands used are simply.

# Setup

Ensure you have minikube running as well as the tunnel and dashboard processes in separate terminal windows.

```
$ minikube start
$ minikube tunnel
$ minikube dashboard
```

# Create the deployment

## Open deployment.yml

Note the apiVersion and kind of item we're creating is included at the top. This is one of the ways Kubernetes knows how to read the rest of the file.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  #
spec:
  replicas:
  selector:
    #
  template:
    metadata:
      # 
    spec:
      # 
```

## Add a name for the deployment

```yaml
metadata:
  name: wordle-deployment
```

## Add a number of replicas for the wordle pods

```yaml
spec:
  replicas: 3
```

## Add a selector to match labels

```yaml
spec:
  ...
  selector:
    matchLabels:
      app: wordle
```

## Add a template and metadata for the containers

```yaml
spec:
  ...
  template:
    metadata:
      name: wordle
      labels:
        app: wordle
```

## Add a spec for the containers

```yaml
spec:
  ...
  template:
    ...
    spec:
      containers:
      - image: joevgreathead/wordle:latest
        name: wordle
        ports:
        - containerPort: 9673
          protocol: TCP
```

## Add a readiness probe, just for fun

This will probably fail when it actually deploys, but it won't stop us from running the app.

```yaml
spec:
  ...
  template:
    ...
    spec:
      containers:
        ...
        readinessProbe:
          periodSeconds: 1
          httpGet:
            path: /
            port: 9673
```

## Create the deployment

```
kubectl apply -f deployment.yml
```

# Create the service

## Open service.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  #
spec:
  selector:
    # 
  ports:
    # 
```

## Add a name

```yaml
metadata:
  name: wordle-service
```

## Add a spec

Add a type of `LoadBalancer`, a selector for the `app` key and value `wordle`, and expose port `8080` to target port `9673`.

```yaml
spec:
  type: LoadBalancer
  selector:
    app: wordle
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 9673
```

## Create the service (i.e. the load balancer)

```
kubectl apply -f service.yml
```

## Play a round of Wordle

http://localhost:8080/

# Close everything down

```
$ kubectl delete deployment wordle-deployment
$ kubectl delete service wordle-service
$ minikube stop
```
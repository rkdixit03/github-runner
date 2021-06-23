# Github self-hosted runner Dockerfile and Kubernetes configuration

[![awesome-runners](https://img.shields.io/badge/listed%20on-awesome--runners-blue.svg)](https://github.com/jonico/awesome-runners)

This repository contains a Dockerfile that builds a Docker image suitable for running a [self-hosted GitHub runner].

You can build this image yourself, or use the Docker image from the [Docker Hub](https://hub.docker.com/repository/docker/sanderknape/github-runner/general).

## Building the container

`docker build -t github-runner .`

## Examples

Register a runner to a repository.

```sh
docker run --name github-runner \
     -e GITHUB_OWNER=username-or-organization \
     -e GITHUB_REPOSITORY=my-repository \
     -e GITHUB_PAT=[PAT] \
     rkdixit03/github-runner
```

**Register a runner with github token.**

```sh
docker run --name github-runner \
     -e GITHUB_OWNER=username-or-organization \
     -e GITHUB_REPOSITORY=my-repository \
     -e GITHUB_TOKEN=[github.token] \
     rkdixit03/github-runner
```

**Create an organization-wide runner.**

```sh
docker run --name github-runner \
    -e GITHUB_OWNER=username-or-organization \
    -e GITHUB_PAT=[PAT] \
    rkdixit03/github-runner
```

**Set labels on the runner.**

```sh
docker run --name github-runner \
    -e GITHUB_OWNER=username-or-organization \
    -e GITHUB_REPOSITORY=my-repository \
    -e GITHUB_PAT=[PAT] \
    -e RUNNER_LABELS=comma,separated,labels \
    rkdixit03/github-runner
```

**Install additional tools on the runner.**

```sh
docker run --name github-runner \
    -e GITHUB_OWNER=username-or-organization \
    -e GITHUB_REPOSITORY=my-repository \
    -e GITHUB_PAT=[PAT] \
    -e ADDITIONAL_PACKAGES=firefox-esr,chromium \
    rkdixit03/github-runner
```


**secret.yaml**

```sh
apiVersion: v1
kind: Secret
metadata:
  name: github-secret
stringData:
  GITHUB_PAT: ghp_XXXXXXXXXXXX
```

```sh
kubectl apply -f ./secret.yaml
```

**Kubernetes.yaml**

```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: github-runner
  labels:
    app: github-runner
spec:
  replicas: 2
  selector:
    matchLabels:
      app: github-runner
  template:
    metadata:
      labels:
        app: github-runner
    spec:
      containers:
      - name: github-runner
        image: rkdixit03/github-runner:latest
        env:
        - name: GITHUB_OWNER
          value: rkdixit03
        - name: GITHUB_PAT # stored separately in a Kubernetes secret
          valueFrom:
            secretKeyRef:
              name: github-secret
              key: GITHUB_PAT
```
**Deploy changes in K8s**
```sh
kubectl apply -f Kubernetes.yaml
```
**Check deployment status**
```sh
kubectl get deployment
```
**Check number of Pods deployed**
```sh
kubectl get pods
```

To test out the things in local, Install Minikube + hyperkit(or Virtual VM) and apply changes there.

**Start Minikube**
```sh
minikube start --vm-driver=hyperkit --alsologtostderr
```

Now, apply above mentioned changes.


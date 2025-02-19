---
title: 'Navigating the Kubernetes Sea: iO Digital Java Team Adventure in Container Orchestration'
date: '2025-01-31'
tags: ['kubernetes', 'devops', 'cloud']
images:
  ['/articles/io-digitals-java-team-adventure-in-container-orchestration/kubenetes-workshop.png']
summary: This article is about A/B testing and Canary releases in Kubernetes with plain kubernetes and without any other tooling like for example istio.
authors: ['arno-koehler']
theme: 'beige'
serie: 'google-days'
---

![kubenetes-workshop.png](/articles/io-digitals-java-team-adventure-in-container-orchestration/kubenetes-workshop.png)

## What is Kubernetes?

Kubernetes, also known as K8s, is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications.
It was originally developed by Google and is now maintained by the Cloud Native Computing Foundation (CNCF).
Kubernetes provides a powerful set of tools for managing containerized applications in a cloud-native environment.

### Knowing Kubernetes components

When using Kubernetes you have to know some basic concepts. To start with there is the top level organization unit called the Namespace.
Next there are the workload resources.

- **Deployments:** Declarative management of application state and rolling updates.
- **StatefulSets:** Manages the deployment and scaling of a set of pods, and provides guarantees about the ordering and uniqueness of these pods.
- **DaemonSets:** Ensures that all (or some) nodes run a copy of a pod.
- **ReplicaSets:** Ensures the desired number of pod replicas are running (though often managed indirectly through Deployments).
- **Pods:** The smallest deployable units, usually containing one container or a group of tightly coupled containers.

Next there are the service resources.

- **Services:** An abstraction of a (set of) pods and a policy to access them.
- **Ingress:** An API object that manages external access to services within a cluster, typically via HTTP or HTTPS.

And last but not least there are the configuration resources.

- **ConfigMaps:** Decouples configuration artifacts from image content to keep containerized applications portable.
- **Secrets:** Securely stores sensitive information, such as passwords, OAuth tokens, and SSH keys.
- **Volumes:** Provides a way to store data in a way that persists beyond the lifetime of a pod.

# The Power of Basic Kubernetes: Blue-Green Releases Demystified

So what did we look into? One of our developers recently found himself working on a greenfield project with Kubernetes (K8s), attempting to clean up the project's Helm charts.
What happened was that he dove deep into the K8s documentation and discovered a wealth of functionality already baked into the platform.
This exploration led him to appreciate the power of "basic" Kubernetes, especially when it comes to implementing deployment strategies like Blue-Green releases.

## Blue-Green Deployments

Before we dive into Blue-Green releases, let's review some core K8s resources:
What's the benefit of using a service instead of connecting directly to a pod?
Services provide a stable endpoint for accessing pods. Why is that important? Well pod's can be ephemeral; meaning they can be destroyed and / or recreated at any given time.
Services also enable load balancing and service discovery within the cluster.

An Ingress is an API object that manages external access to services within a cluster, typically via HTTP or HTTPS.
It acts as a smart router for your cluster.

### Workshop time!

Now if you go to https://github.com/iodigital-com/kubernetes-greenblue-workshop you can find the code of the workshop.

First thing you need to do is create an environment to work with.
Our team explored various ways to run Kubernetes locally, each with its own advantages:

1. **Docker Desktop with Kubernetes**: Most of our team opted for this method, enabling the Kubernetes feature in Docker Desktop. This approach doesn't require a VM, resulting in less overhead and a smoother experience for many developers.
2. **Minikube**: A couple of team members chose Minikube, finding it relatively easy to set up. Minikube creates a VM to run a single-node Kubernetes cluster, which also works well with kubectl (the Kubernetes command-line tool).

To enable Kubernetes in Docker Desktop, just go to settings and enable Kubernetes in the Kubernetes tab, you will have to restart Docker Desktop after that.

When you did this make sure to have kubectl installed and configured.
For the Mac users that is as easy as:

```shell
brew install kubectl
```

As you see in the readme, after building the application with gradle, we start with creating the first pod.
You can do so by first searching for `{buildversion}` and change it into a version that you like.
Be sure to keep it in sync with the docker setting in the `build.gradle.kts` file.

```kotlin
ktor {
    docker {
        localImageName.set("my-app")
        imageTag.set("1.0.0")
    }
}

```

When you did that, validate your docker image is build by running `docker images | grep my-app` and you should see the image you just build.

```shell
kubectl apply -f blue-deployment.yaml,service-node-port-blue.yaml
```

To check if that worked you can run (assuming you have httpie installed):

```shell
http get localhost:30081
```

or for the curl users:

```shell
curl localhost:30081
```

Now create the second pod and service:

```shell
kubectl apply -f green-deployment.yaml,service-node-port-green.yaml
```

And again check if that worked:

```shell
http get localhost:30082
```

If you look at the service-node-port scripts you will see the service definition helping you to access the pods from outside the cluster.

Now we are going to add the nginx ingress controller. The YAML to deploy NGNIX Ingress Controller can be found at their github.
We used the 1.12.0 release from the tag: [1.12.0](https://raw.githubusercontent.com/kubernetes/ingress-nginx/refs/heads/release-1.12/deploy/static/provider/kind/deploy.yaml).

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/refs/heads/release-1.12/deploy/static/provider/kind/deploy.yaml
```

# Creating an ingress

We need the services to run on a clusterIp now because we're going to access them from outside

```shell
kubectl apply -f service-cluster-ip.yaml
```

```shell
kubectl apply -f ingress-single.yaml
```

```shell
http get app.localhost
```

## The Power of Simplicity

What makes this approach powerful is its simplicity. With just a few Kubernetes resources and some YAML configurations, we've implemented a sophisticated deployment strategy. This "basic" setup provides:

- Zero-downtime deployments
- Easy rollbacks
- Gradual rollout with traffic splitting
- Reduced risk in production deployments

## **Conclusion**

Kubernetes (k8s) is a powerful tool that offers a wide range of features for managing containerized applications. While it can be complex, Kubernetes provides a solid foundation for deploying, scaling, and managing applications in a cloud-native environment.
Tools like Istio can make certain tasks easier, but it is more important to understand the basic functionality that Kubernetes provides.
Sometimes, you don't need additional tools to achieve the same result. Kubernetes can be quite powerful on its own, and it's very easy to try out these deployment strategies.

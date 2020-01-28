# k8s-p4proxy
Kuberenetes Perforce Helix Proxy

This repository contains sample manifests to run a Perforce Helix Proxy in Kubernetes.

## Manifests
There are three different components in the repo.

* `p4proxy-secrets.yml` - This will create a secret to be used by the Perforce Helix Proxy to cache the specified depots.
* `p4proxy-stateful.yml` - This will create a proxy pod with 3 containers with persistent volumes
* `p4proxy-service.yml` - This will create a service using a LoadBalancer to load balance traffic between the proxy servers (if number of replicas is more than 1). **Note:** The annotations in this example are specific to AWS. These values will either be different or not needed depending on your specific load balancer implementation.

## Architecture
The StatefulSet will create a pod with 3 containers.

* `p4proxy` - This is the main container in the pod that runs the Perforce Helix Proxy service
* `p4proxy-cleanup` - This container is responsible for cleaning up the cache volume when it hits the 90% threshold. It will bring the volume to 80% utilization. By default, it runs every 5 minutes.
* `p4proxy-sync` - This container is responsible for caching the contents of the Perforce Helix Core server and runs the sync process every 5 minutes by default.

## Configuration
There are many settings you can configure for each of the containers in the pods. They may be set via an environment variable or by a static entry in the container execution command.

#### Global
You can configure the username and password for the account you'd like to use to authenticate and sync with the Perforce Helix Core server.

In the p4proxy-secrets.yml, enter the desired username and password. You'll need to base64 encode the values.

Example:
```shell
echo 'svc-p4proxy' | base64
c3ZjLXA0cHJveHkK

echo 'supersecret' | base64
c3VwZXJzZWNyZXQK
```

Based on the values above, you're p4proxy-secrets.yml would look like this:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: p4proxy
type: Opaque
data:
  username: c3ZjLXA0cHJveHkK
  password: c3VwZXJzZWNyZXQK
```

The following are global defaults using environment variables that can be overridden for each container.

* `P4PORT` - The port you'd like to listen/connect to for the Perforce Proxy
* `P4CHARSET` - The charset to use when connecting

#### p4proxy
This environmental variable is only applicable to the p4proxy container.

* `P4TARGET` - The hostname of the Perforce Helix Core server that you to proxy

#### p4proxy-sync
This environmental variable is only applicable to the p4proxy-sync container.

* `P4CLIENT` - The client/workspace to use to cache the depot of the Perforce Helix Core server

## Install

To install this on Kubernetes by applying each manifest. The example below installs the p4proxy pod in a separate namespace.

```shell
kubectl create -n p4proxy
kubectl apply -f p4proxy-secrets.yml -n p4proxy
kubectl apply -f p4proxy-stateful.yml -n p4proxy
kubectl apply -f p4proxy-service.yml -n p4proxy
```

To view the various resources created, run:

```shell
kubectl get all -n p4proxy
```

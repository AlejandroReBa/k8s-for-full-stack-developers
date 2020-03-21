# How To Install Software on Kubernetes Clusters with the Helm Package Manager

Notes and files for the tutorial [How To Install Software on Kubernetes Clusters with the Helm Package Manager](https://www.digitalocean.com/community/tutorials/how-to-install-software-on-kubernetes-clusters-with-the-helm-package-manager).

## Step 1 - Installing Helm easier under Linux/macOS

This one-liner will install Helm in a glimpse:

```sh
bash <(curl -sSL https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get)
```

## Step 2 - Commands to install Helm

```sh
# Create Service Account tiller
kubectl -n kube-system create serviceaccount tiller
# Bind tiller to cluser-admin role
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
# Initialize Helm on your host and create tiller pod
helm init --service-account tiller
# Check out tiller is working
kubectl get pods --namespace kube-system
```

## Step 3 - Installing a Helm Chart

```sh
# Install dashboard Chart
helm install stable/kubernetes-dashboard --name dashboard-demo
# Check status
helm list
# Check status of the service
kubectl get services
```

## Step 4 - Updating a Chart

```sh
# Change name to 'dashboard'
helm upgrade dashboard-demo stable/kubernetes-dashboard --set fullnameOverride="dashboard"
kubectl get services
```

> Note: I did not managed to get access to the service either using `kubectl proxy` URL nor port-forwarding command

## Step 5 - Rolling back a Release

```sh
# Do the rollback
helm rollback dashboard-demo 1
kubectl get services
```

## Step 6 - Deleting a Release

```sh
# Deletes the release (but some stuff will be left)
helm delete dashboard-demo
helm list --deleted
# Delete the release once for all
helm delete dashboard-demo --purge
helm list --deleted
```
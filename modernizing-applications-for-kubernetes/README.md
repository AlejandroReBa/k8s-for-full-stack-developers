# Modernizing Applications for Kubernetes

For [this guide][1], I will create two deployment for [MajorcaDevs/ecosia-treeminer][2] using this guide.

- [simple](./simple/): simple deployment with only the treeminer and a service to be able to acces it.
- [with-tor](./with-tor/): treeminer with tor in a pod, and its service.

You can deploy each using:

```sh
# Simple deployment
kubectl apply -f simple
# With tor deployment
kubectl apply -f with-tor

kubectl get deployments
kubectl get services
```

  [1]: https://www.digitalocean.com/community/tutorials/modernizing-applications-for-kubernetes
  [2]: https://github.com/MajorcaDevs/ecosia-treeminer

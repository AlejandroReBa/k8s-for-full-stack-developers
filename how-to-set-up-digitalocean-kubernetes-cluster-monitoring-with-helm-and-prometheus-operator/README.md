# How to Set Up DigitalOcean Kubernetes Cluster Monitoring with Helm and Prometheus Operator

Notes and files for [the tutorial][1] about setting up monitoring on our cluster using Prometheus Operator.

## Step 1 — Creating a Custom Values File

In this step, we need to override some values from the Helm chart in order to customize it for our deployment. The version included in this repo of the [custom-values.yml](./custom-values.yml) is ready to be used in a local cluster by changing the storage types. If you are using DO, then change it back to the original type.

## Step 2 - Installing the chart

Before continue, if you are deploying to a custom cluster, then you need to deploy a `storageclass` for our chart. Go to [configuring local-storage in cluster](#configuring-local-storage-in-cluster) section.

Let's start installing the chart, it will take some time to do so, there are a lot of stuff to deploy:

```sh
helm install --namespace monitoring --name ks-cluster-monitoring -f custom-values.yml stable/prometheus-operator
```

Then to check if the Prometheus Operator chart is working, run the following:

```sh
kubectl --namespace monitoring get pods -l "release=ks-cluster-monitoring"
```

## Step 3 - Accessing Grafana and Exploring Metrics Data

By default, all services uses `ClusterIP` which is not accessible outside the cluster itself, so there are several ways to be able to access it. But we are going to use the port forward one:

```sh
kubectl get svc -n monitoring
kubectl port-forward -n monitoring svc/ks-cluster-monitoring-grafana 8000:80
```

Now go to <http://localhost:8000> and enjoy the data visualization.

## Cleanup

For some reasong, the delete does not remove everything, so here it is some cleanup commands:

```sh
helm delete --purge ks-cluster-monitoring
kubectl delete -n monitoring customresourcedefinition alertmanagers.monitoring.coreos.com podmonitors.monitoring.coreos.com prometheuses.monitoring.coreos.com prometheusrules.monitoring.coreos.com servicemonitors.monitoring.coreos.com thanosrulers.monitoring.coreos.com
kubectl delete -n monitoring pvc alertmanager-ks-cluster-monitoring-prom-alertmanager-db-alertmanager-ks-cluster-monitoring-prom-alertmanager-0 prometheus-ks-cluster-monitoring-prom-prometheus-db-prometheus-ks-cluster-monitoring-prom-prometheus-0
```

## Configuring local-storage in Cluster

In this section, we are going to deploy a [local persistent volume provisioner][2] using [sig-storage-local-static-provisioner][3]. The deployment will use filesystem mode for the provisioner with one hard drive for node and several bind-mounts to simulate several drives.

First create a new hard drive for each worker node of at least 20GB of space. Boot up the workers and let's work on creating the volumes for the provisioner.

First, create a partition table for the disk `/dev/sdb` (ensure this is the new hard drive, before doing anything):

```sh
sudo fdisk /dev/sdb
...
Command (m for help): g # type g and press enter
Created a new GPT disklabel (GUID: ...)

Command (m for help): n # type n and press enter
Partition number (1-128, default 1): # press enter
First sector (2048-..., default 2048): # press enter
Last sector, +sectors or +size{K,M,G,T,P} (...): #press enter

Created a new partition 1 of type 'Linux Filesystem' and size x GiB.

Command (m for help): w # type w and press enter
```

Now it is time to format the partition with ext4 and mount it:

```sh
sudo mkfs.ext4 -m 0 /dev/sdb1
DISK_UUID=`sudo blkid -s UUID -o value /dev/sdb1`
sudo mkdir /mnt/$DISK_UUID
echo "UUID=$DISK_UUID /mnt/$DISK_UUID ext4 defaults 0 2" | sudo tee -a /etc/fstab
sudo mount /mnt/$DISK_UUID
```

To be able to use several volumes in the provisioner, we are going to create some folders in the new disk and bind-mount them to another folder, for the provisioner:

```sh
for i in $(seq 1 5); do
  sudo mkdir -p /mnt/${DISK_UUID}/vol${i} /mnt/disks/${DISK_UUID}_vol${i}
  echo "/mnt/${DISK_UUID}/vol${i} /mnt/disks/${DISK_UUID}_vol${i} none bind 0 0" | sudo tee -a /etc/fstab
  sudo mount /mnt/disks/${DISK_UUID}_vol${i}
done
```

On your host, clone the repository `git clone --depth=1 https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner.git`, then generate the template using `helm template -f .../how-to-set-up-digitalocean-kubernetes-cluster-monitoring-with-helm-and-prometheus-operator/provisioner-custom-values.yml --name local-storage-provisioner --namespace kube-system ./helm/provisioner > local-volume-provisioner.generate.yaml`. There is an example provided of the [generated file](./my-local-volume-provisioner.generate.yaml) if you would like to use it directly instead of generating one. Then deploy the provisioner with `kubectl apply -f local-volume-provisioner.generate.yaml`.

  [1]: https://www.digitalocean.com/community/tutorials/how-to-set-up-digitalocean-kubernetes-cluster-monitoring-with-helm-and-prometheus-operator
  [2]: https://kubernetes.io/blog/2019/04/04/kubernetes-1.14-local-persistent-volumes-ga/
  [3]: https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner/
# kubernetes-playground

Create a standalone Kubernetes cluster with one master node and two worker nodes using Vagrant and Ansible.

## Before you start

Download and install Vagrant. I am using version 2.2.3.

Download and install VirtualBox (with extension pack). I am using version 6.0.4.

I tested my scripts on Mac, but the process should be the same on Linux. Not sure about Windows.

## Install plugins (optional)

Install the vagrant-disksize plugin if you want to configure the disk size (default is 20Gb):

    vagrant plugin install vagrant-disksize

## Create nodes

Create and start the nodes:

    vagrant up

Verify that the nodes are running:

    vagrant status k8s1 k8s2 k8s3

## Connect to master node

Open a shell on the master node:

    vagrant ssh k8s1

## Install CNI plugin (required)

Execute script on master node:

    start-calico

## Verify nodes are ready

Execute command on master node:

    kubectl get nodes

    NAME   STATUS   ROLES    AGE     VERSION
    k8s1   Ready    master   3h20m   v1.13.2
    k8s2   Ready    <none>   3h19m   v1.13.2
    k8s3   Ready    <none>   3h17m   v1.13.2

## Install Tiller (optional, required for using Helm)

Execute script on master node:

    start-tiller

## Create the default Storage Class

Execute script on master node:

    create-standard-storageclass

A Storage Class is required in order to create Persistent Volumes which live on cluster nodes.

A Persistent Volume configuration for creating a volume on node k8s1 looks like:

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: local-pv-k8s1-1
    spec:
      capacity:
        storage: 5Gi
      accessModes:
      - ReadWriteOnce
      persistentVolumeReclaimPolicy: Retain
      storageClassName: standard
      local:
        path: /var/tmp/disk1
      nodeAffinity:
        required:
          nodeSelectorTerms:
          - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
              - k8s1

Create the volume with the command:

    kubectl create -f pv-k8s1-1.yaml

The persistent volume will be assigned to any pod requesting a volume with a Persistent Volume Claim and running on node k8s1.

## Verify pods are running

Execute command on master node:

    kubectl get pods --all-namespaces

    NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
    kube-system   calico-node-65f2k                      2/2     Running   0          106m
    kube-system   calico-node-c4z8r                      2/2     Running   0          106m
    kube-system   calico-node-r54wn                      2/2     Running   0          106m
    kube-system   coredns-86c58d9df4-dlcfs               1/1     Running   0          3h20m
    kube-system   coredns-86c58d9df4-t5hbr               1/1     Running   0          3h20m
    kube-system   etcd-k8s1                              1/1     Running   0          3h20m
    kube-system   kube-apiserver-k8s1                    1/1     Running   0          3h20m
    kube-system   kube-controller-manager-k8s1           1/1     Running   0          3h20m
    kube-system   kube-proxy-5lbkn                       1/1     Running   0          3h17m
    kube-system   kube-proxy-67lnb                       1/1     Running   0          3h19m
    kube-system   kube-proxy-z8c7m                       1/1     Running   0          3h20m
    kube-system   kube-scheduler-k8s1                    1/1     Running   0          3h20m
    kube-system   kubernetes-dashboard-57df4db6b-82qxc   1/1     Running   0          3h20m
    kube-system   metrics-server-68d85f76bb-t7gks        1/1     Running   0          3h20m
    kube-system   tiller-deploy-8485766469-kr9t5         1/1     Running   0          75m

## Get token for accessing Dashboard

Execute script on master node and get token from output:

    dashboard-token

## Expose Dashboard on host

Execute script on host:

    kubectl --kubeconfig=admin.conf proxy

Open browser at address and login using dashboard token:

    http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/namespace/kube-system?namespace=kube-system

## Enable pods scheduling on Master node (optional)

Execute script on master node to run pods on master node:

    taint-nodes

## Stop nodes

Execute command on host:

    vagrant halt

## Remove nodes

Execute command on host:

    vagrant destroy -f

## Credits

Vagrant and Ansible scripts have been inspired by: https://github.com/davidkbainbridge/k8s-playground

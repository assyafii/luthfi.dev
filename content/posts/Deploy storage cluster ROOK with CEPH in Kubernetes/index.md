---
title: '2. Deploy storage cluster ROOK with CEPH in Kubernetes'
date: 2021-01-02
weight: 2
layout: 'list'
---
---
> Specification : Kubernetes, ROOK, CEPH

### **Lab Topology**
You can check installation kubernetes cluster in previous documentation, https://assyafii.com/docs/install-kubernetes-cluster-multi-master-ha/
![k8s-topology](./k8s-topology.png)

&nbsp;
### **Storages nodes disks**

We use 3 disks extended (vdb, vdc, vdd) in each of storage-nodes, total 6 disks for rook cluster.

#### Detail disks
![rook-cluster](./cluster-rook.png)


![rook-disks](./disks.png)

### **Master node**
---
#### Clone ROOK Project 
```
cd ~
git clone --single-branch --branch release-1.7 https://github.com/rook/rook.git
```

#### Deploy the Rook Operator
```
cd rook/cluster/examples/kubernetes/ceph
kubectl create -f crds.yaml
kubectl create -f common.yaml
kubectl create -f operator.yaml
```

#### Make sure all Rook components already UP
![rook-operator](./operator.png)

#### Verify the rook-ceph-operator is in the Running
![rook-pod](./pod.png)

#### Create a Ceph Storage Cluster
Set default namespace to rook-ceph, you can set to default namespace agaian after installation.

```
kubectl config set-context --current --namespace rook-ceph
```

##### Important : Expicitly define the nodes and raw disks devices to be used.
For any further customizations, check in [ROOK Ceph Cluster CRD documentation.](https://rook.io/docs/rook/latest/ceph-cluster-crd.html)

```
sudo nano cluster.yaml
```
![rook-cluster](./cluster.png)

#### Deploy Rook ceph cluster
`kubectl create -f cluster.yaml`

Need some minutes to deploy it, make sure all completed 
```
kubectl get -n rook-ceph jobs.batch
kubectl -n rook-ceph get cephcluster
```

![rook-jobs](./jobs.png)


#### Deploy Rook Ceph toolbox
The Rook Ceph toolbox is a container with common tools used for rook debugging and testing.

```
cd ~
cd rook/cluster/examples/kubernetes/ceph
kubectl  apply  -f toolbox.yaml
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
```

My cluster status CEPH Warning, because degraded data, but no problem.
![rook-ceph](./ceph.png)

All OSD UP
![rook-osd](./osd.png)

#### Create pool shared filesystem in CEPH (cephfs)

```
cd ~
cd rook/cluster/examples/kubernetes/ceph/
sudo nano filesystem.yaml
```

Write your filesystem metadata name.

![rook-cephfs](./cephfs.png)

Verify if metadata and data pools are created.
![rook-verify](./verify.png)

#### Create storage class for cephfs
```
sudo nano csi/cephfs/storageclass.yaml
kubectl create -f csi/cephfs/storageclass.yaml
kubectl get sc
```

change fsName & pool name 
![rook-sc-create](./create-sc.png)

Verify storage class created 
![rook-sc](./sc.png)

#### Last step, Testing create PVC & POD
```
kubectl create  -f  csi/cephfs/pvc.yaml
kubectl get pvc
kubectl create -f csi/cephfs/pod.yaml
kubectl get pod
```

Example manifest file PVC

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cephfs-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: rook-cephfs
```

Example manifest file POD

```
apiVersion: v1
kind: Pod
metadata:
  name: csicephfs-demo-pod
spec:
  containers:
    - name: web-server
      image: nginx
      volumeMounts:
        - name: mypvc
          mountPath: /var/lib/www/html
  volumes:
    - name: mypvc
      persistentVolumeClaim:
        claimName: cephfs-pvc
        readOnly: false
```

#### Verify POD & PVC Already created

Persistent Volume
![rook-pvc](./pvc.png)

POD with PVC
![rook-pod-testing](./pod-testing.png)

&nbsp;
&nbsp;
#### Reference :

https://computingforgeeks.com/how-to-deploy-rook-ceph-storage-on-kubernetes-cluster/











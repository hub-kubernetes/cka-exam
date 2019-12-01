# Storage Class on GKE 


In the Kubernetes 1.6 release, dynamic provisioning has been promoted to stable (having entered beta in 1.4). This is a big step forward in completing the Kubernetes storage automation vision, allowing cluster administrators to control how resources are provisioned and giving users the ability to focus more on their application. With all of these benefits, there are a few important user-facing changes (discussed below) that are important to understand before using Kubernetes 1.6.

# Storage Classes and How to Use them

StorageClasses are the foundation of dynamic provisioning, allowing cluster administrators to define abstractions for the underlying storage platform. Users simply refer to a StorageClass by name in the PersistentVolumeClaim (PVC) using the “storageClassName” parameter.

In the following example, a PVC refers to a specific storage class named “gold”.

```
apiVersion: v1

kind: PersistentVolumeClaim

metadata:

  name: mypvc

  namespace: testns

spec:

  accessModes:

  - ReadWriteOnce

  resources:

    requests:

      storage: 100Gi

  storageClassName: gold

```


## Dynamic provisioning of GCP PD 

```
apiVersion: v1

kind: PersistentVolumeClaim

metadata:

  name: mypvc

  namespace: testns

spec:

  accessModes:

  - ReadWriteOnce

  resources:

    requests:

      storage: 100Gi

```

## ADD GCP PD Storage class in kubernetes

```
kind: StorageClass

apiVersion: storage.k8s.io/v1

metadata:

  name: gold

provisioner: kubernetes.io/gce-pd

parameters:

  type: pd-ssd

```

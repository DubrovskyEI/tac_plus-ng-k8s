### Using PersistentVolume for storing logs

**Local Persistent Volume - Kubernetes storage on local disks**
#### Create StorageClass for local storage

A StorageClass provides a way for administrators to describe the _classes_ of storage they offer.

Create StorageClass from the manifest

```Bash
kubectl create -f persistent-volumes-manifests/sc-local.yaml
```

#### Create Local PersistentVolume

The PersistentVolume subsystem provides an API for users and administrators that abstracts details of how storage is provided from how it is consumed.

It is a resource in the cluster just like a node is a cluster resource.
PVs are volume plugins like Volumes, but have a lifecycle independent of any individual Pod that uses the PV.

Create PersistentVolume from the manifest

```Bash
kubectl create -f persistent-volumes-manifests/tac-plus-ng-pv-local.yaml
```

>[!Note]
> To get Cluster's Node names use command:
> `kubectl get nodes`

#### Create PersistentVolumeClaim for local storage

Pods use PersistentVolumeClaims to request physical storage.
Creating a PersistentVolumeClaim that requests a volume of at least 300 mibibytes that can provide read-write access for at most one Node at a time.

Create PersistentVolumeClaim from the manifest

```Bash
kubectl create -f persistent-volumes-manifests/tac-plus-ng-pvc-local.yaml
```

##### Check out the state of created resources

```Bash
 kubectl get sc
```

```Bash
kubectl get pv
```

```Bash
kubectl get pvc
```

#### Create a Pod that uses PersistentVolumeClaim as a volume

Create Pod from the manifest

```Bash
kubectl create -f persistent-volumes-manifests/tac-plus-ng-pod-local-pvc.yaml
```

> [!NOTE]
> If run on a minikube, persistent volume stores on Cluster's Node `minikube`.
> 
> To check manually you need to connect to Node via ssh by execute:
> `minikube ssh`

#### Create a Deployment that uses PersistentVolumeClaim as a volume

Cleanup Pod created below

```Bash
kubectl delete pods tac-plus-ng-with-local-pvc
```

Create Deployment from the manifest

```Bash
kubectl create -f persistent-volumes-manifests/tac-plus-ng-deployment-local-pvc.yaml
```

##### References

> References
> 
> 1. https://serveradmin.ru/hranilishha-dannyh-persistent-volumes-v-kubernetes/
> 2. https://kubernetes.io/docs/concepts/storage/storage-classes/#local
> 3. https://kubernetes.io/docs/concepts/storage/persistent-volumes/
> 4. https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume
> 5. https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim



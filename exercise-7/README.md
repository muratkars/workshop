# Exercise 7 - Taking a snapshot of a Persistent Volume

## What is a Snapshot?

A storage snapshot is a set of reference markers for data at a particular point in time. A snapshot acts like a detailed table of contents, providing you with accessible copies of data that you can roll back to.

The possible operations of this feature is creating, restoring, and deleting a snapshot.

OpenEBS operator will deploy a `snapshot-controller` and a `snapshot-provisioner` container inside a single pod called `snapshot-controller`.

`Snapshot-controller` will create a CRD for `VolumeSnapshot` and `VolumeSnapshotData` custom resources when it starts and will also watch for `VolumeSnapshotresources` and take snapshots of the volumes based on the referred snapshot plugin. 

Snapshot-provisioner will be used to restore a snapshot as a new persistent volume via dynamic provisioning.

With OpenEBS 0.6 release [openebs-operator.yaml](https://raw.githubusercontent.com/openebs/openebs/master/k8s/openebs-operator.yaml) deploys both snapshot-controller and snapshot-provisioner. 

1.  Once snapshot-controller is running, run the command below to see the created CustomResourceDefinitions (CRD):

    ```
    $ kubectl get crd
    kubectl get crd
    NAME                                                         AGE
    storagepoolclaims.openebs.io                                 4h
    storagepools.openebs.io                                      4h
    volumepolicies.openebs.io                                    4h
    volumesnapshotdatas.volumesnapshot.external-storage.k8s.io   4h
    volumesnapshots.volumesnapshot.external-storage.k8s.io       4h
    ```

## Creating a stateful workload

Before creating a snapshot, you must have a PersistentVolumeClaim for which you need to take a snapshot.

Let's deploy a basic application that writes to persistent disk called [blue.yaml](blue.yaml). Once you deploy the blue application, it's PVC will be created by the name specified in the application deployment yaml, for example `demo-vol1-claim`.  

1.  Create the application YAML file:

    ```
    touch blue.yaml
    nano blue.yaml
    ```
    
2.  Paste the content and save.

    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: blue
    spec:
      restartPolicy: Never
      containers:
      - name: blue
        image: busybox
        command:
        - "/bin/sh"
        - "-c"
        - "while true; do date >> /tmp/pod-out.txt; sleep 1; done"
        volumeMounts:
        - name: blue-volume
          mountPath: /tmp
      volumes:
      - name: blue-volume
        persistentVolumeClaim:
          claimName: demo-vol1-claim
    ---
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: demo-vol1-claim
    spec:
      storageClassName: openebs-standard
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1G
    ```

3.  Create the deployment.

    ```
    $ kubectl create -f blue.yaml
    pod "blue" created
    persistentvolumeclaim "demo-vol1-claim" created
    ```

## Taking a snapshot of a Persistent Volume

Once the `blue` application is running, you can take a snapshot. After creating the `VolumeSnapshot` resource, `snapshot-controller` will attempt to create the actual snapshot by interacting with the snapshot APIs. If successful, the `VolumeSnapshot` resource is bound to a corresponding `VolumeSnapshotData` resource. 

To create a snapshot you must reference the `PersistentVolumeClaim` name in the snapshot specification that references the data you want to snapshot. 

1.  Validate the PV status and name of the claim.

    ```
    $ kubectl get pv
    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                     STORAGECLASS       REASON    AGE
    pvc-d2e08c54-733e-11e8-9f52-06731769042c   1G         RWO            Delete           Bound     default/demo-vol1-claim   openebs-standard             2m
    ```
    
2.  Create the snapshot YAML file. Example used here [snapshot.yaml](snapshot.yaml)

    ```
    touch blue.yaml
    nano blue.yaml
    ```

2.  Paste the content and save.

    ```
    apiVersion: volumesnapshot.external-storage.k8s.io/v1
    kind: VolumeSnapshot
    metadata:
      name: snapshot-blue
      namespace: default
    spec:
      persistentVolumeClaimName: demo-vol1-claim
```

3.  Create snapshot:
    
    ```
    $ kubectl create -f snapshot.yaml
    volumesnapshot.volumesnapshot.external-storage.k8s.io "snapshot-blue" created
    ```

4.  List snapshots:

   ```$ kubectl get volumesnapshot 
   NAME            AGE 
   snapshot-demo   18s
   ```

5.  The output above shows that your snapshot was created successfully. You can also check the snapshot-controller’s logs to verify this by using the following commands. 

```
$ kubectl get volumesnapshot -o yaml
 apiVersion: v1
 items:
   - apiVersion: volumesnapshot.external-storage.k8s.io/v1
  kind: VolumeSnapshot
  metadata:
    clusterName: ""
    creationTimestamp: 2018-01-24T06:58:38Z
    generation: 0
    labels:
      SnapshotMetadata-PVName: pvc-f1c1fdf2-00d2-11e8-acdc-54e1ad0c1ccc
      SnapshotMetadata-Timestamp: "1516777187974315350"
    name: snapshot-demo
    namespace: default
    resourceVersion: "1345"
    selfLink: /apis/volumesnapshot.external-storage.k8s.io/v1/namespaces/default/volumesnapshots/fastfurious
    uid: 014ec851-00d4-11e8-acdc-54e1ad0c1ccc
  spec:
    persistentVolumeClaimName: demo-vol1-claim
    snapshotDataName: k8s-volume-snapshot-2a788036-00d4-11e8-9aa2-54e1ad0c1ccc
  status:
    conditions:
    - lastTransitionTime: 2018-01-24T06:59:48Z
      message: Snapshot created successfully
      reason: ""
      status: "True"
      type: Ready
    creationTimestamp: null
```



```
$ kubectl get volumesnapshotdata -o yaml
apiVersion: volumesnapshot.external-storage.k8s.io/v1
  kind: VolumeSnapshotData
  metadata:
    clusterName: ""
    creationTimestamp: 2018-01-24T06:59:48Z
    name: k8s-volume-snapshot-2a788036-00d4-11e8-9aa2-54e1ad0c1ccc
    namespace: ""
    resourceVersion: "1344"
    selfLink: /apis/volumesnapshot.external-storage.k8s.io/v1/k8s-volume-snapshot-2a788036-00d4-11e8-9aa2-54e1ad0c1ccc
    uid: 2a789f5a-00d4-11e8-acdc-54e1ad0c1ccc
  spec:
    openebsVolume:
      snapshotId: pvc-f1c1fdf2-00d2-11e8-acdc 54e1ad0c1ccc_1516777187978793304
    persistentVolumeRef:
      kind: PersistentVolume
      name: pvc-f1c1fdf2-00d2-11e8-acdc-54e1ad0c1ccc
    volumeSnapshotRef:
      kind: VolumeSnapshot
      name: default/snapshot-demo
  status:
    conditions:
    - lastTransitionTime: null
      message: Snapshot created successfully
      reason: ""
      status: "True"
      type: Ready
    creationTimestamp: null
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

Once you have taken a snapshot, you can create a clone from the snapshot and restore your data.



## Deleting a Snapshot

You can delete a snapshot that you have created which will also delete the corresponding Volume Snapshot Data resource from Kubernetes. The following command will delete the *snapshot.yaml* file.

```
kubectl apply -f snapshot.yaml
```
   
### [Continue to Exercise 8 - Chaos Engineering with Litmus](../exercise-5/README.md)

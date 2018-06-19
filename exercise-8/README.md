# Exercise 8 - Cloning and Restoring a Snapshot

## Cloning and Restoring a Snapshot 

After creating a snapshot, you can restore it to a new Persitent Volume Claim. To do this you must create a special `StorageClass` implemented by snapshot-provisioner. 
We will then create a `PersistentVolumeClaim` referencing this `StorageClass` for dynamically provisioning new `PersistentVolume`. 
An annotation on the `PersistentVolumeClaim` will inform `snapshot-provisioner` about where to find the information it needs to deal with the OpenEBS API Server to restore the snapshot. 

A `StorageClass` can be defined as in the following example. Here, the `provisioner` field defines which provisioner should be used and what parameters should be passed to that provisioner when dynamic provisioning is invoked.

Such a `StorageClass` is necessary for restoring a Persistent Volume from an already created Volume Snapshot and Volume Snapshot Data.

## Create a Storage Class to restore from Snapshot

1.  Create the storageclass YAML file:

    ```
    touch openebs-restore-storageclass.yaml
    nano openebs-restore-storageclass.yaml
    ```
    
2.  Paste the content and save.

    ```
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: snapshot-promoter
    provisioner: volumesnapshot.external-storage.k8s.io/snapshot-promoter
    ```
    
    `annotations:` snapshot.alpha.kubernetes.io/snapshot: the name of the Volume Snapshot that will be restored.
    `storageClassName:` Storage Class created by the admin for restoring Volume Snapshots.

3.   Once the openebs-restore-storageclass is created, you have to deploy the YAML by using the following command.

     ```
     kubectl apply -f restore-storageclass.yaml
     ```

## Restore from Snapshot

You can now create a `PersistentVolumeClaim` that will use the `StorageClass` to dynamically provision a `PersistentVolume` that contains the information of your snapshot. Create a YAML file that will delpoy a `PersistentVolumeClaim` using the following details.  

1.  Create the persistentvolumeclaim YAML file:

    ```
    touch restore-pvc.yaml
    nano restore-pvc.yaml
    ``    `
    
2.  Paste the content and save.

    ```
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: demo-snap-vol-claim
      annotations:
        snapshot.alpha.kubernetes.io/snapshot: snapshot-blue
    spec:
      storageClassName: snapshot-promoter
      accessModes: ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
    ```

3.  Once the `restore-pvc.yaml` is created,  you have to deploy the YAML by using the following command.

    ```
    kubectl apply -f restore-pvc.yaml
    ```

4.  Finally mount the `demo-snap-vol-claim` PersistentVolumeClaim into a green pod to see that the snapshot was restored. While deploying the green pod, you have to edit the deplyment YAML and mention the restore PersistentVolumeClaim name, volume name, and volume mount accordingly. An example for your reference is given below. 

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
        - name: demo-snap-vol1
          mountPath: /tmp
      volumes:
        - name: demo-snap-vol1
          persistentVolumeClaim:
            claimName: demo-snap-vol-claim
    ---

3.  Once `green.yaml` is created, you can deploy the YAML by using the following command:

    ```
    kubectl apply -f green.yaml
    ```

Once the green pod is in running state, you can check the integrity of files which were created earlier before taking the snapshot. 
   
### [Continue to Exercise 9 - Rollout your application with Blue/Green deployment](../exercise-9/README.md)

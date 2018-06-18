# Exercise 8 - Cloning and Restoring a Snapshot

## Cloning and Restoring a Snapshot 

After creating a snapshot, you can restore it to a new Persitent Volume Claim. To do this you must create a special `StorageClass` implemented by snapshot-provisioner. 
We will then create a `PersistentVolumeClaim` referencing this `StorageClass` for dynamically provisioning new `PersistentVolume`. 
An annotation on the `PersistentVolumeClaim` will inform `snapshot-provisioner` about where to find the information it needs to deal with the OpenEBS API Server to restore the snapshot. 

A `StorageClass` can be defined as in the following example. Here, the `provisioner` field defines which provisioner should be used and what parameters should be passed to that provisioner when dynamic provisioning is invoked.

Such a `StorageClass` is necessary for restoring a Persistent Volume from an already created Volume Snapshot and Volume Snapshot Data.

## Create a Storage Class to restore from Snapshot

1.  Create the application YAML file:

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

Once the *restore-storageclass.yaml* is created, you have to deploy the yaml by using the following command.

```
$ kubectl apply -f restore-storageclass.yaml
```

You can now create a PersistentVolumeClaim that will use the StorageClass to dynamically provision a PersistentVolume that contains the information of your snapshot. Create a yaml file that will delpoy  a PersistentVolumeClaim  using the following details.  Use `$ vi restore-pvc.yaml` command to create the yaml and add the following details to the yaml.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-snap-vol-claim
  annotations:
    snapshot.alpha.kubernetes.io/snapshot: snapshot-demo
spec:
  storageClassName: snapshot-promoter
  accessModes: ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

Once the *restore-pvc.yaml* is created ,  you have to deploy the yaml by using the following command.

```
$ kubectl apply -f restore-pvc.yaml
```

Finally mount the **demo-snap-vol-claim** PersistentVolumeClaim into a percona-snapsot pod to see that the snapshot was restored. While deploying the percona-snapshot pod, you have to edit the deplyment yaml and mention the restore PersistentVolumeClaim name, volume name, and volume mount accordingly. An example for your reference is given below. You can create a yaml by using the command `$ vi percona-openebs-deployment-2.yaml` and add the following details.

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: percona2
  labels:
    name: percona
spec:
  replicas: 1
  selector:
    matchLabels:
      name: percona
  template:
    metadata:
      labels:
        name: percona
    spec:
      tolerations:
      - key: "ak"
        value: "av"
        operator: "Equal"
        effect: "NoSchedule"
      containers:
        - resources:
            limits:
              cpu: 0.5
          name: percona
          image: percona
          args:
            - "--ignore-db-dir"
            - "lost+found"
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: k8sDem0
          ports:
            - containerPort: 3306
              name: percona
          volumeMounts:
            - mountPath: /var/lib/mysql
              name: demo-snap-vol1
      volumes:
        - name: demo-snap-vol1
          persistentVolumeClaim:
            claimName: demo-snap-vol-claim
---
apiVersion: v1
kind: Service
metadata:
  name: percona-mysql
  labels:
    name: percona-mysql
spec:
  ports:
    - port: 3306
      targetPort: 3306
  selector:
      name: percona
```

Once *restore-storageclass.yaml* is created, you have to deploy the yaml by using the following command.

```
$ kubectl apply -f percona-openebs-deployment-2.yaml
```

Once the percona-snapshot pod is in running state, you can check the integrity of files which were created earlier before taking the snapshot. 
   
### [Continue to Exercise 9 - Rollout your application with Blue/Green deployment](../exercise-9/README.md)

# Exercise 9 - Rollout your application with Blue/Green deployment

## What is Blue/Green Deployment?

Kubernetes has a controller object called [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/). 
A Deployment controller provides declarative updates for Pods and ReplicaSets.
Deployments are used to provide rolling updates, change the desired state of the pods, rollback to an earlier revision, etc.

However, there are many traditional workloads that won't work with Kubernetes way of rolling updates. 
If your workload needs to deploy a new version and cut over to it immediately then you may need to perform blue/green deployment instead.

Using Blue/Green deployment approach, you label the current production as “Blue”. Create an identical production environment called “Green” - Redirect the services to “Green”. 
If services are functional in “Green” - destroy “Blue”. If “Green” is bad - rollback to “Blue”.

## Create the Blue Application 

For this final exercise you will create a Percona (blue) workload, a service, job running a sql load generator. 
Then you will create another Percona (green) workload with the snapshot of the first Percona workload's persistent volume and switch the service at the same time.

First, create a Percona (blue) pod. Blue pod runs a Percona workload and writes to a persistent volume called `demo-vol1` created by the persistent volume claim `demo-vol1-claim`.  

1.  Make sure that you have cleared your environment. Delete all previosuly created pods, snapshots and PV/PVCs.

2.  




4.  Create the blue-percona.yaml pod.

    ```
    $ kubectl create -f blue.yaml
    pod "blue" created
    persistentvolumeclaim "demo-vol1-claim" created
    ```
## Create the service
    
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
    
## Create the Green Application with the Snapshot of Blue's Data

Once the `blue` application is running, you can take a snapshot, restore and use it for another pod. 

4.  Create snapshot, restore and deploy green application:
    
    ```
    $ kubectl create -f snapshot.yaml,restore-pvc.yaml,green.yaml
    volumesnapshot.volumesnapshot.external-storage.k8s.io "snapshot-blue" created
    ```
   
4.  List pods and verify green pod is running. 

    ```
    $ kubectl get pods
    NAME                                                             READY     STATUS    RESTARTS   AGE
    alertmanager-79fc4b59db-82zmr                                    1/1       Running   0          21h
    blue                                                             1/1       Running   0          20h
    grafana-7688b94dc-lr578                                          1/1       Running   0          21h
    green                                                            1/1       Running   0          2m
    node-exporter-6gtgx                                              1/1       Running   0          21h
    node-exporter-btdp8                                              1/1       Running   0          21h
    node-exporter-k5v6t                                              1/1       Running   0          21h
    prometheus-deployment-78cd57c5d7-q4sbl                           1/1       Running   0          21h
    pvc-b6cd585c-73e8-11e8-9f52-06731769042c-ctrl-546ddc8969-f9s97   2/2       Running   0          6m
    pvc-b6cd585c-73e8-11e8-9f52-06731769042c-rep-699f4948c5-bmgdn    1/1       Running   0          6m
    pvc-b6cd585c-73e8-11e8-9f52-06731769042c-rep-699f4948c5-djmv6    1/1       Running   1          6m
    pvc-b6cd585c-73e8-11e8-9f52-06731769042c-rep-699f4948c5-thl82    1/1       Running   0          6m
    pvc-d2e08c54-733e-11e8-9f52-06731769042c-ctrl-949dd445f-czm2b    2/2       Running   0          20h
    pvc-d2e08c54-733e-11e8-9f52-06731769042c-rep-878485f7-qnvd9      1/1       Running   0          20h
    pvc-d2e08c54-733e-11e8-9f52-06731769042c-rep-878485f7-s5nsn      1/1       Running   0          20h
    pvc-d2e08c54-733e-11e8-9f52-06731769042c-rep-878485f7-xmdpc      1/1       Running   0          20h
    ```
    
5.  List persistent volumes and verify that `demo-snap-vol-claim` exist claimed by the `snapshot-promoter`.

    ```
    $ kubectl get pv
    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                         STORAGECLASS        REASON    AGE
    pvc-2dcb6cfd-733a-11e8-9f52-06731769042c   400M       RWO            Delete           Bound     default/pgdata-pgset-1        openebs-standard              20h
    pvc-b6cd585c-73e8-11e8-9f52-06731769042c   1G         RWO            Delete           Bound     default/demo-snap-vol-claim   snapshot-promoter             5m
    pvc-d2e08c54-733e-11e8-9f52-06731769042c   1G         RWO            Delete           Bound     default/demo-vol1-claim       openebs-standard              20h
    ```
    
6.  Once the green pod is in running state, you can check the integrity of files which were created earlier before taking the snapshot.

    ```
    $ kubectl exec -it green cat /tmp/pod-out.txt
    ...
    Mon Jun 18 21:31:19 UTC 2018
    Mon Jun 18 21:31:20 UTC 2018
    Mon Jun 18 21:31:21 UTC 2018
    ...
    ```






----------


5.  List snapshots:

    ```
    $ kubectl get volumesnapshot 
    NAME            AGE 
    snapshot-blue   18s
    ```

6.  The output above shows that your snapshot was created successfully. You can also check the snapshot-controller’s logs to verify this by using the following commands. 

    ```
    $ kubectl create -f snapshot.yaml
    volumesnapshot.volumesnapshot.external-storage.k8s.io "snapshot-blue" created
    ubuntu@ip-172-23-1-130:~$ kubectl get volumesnapshot
    NAME            AGE
    snapshot-blue   11m
    ubuntu@ip-172-23-1-130:~$ kubectl get volumesnapshot -o yaml
    apiVersion: v1
    items:
    - apiVersion: volumesnapshot.external-storage.k8s.io/v1
      kind: VolumeSnapshot
      metadata:
        clusterName: ""
        creationTimestamp: 2018-06-18T21:42:50Z
        generation: 1
        labels:
          SnapshotMetadata-PVName: pvc-d2e08c54-733e-11e8-9f52-06731769042c
          SnapshotMetadata-Timestamp: "1529358170229916897"
        name: snapshot-blue
        namespace: default
        resourceVersion: "26424"
        selfLink: /apis/volumesnapshot.external-storage.k8s.io/v1/namespaces/default/volumesnapshots/snapshot-blue
        uid: 8c2f17bc-7340-11e8-9f52-06731769042c
      spec:
        persistentVolumeClaimName: demo-vol1-claim
        snapshotDataName: k8s-volume-snapshot-8c3bf11f-7340-11e8-8b06-0a580a020205
      status:
        conditions:
        - lastTransitionTime: 2018-06-18T21:42:50Z
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

You can delete a snapshot that you have created which will also delete the corresponding Volume Snapshot Data resource from Kubernetes.

1.  The following command will delete the snapshot you have created.
    
    ```
    $ kubectl delete -f snapshot.yaml
    ```
 

# Beyond the Basics: OpenEBS - Container Attached Storage (CAS) for Kubernetes 
[OpenEBS](https://www.openebs.io) solution is a new approach to solving persistent storage issues for Kubernetes stateful applications. It provides a true cloud native storage solution to the containerized applications, in that the storage software itself is containerized. In other words, with OpenEBS each stateful workload is allocated a dedicated storage controller, which increases the agility and flexibility of data protection operations and granularity of storage policies of the given stateful workload.

Please read the [Introduction to OpenEBS](https://docs.openebs.io/docs/next/introduction.html), and also [Concepts](https://docs.openebs.io/docs/next/conceptscas.html) to familirize yourself with OpenEBS and its arhitecture. We will discuss all in details in the following workshop.

In this workshop, we will go over some of the basics that have emerged in Kubernetes around persistent workloads. We will go over Persistent Volumes (PVs), Persistent Volume Claims (PVCs) and how Kubernetes provisions them. 

We will also discuss the dynamic provisioners and how they can be used to setup PVs and PVCs dynamically and show how we can build a resilient storage system using Container Attached Storage (CAS) concepts. We will go over Storage Engines, Storage Schemas, and Policies and how they can help you build a fully DevOps centric deployment with persistent storage, while not losing any of agility that Kubernetes provides. 

We will finish the workshop with K8s snapshots and how they can be used to create 50 shades of green/blue CI/CD pipelines.


## Objectives
After you complete this course, you'll be able to: 
- Explain benefits of CAS
- Download and install OpenEBS in your cluster
- Deploy stateful workloads
- Modify and create your own storage classes 
- Use metrics, to observe services
- Perform simple green/blue deployments
- Take snapshot of your stateful workloads


## Prerequisites / Preparation for the workshop
You need to either BYOKC (Bring Your Own Kubernetes Cluster) or pick one of the options below (local or cloud) and perform the steps prior to the workshop.
1) Local: [A laptop with sufficient CPU and memory to run Kubernetes locally using Minikube on Virtualbox VM](preq/LOCAL.md)
2) Cloud: [A subscription with a cloud provider (AWS) and StackPointCloud account (Free 30 Day trial available) to run a K8s cluster](preq/CLOUD.md)

You also need to have your development environment with all your favorite tools with you. 
And finally, interest in figuring out how to use K8s with persistent workloads.

You should have a basic understanding of containers, Kubernetes, CAS and OpenEBS. If you have no experience with those, go through the following documentation before the workshop:
1. [Get started with Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
2. [Get started with OpenEBS](https://docs.openebs.io/docs/next/introduction.html)


## Workshop setup
- [Exercise 1 - Accessing and validating your Kubernetes cluster](exercise-1/README.md)
- [Exercise 2 - Installing OpenEBS](exercise-2/README.md)
- [Exercise 3 - Deploying stateful workload on OpenEBS](exercise-3/README.md)

## Configuring OpenEBS
- [Exercise 4 - Storage Classes](exercise-4/README.md)
- [Exercise 5 - Configuring Node Affinity](exercise-5/README.md)

## (Optional) Monitoring OpenEBS
- [Exercise 6 - ABC](exercise-6/README.md)

## Blue/Green Deployment
- [Exercise 7 - Rollout your application with Blue/Green deployment](exercise-7/README.md)

## (Optional) Introduction to E2E Testing
- [Exercise 8 - Litmus](exercise-8/README.md

---
layout: post
title: Introduction to Kubernetes
---

Kubernetes is an open-source platform for deploying and operating containerized applications. It helps keep applications running, scale them when demand changes, and provide stable ways for other services to reach them. Kubernetes is often shortened to **K8s**.

Running one container locally is straightforward. Operating many containers across several machines is harder: a machine can fail, an application may need more copies, and containers need a reliable way to find one another. Kubernetes coordinates that work by letting you describe the state you want and continually working toward it.

## How a Kubernetes cluster works

A Kubernetes cluster is made up of a **control plane** and one or more **worker nodes**:

- The **control plane** manages the cluster and works to keep its actual state aligned with the desired state.
- The **API server** is the entry point to the Kubernetes API. `kubectl`, cluster components, and other clients use it to read or change cluster resources.
- The **scheduler** watches for Pods that have not been assigned to a node and chooses a suitable worker node for each one. It makes the placement decision; it does not run the Pod itself.
- **Worker nodes** provide compute resources and run application workloads.
- The **kubelet** runs on each worker node. It communicates with the API server and makes sure the containers described by assigned Pods are running, using the node's container runtime.

The control plane also includes controllers that respond to changes and work toward the desired state, as well as a data store for cluster information. `kubectl` is the command-line client you can use to send requests to the API server; it is not itself a cluster component.

You generally describe resources in YAML files and submit them to the cluster. Kubernetes compares the desired state in those files with the current state and takes action when they differ. For example, if you request two copies of an application and one stops, Kubernetes attempts to start another.

## The main building blocks

### Pod

A **Pod** is the smallest deployable unit in Kubernetes. It contains one or more tightly related containers that share networking and storage, and it is assigned to a worker node to run. You usually do not create Pods directly for an application; a controller creates and replaces them as needed.

### Deployment

A **Deployment** manages a set of interchangeable Pods. You specify the container image, the number of replicas, and other settings. The Deployment's controller creates Pods and replaces them if they fail. It can also manage updates to the application.

### Service

Pods can be replaced, so their individual IP addresses are not a stable way to connect to an application. A **Service** provides a stable network endpoint and routes traffic to matching Pods.

Other resources, such as ConfigMaps, Secrets, and PersistentVolumes, handle configuration, sensitive values, and data that needs to outlive a Pod. They are useful once you move beyond a simple stateless application.

## Deploy an example application

You need access to a Kubernetes cluster and a configured `kubectl` context. For local practice, tools such as Minikube or kind can create a cluster on your computer. Check that `kubectl` is pointing to the cluster you intend to use before applying files:

```bash
kubectl config current-context
kubectl get nodes
```

Create a file named `web.yaml` with a Deployment and a Service:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.29
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
---
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  ports:
    - name: http
      port: 80
      targetPort: 80
  type: ClusterIP
```

The Deployment requests two Nginx Pods. Its selector matches the `app: web` label on the Pod template. The Service uses the same label to find those Pods and gives them a stable endpoint inside the cluster.

Apply the file and check the resources:

```bash
kubectl apply -f web.yaml
kubectl get deployments
kubectl get pods
kubectl get services
```

Wait until the Deployment reports that its available replicas match the requested replicas.

## Scale the Deployment with an HPA

A **Horizontal Pod Autoscaler (HPA)** adjusts the number of replicas in a Deployment or another scalable workload based on observed metrics. For this example, it keeps between two and five Nginx Pods and targets average CPU utilization of 60 percent of the requested CPU.

CPU-based autoscaling needs working resource metrics in the cluster, commonly provided by Metrics Server. Check that Metrics Server is installed and that `kubectl top pods` returns CPU data. The container also needs a CPU request: here it is `100m`, or one tenth of a CPU core. Without a request, Kubernetes cannot calculate CPU utilization for this HPA.

Create a separate file named `web-hpa.yaml`:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
```

Apply the HPA after the Deployment is running, then inspect its status:

```bash
kubectl apply -f web-hpa.yaml
kubectl get hpa
kubectl get hpa web --watch
```

The HPA periodically compares current CPU metrics with the target and updates the Deployment's replica count when needed. Scaling is not instantaneous, and it will not add replicas when there is no sustained CPU demand. Once the HPA is managing the Deployment, let it control the replica count rather than repeatedly applying a fixed `spec.replicas` value.

To access the Service from your computer without exposing it outside the cluster, open a port-forward in one terminal:

```bash
kubectl port-forward service/web 8080:80
```

While that command is running, visit `http://localhost:8080` in a browser. Stop the port-forward with `Ctrl+C` when you are finished.

When you are done practicing, remove the resources created from the file:

```bash
kubectl delete -f web-hpa.yaml
kubectl delete -f web.yaml
```

## Useful kubectl commands

These commands help inspect and troubleshoot the example:

```bash
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl logs deployment/web
kubectl get events --sort-by=.metadata.creationTimestamp
```

`get` lists resources, `describe` shows details and recent events for one resource, and `logs` prints output from containers. Replace `<pod-name>` with a Pod name shown by `kubectl get pods`; do not type the angle brackets.

If a Pod is not starting, `kubectl describe pod` often shows why, such as an image that cannot be pulled or a scheduling problem. Logs can help diagnose an application that starts but then exits or reports an error.

## What Kubernetes does not do

Kubernetes does not build your application or automatically make it reliable. You still need to build and publish container images, configure health checks and resource requests, protect credentials, and plan how application data is stored. It also adds operational complexity, so a single application may not need Kubernetes at all.

The key idea is to describe the outcome you want, then let Kubernetes controllers work to keep the cluster close to that desired state. Start with a small Deployment and Service, learn to inspect what the cluster is doing, and add more resources as your application needs them.

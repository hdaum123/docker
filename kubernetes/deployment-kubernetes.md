# Kubernetes - Deploying Nginx with a Deployment and Service

## Overview

In this task, I deployed an Nginx application to a Kubernetes cluster using two YAML configuration files:

- `nginx-deploy.yml` – Creates and manages the Nginx Deployment.
- `nginx-service.yml` – Creates a Service to expose the Deployment outside the cluster.

This task introduced the core Kubernetes concepts of **Deployments**, **Pods**, **ReplicaSets**, and **Services**, and demonstrated how Kubernetes manages applications declaratively using YAML files.

---

# Folder Structure

```text
kubernetes/
├── nginx-deploy.yml
└── nginx-service.yml
```

---

# Step 1 - Create the Deployment File

Create a file called:

```bash
nano nginx-deploy.yml
```

Paste in the following configuration:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  selector:
    matchLabels:
      app: nginx

  replicas: 3

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: daraymonsta/nginx-257:dreamteam
        ports:
        - containerPort: 80
```

---

# Understanding the Deployment

## apiVersion

```yaml
apiVersion: apps/v1
```

Specifies which Kubernetes API should be used.

Deployments belong to the **apps/v1** API.

---

## kind

```yaml
kind: Deployment
```

Defines the type of Kubernetes object being created.

In this case, we are creating a **Deployment**.

A Deployment is responsible for:

- creating Pods
- managing ReplicaSets
- replacing failed Pods
- performing rolling updates
- scaling applications

---

## metadata

```yaml
metadata:
  name: nginx-deployment
```

Assigns a name to the Deployment.

This is how Kubernetes identifies the Deployment.

---

## selector

```yaml
selector:
  matchLabels:
    app: nginx
```

The selector tells the Deployment which Pods belong to it.

It searches for Pods with the label:

```yaml
app: nginx
```

If the labels do not match, the Deployment will not manage those Pods.

---

## replicas

```yaml
replicas: 3
```

This tells Kubernetes to maintain **three running Pods**.

If:

- one Pod crashes
- one Pod is deleted
- one worker node fails

Kubernetes automatically creates a replacement to ensure three Pods are always running.

---

## template

Everything inside:

```yaml
template:
```

describes what each Pod should look like.

Each Pod will contain:

- one Nginx container
- running the specified Docker image

---

## labels

```yaml
labels:
  app: nginx
```

Labels are key-value pairs attached to Kubernetes objects.

These labels are later used by the Service to identify which Pods should receive incoming traffic.

---

## container

```yaml
containers:
- name: nginx
```

Defines the container inside the Pod.

---

## image

```yaml
image: daraymonsta/nginx-257:dreamteam
```

Specifies the Docker image Kubernetes should pull from Docker Hub.

If the image is not already present on the node, Kubernetes downloads it automatically.

---

## containerPort

```yaml
containerPort: 80
```

Indicates that the Nginx container listens on port **80**.

This is the port exposed inside the container.

---

# Step 2 - Create the Service File

Create another file:

```bash
nano nginx-service.yml
```

Paste in:

```yaml
apiVersion: v1

kind: Service

metadata:
  name: nginx-svc
  namespace: default

spec:
  ports:
  - nodePort: 30001
    port: 80
    targetPort: 80

  selector:
    app: nginx

  type: NodePort
```

---

# Understanding the Service

## apiVersion

```yaml
apiVersion: v1
```

Services belong to the core Kubernetes API.

---

## kind

```yaml
kind: Service
```

Creates a Kubernetes Service.

A Service provides a stable network endpoint that allows traffic to reach Pods, even if Pods are recreated or their IP addresses change.

---

## metadata

```yaml
metadata:
  name: nginx-svc
```

Names the Service.

---

## namespace

```yaml
namespace: default
```

Places the Service into the default Kubernetes namespace.

---

## selector

```yaml
selector:
  app: nginx
```

The Service searches for Pods with the label:

```yaml
app: nginx
```

This label matches the Pods created by the Deployment.

Without matching labels, the Service would not know where to send traffic.

---

## targetPort

```yaml
targetPort: 80
```

The port inside each Pod where the application is listening.

Nginx listens on port **80**.

---

## port

```yaml
port: 80
```

The internal Service port inside the Kubernetes cluster.

Other Pods communicate with the Service using this port.

---

## nodePort

```yaml
nodePort: 30001
```

Exposes the Service externally.

NodePorts must be within the Kubernetes range:

```text
30000 - 32767
```

Traffic sent to:

```text
<Node-IP>:30001
```

is forwarded to:

```text
Service Port 80
```

which then forwards it to:

```text
Container Port 80
```

---

## type

```yaml
type: NodePort
```

This makes the Service accessible outside the Kubernetes cluster.

Common Service types include:

| Type | Purpose |
|-------|----------|
| ClusterIP | Internal communication between Pods (default). |
| NodePort | Exposes the application on a port of every worker node. |
| LoadBalancer | Creates a cloud load balancer (AWS, Azure, GCP). |

---

# Step 3 - Navigate to the Kubernetes Directory

Move into the folder containing the YAML files.

```bash
cd kubernetes
```

---

# Step 4 - View Existing Resources

Check what resources already exist in the cluster.

```bash
kubectl get all
```

This command displays:

- Pods
- Services
- Deployments
- ReplicaSets
- and other Kubernetes resources

---

# Step 5 - Create the Deployment

Deploy the application using the Deployment YAML file.

```bash
kubectl create -f nginx-deploy.yml
```

Kubernetes creates:

- Deployment
- ReplicaSet
- Three Pods

---

# Step 6 - Verify the Deployment

View the Deployment.

```bash
kubectl get deployment
```

Example output:

```text
NAME                READY   UP-TO-DATE   AVAILABLE
nginx-deployment    3/3     3            3
```

This confirms that all three replicas are running successfully.

---

# Step 7 - Verify the Pods

List all running Pods.

```bash
kubectl get pods
```

Example output:

```text
nginx-deployment-xxxxx
nginx-deployment-yyyyy
nginx-deployment-zzzzz
```

These Pods are managed automatically by the Deployment.

---

# Step 8 - Create the Service

Create the NodePort Service.

```bash
kubectl create -f nginx-service.yml
```

The Service is linked to the Pods using the label:

```yaml
app: nginx
```

---

# Step 9 - Apply Configuration Changes

After editing either YAML file, apply the updated configuration.

Deployment:

```bash
kubectl apply -f nginx-deploy.yml
```

Service:

```bash
kubectl apply -f nginx-service.yml
```

Unlike `create`, the `apply` command updates existing resources instead of attempting to create new ones.

---
---

# Step 10 - Describe the Deployment

To view detailed information about the Deployment, use:

```bash
kubectl describe deployment nginx-deployment
```

This command displays detailed information including:

- Deployment name
- Namespace
- Labels
- Replica count
- Container image
- Pod template
- Rolling update strategy
- Events
- Current status

This is useful for troubleshooting and verifying that Kubernetes is using the correct Docker image.

---

# Step 11 - Scaling the Deployment

One of the benefits of using a Deployment is that it can be scaled without downtime.

To increase the number of replicas, use:

```bash
kubectl scale --current-replicas=5 --replicas=6 deployment/nginx-deployment
```

### Explanation

- `--current-replicas=5` confirms Kubernetes expects the Deployment to currently have five replicas before scaling.
- `--replicas=6` tells Kubernetes to increase the Deployment to six Pods.
- `deployment/nginx-deployment` specifies which Deployment should be scaled.

Kubernetes creates the additional Pod while the existing Pods continue serving traffic, meaning the application remains available during the scaling process.

You can verify the new replica count by running:

```bash
kubectl get deployment
```

or

```bash
kubectl get pods
```

---

# Step 12 - Editing a Deployment

Instead of modifying the YAML file, Kubernetes also allows live editing of a Deployment.

```bash
kubectl edit deployment nginx-deployment
```

This opens the Deployment configuration in your default text editor.

For example, you can edit:

```yaml
replicas: 3
```

to

```yaml
replicas: 5
```

After saving and closing the editor, Kubernetes immediately applies the changes and updates the Deployment.

This is useful for quick changes, although updating the YAML file and using `kubectl apply` is generally preferred for version-controlled environments.

---

# Step 13 - Testing the Service

Once the NodePort Service has been created, test that it is accessible.

```bash
curl localhost:30001
```

If the Service is working correctly, the command returns the default Nginx HTML page.

This confirms that:

- the Service is running
- the Service is forwarding traffic correctly
- the Pods are responding to requests

---

# Step 14 - Deleting a Pod

Delete one of the Pods managed by the Deployment.

```bash
kubectl delete pod/nginx-deployment-67c68f6d64-bpddk
```

Although the Pod is deleted, the application remains available.

This is because the Deployment manages a ReplicaSet, which constantly checks that the desired number of replicas are running.

After detecting that one Pod has been removed, Kubernetes automatically creates a replacement Pod to maintain the desired state.

You can observe this by running:

```bash
kubectl get pods
```

You'll notice the deleted Pod disappears and a new Pod with a different name is created automatically.

---

# Additional Commands Used

```bash
kubectl describe deployment nginx-deployment

kubectl scale --current-replicas=5 --replicas=6 deployment/nginx-deployment

kubectl edit deployment nginx-deployment

curl localhost:30001

kubectl delete pod/nginx-deployment-67c68f6d64-bpddk

kubectl get pods
```

---

# Additional Learning Points

- `kubectl describe` provides detailed information about a Kubernetes resource.
- Deployments can be scaled without downtime.
- `kubectl scale` quickly changes the number of replicas.
- `kubectl edit` allows live editing of Kubernetes resources.
- A NodePort Service can be tested using `curl`.
- Pods managed by a Deployment are automatically recreated if they are deleted.
- The ReplicaSet ensures the desired number of Pods are always running.

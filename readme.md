# Kubernetes Deployment of the NodeJS TTT App and MongoDB

## Aim

The aim of this task was to deploy the NodeJS Tic Tac Toe application using Kubernetes.

The task was completed in two stages:

- Deploy the NodeJS application using a Deployment with 3 replicas and expose it using a NodePort Service.
- Deploy MongoDB using a Deployment with 1 replica and connect it to the application using a ClusterIP Service.

---

## Prerequisites

Before starting, I needed:

- Docker Desktop with Kubernetes enabled
- kubectl installed
- A NodeJS application image available on Docker Hub
- The official MongoDB Docker image

The images used were:

- **NodeJS:** `hdaum123/tech610-tttapp:1.2.0`
- **MongoDB:** `mongo:8.0`

---

## Project Structure

```text
local-app-db-deploy/
├── app-deploy.yml
├── app-service.yml
├── mongodb-deploy.yml
└── mongodb-service.yml
```

---

## File Descriptions

### app-deploy.yml

Creates the Deployment for the NodeJS application.

- Deploys 3 replicas of the application.
- Pulls the NodeJS Docker image from Docker Hub.
- Exposes port 3000.
- Configures the `MONGODB_URI` environment variable so the application can connect to MongoDB.

---

### app-service.yml

Creates a NodePort Service for the NodeJS application.

- Exposes the application outside the Kubernetes cluster.
- Routes incoming requests to the NodeJS Pods.
- Uses the `app: nodejs` label to identify the Pods.
- Uses NodePort **30001**.

---

### mongodb-deploy.yml

Creates the MongoDB Deployment.

- Deploys a single MongoDB Pod.
- Uses the official MongoDB Docker image.
- Exposes MongoDB on port 27017.

---

### mongodb-service.yml

Creates an internal ClusterIP Service for MongoDB.

- Allows the NodeJS application to communicate with MongoDB.
- Uses the label `app: mongodb`.
- Exposes port 27017 inside the cluster.

---

## Deploy the Application

Deploy the MongoDB resources:

```bash
kubectl apply -f mongodb-deploy.yml
kubectl apply -f mongodb-service.yml
```

Deploy the NodeJS application:

```bash
kubectl apply -f app-deploy.yml
kubectl apply -f app-service.yml
```

Verify the deployment:

```bash
kubectl get all
```

---

## Architecture

```text
                    Browser
                       |
              localhost:30001
                       |
                       v
              NodePort Service
                 nodejs-svc
                       |
          +------------+------------+
          |            |            |
          v            v            v
      NodeJS Pod   NodeJS Pod   NodeJS Pod
          |            |            |
          +------------+------------+
                       |
      mongodb://mongodb-service:27017/tictactoe
                       |
                       v
             MongoDB ClusterIP Service
                       |
                       v
                  MongoDB Pod
```

---

## Useful Commands

### Deploy

```bash
kubectl apply -f app-deploy.yml
kubectl apply -f app-service.yml

kubectl apply -f mongodb-deploy.yml
kubectl apply -f mongodb-service.yml
```

### Check Resources

```bash
kubectl get all

kubectl get pods

kubectl get services

kubectl get deployments
```

### Check Logs

```bash
kubectl logs deployment/nodejs-deployment

kubectl logs deployment/mongodb-deployment
```

### Check MongoDB Connection

```bash
kubectl exec deployment/nodejs-deployment -- printenv MONGODB_URI
```

### Delete Resources

```bash
kubectl delete -f app-deploy.yml
kubectl delete -f app-service.yml

kubectl delete -f mongodb-deploy.yml
kubectl delete -f mongodb-service.yml
```

---

## Final Result

The final deployment contained:

- 3 NodeJS application Pods
- 1 MongoDB Pod
- 1 NodePort Service
- 1 ClusterIP Service

The NodeJS application communicates with MongoDB using the Kubernetes Service name rather than a fixed IP address, demonstrating Kubernetes service discovery.

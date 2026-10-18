# Kubernetes Hands-On Exercises

A collection of hands-on Kubernetes exercises using Minikube, Docker, and kubectl.

## Exercise 1: Hello Pod

### Objective
Deploy an Nginx web server on a local Kubernetes cluster using Minikube.

### Technology Stack
- Docker Desktop
- Kubernetes
- Minikube
- kubectl
- Nginx

### Architecture
Browser -> NodePort Service -> Nginx Pod

### Prerequisites
- Docker Desktop installed and running
- Minikube installed
- kubectl installed

### Step 1: Start the Cluster
```powershell
minikube start --driver=docker
```

### Step 2: Deploy the Application
Run these commands from the exercise-1-hello-pod directory:

```powershell
kubectl apply -f pod.yaml
kubectl apply -f service.yaml
```

### Step 3: Verify Resources
```powershell
kubectl get nodes
kubectl get pods
kubectl get services
```

### Step 4: Access the Application
```powershell
minikube service hello-k8s --url
```

Open the URL printed in the terminal. With Docker on Windows, keep the terminal running if required.

### Step 5: Clean Up
```powershell
kubectl delete -f service.yaml
kubectl delete -f pod.yaml
```

### Concepts Learned
- Kubernetes Pods
- Container images
- Kubernetes Services
- NodePort
- YAML manifests
- Deploying and accessing a containerized application

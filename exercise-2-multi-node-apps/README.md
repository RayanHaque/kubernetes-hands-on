# Exercise 2: Multi-Node Kubernetes Cluster with Multiple Applications and Replicas

## Objective
Deploy two simple Flask applications to a three-node Minikube cluster:
- **Product Catalog**: 2 replicas
- **Shopping Cart**: 3 replicas

Pod anti-affinity is configured so replicas of each application are scheduled on different nodes when the cluster has enough available nodes.

## Architecture
Browser / curl → NodePort Service → Kubernetes Pods → Flask application

## Technology Stack
- Docker Desktop
- Docker
- Minikube
- Kubernetes
- kubectl
- Python
- Flask

## Prerequisites
Install and start Docker Desktop, Minikube, and kubectl. Run the commands below in PowerShell on Windows.

> This exercise uses a separate Minikube profile named `devops-multinode`, so it does not require deleting your Exercise 1 cluster.

## 1. Create a three-node cluster
```powershell
minikube start --nodes 3 -p devops-multinode --driver=docker
kubectl --context=devops-multinode get nodes
```

## 2. Build the Docker images
Run these commands from this exercise folder:
```powershell
docker build -t product-catalog:latest -f Dockerfile.product .
docker build -t shopping-cart:latest -f Dockerfile.shopping .
```

## 3. Load the images into Minikube
```powershell
minikube -p devops-multinode image load product-catalog:latest
minikube -p devops-multinode image load shopping-cart:latest
```

## 4. Deploy the applications
The manifests use the `default` namespace consistently.
```powershell
kubectl --context=devops-multinode apply -f product_catalog_deployment.yaml
kubectl --context=devops-multinode apply -f shopping_cart_deployment.yaml
kubectl --context=devops-multinode apply -f product_catalog_service.yaml
kubectl --context=devops-multinode apply -f shopping_cart_service.yaml
```

## 5. Verify replicas and node placement
```powershell
kubectl --context=devops-multinode get pods -o wide
kubectl --context=devops-multinode get deployments
kubectl --context=devops-multinode get services
```

Expected replica counts:
- `product-catalog`: 2
- `shopping-cart`: 3

Pod placement may take a little time. If a Pod remains Pending, inspect it with `kubectl --context=devops-multinode describe pod <pod-name>`.

## 6. Access and test the services
Open separate PowerShell windows for the following commands if both services need to remain accessible at the same time:
```powershell
minikube -p devops-multinode service product-catalog-service --url
minikube -p devops-multinode service shopping-cart-service --url
```

Keep the relevant terminal window open while using each URL. Use the actual URLs printed by Minikube; the ports can vary.

Test the Product Catalog endpoint by replacing `<PRODUCT_URL>` with its printed URL:
```powershell
curl.exe <PRODUCT_URL>/products
```

Test the Shopping Cart endpoint by replacing `<CART_URL>` with its printed URL:
```powershell
curl.exe <CART_URL>/cart
```

Add an item to the cart:
```powershell
curl.exe -X POST <CART_URL>/cart -H "Content-Type: application/json" -d "{\"id\":1,\"name\":\"Laptop\",\"quantity\":1}"
```

The cart is stored in application memory, so it is temporary and is not shared between replicas. This exercise demonstrates replicas and Services, not persistent cart storage.

## 7. Clean up this exercise
Delete only the resources created for this exercise:
```powershell
kubectl --context=devops-multinode delete -f shopping_cart_service.yaml
kubectl --context=devops-multinode delete -f product_catalog_service.yaml
kubectl --context=devops-multinode delete -f shopping_cart_deployment.yaml
kubectl --context=devops-multinode delete -f product_catalog_deployment.yaml
```

To stop this profile when finished:
```powershell
minikube -p devops-multinode stop
```

To delete this profile and its cluster entirely (optional):
```powershell
minikube -p devops-multinode delete
```

## Concepts Learned
- Multi-node Minikube clusters
- Dockerizing Python applications
- Kubernetes Deployments and replicas
- Pod anti-affinity
- Kubernetes Services and NodePort
- Loading local images into Minikube
- Verifying Pod placement across nodes

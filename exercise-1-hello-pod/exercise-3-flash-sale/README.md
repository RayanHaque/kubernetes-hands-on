# Exercise 3: Scaling a Flask App on a Single Node Using ReplicaSets

## Real-life use case: E-commerce flash sale

During a flash sale, request traffic can rise suddenly. A ReplicaSet keeps the desired number of identical Pods running. In this exercise, the app starts with 3 Pods, is scaled to 5, and one Pod is deleted to observe automatic replacement.

> This exercise demonstrates Kubernetes replica management. It does not generate 10,000 requests per minute or benchmark real traffic.

## Objectives

- Understand Pods and ReplicaSets.
- Deploy a Flask flash-sale API.
- Scale the ReplicaSet from 3 to 5 Pods.
- Delete a Pod and observe automatic replacement.
- Inspect which node runs each Pod.
- Access endpoints that report the Pod handling the request.

## Files

- `app.py` — Flask endpoints `/`, `/buy`, and `/health`.
- `Dockerfile` — packages Flask and Gunicorn.
- `flashsale-replicaset.yaml` — ReplicaSet and ClusterIP Service.
- `screenshots/` — save your own screenshots here.

## Prerequisites

Docker Desktop, Minikube, kubectl, and Git must be installed.

This guide uses a separate Minikube profile named `flashsale` so it does not delete or modify the clusters used by other exercises.

## 1. Start a separate single-node Minikube profile

Run in Windows PowerShell from this folder:

```powershell
minikube start -p flashsale --nodes=1 --driver=docker
kubectl --context=flashsale get nodes
```

You should see one node with status `Ready`.

## 2. Build the image inside this Minikube profile

Run from the folder containing `app.py` and `Dockerfile`:

```powershell
minikube -p flashsale image build -t flashsale:1.0 .
```

The YAML uses `imagePullPolicy: Never`, so Kubernetes uses the image already built into this Minikube profile.

## 3. Create the ReplicaSet and Service

```powershell
kubectl --context=flashsale apply -f flashsale-replicaset.yaml
kubectl --context=flashsale rollout status rs/flashsale-rs
```

A ReplicaSet does not support `kubectl rollout status` consistently across all kubectl versions. If that command reports that the resource type is unsupported, use the checks below instead:

```powershell
kubectl --context=flashsale get rs
kubectl --context=flashsale get pods -l app=flashsale -o wide
kubectl --context=flashsale get services
```

Wait until all 3 Pods show `1/1` in READY and `Running` in STATUS.

## 4. Test the service

The Service is `ClusterIP`, so use port-forwarding to access it from Windows. Keep this PowerShell window open:

```powershell
kubectl --context=flashsale port-forward service/flashsale-svc 8080:80
```

Open a second PowerShell window and run:

```powershell
curl.exe http://127.0.0.1:8080/
curl.exe "http://127.0.0.1:8080/buy?user=123"
curl.exe http://127.0.0.1:8080/health
```

The `/` and `/buy` responses include `pod` or `served_by_pod`, which lets you observe which Pod handled a request. Repeated requests may or may not hit different Pods; the Service does not guarantee round-robin results for every request.

## 5. Scale from 3 Pods to 5 Pods

In a PowerShell window:

```powershell
kubectl --context=flashsale scale rs/flashsale-rs --replicas=5
kubectl --context=flashsale get rs
kubectl --context=flashsale get pods -l app=flashsale -o wide
```

Wait until 5 Pods are `Running` and ready. Capture the ReplicaSet and Pod output.

## 6. Delete one Pod and observe self-healing

List the Pods:

```powershell
kubectl --context=flashsale get pods -l app=flashsale
```

Copy one actual Pod name from the output and replace `<pod-name>` below:

```powershell
kubectl --context=flashsale delete pod <pod-name>
```

Then watch the replacement:

```powershell
kubectl --context=flashsale get pods -l app=flashsale -w
```

Press `Ctrl+C` when you have observed the new Pod. Confirm there are still 5 ready Pods:

```powershell
kubectl --context=flashsale get rs
kubectl --context=flashsale get pods -l app=flashsale -o wide
```

## 7. Inspect details and logs

```powershell
kubectl --context=flashsale describe rs flashsale-rs
kubectl --context=flashsale logs -l app=flashsale --tail=30
```

## Screenshots to capture

Save screenshots you actually take in this folder. Use these filenames:

- `screenshots/node-ready.png` — the one-node cluster is Ready.
- `screenshots/initial-three-pods.png` — the initial 3 Pods are Running.
- `screenshots/service-test.png` — successful `/`, `/buy`, and/or `/health` response.
- `screenshots/scaled-five-pods.png` — 5 ready Pods after scaling.
- `screenshots/pod-replaced.png` — output showing a replacement Pod after deletion.
- `screenshots/pod-distribution.png` — `kubectl get pods -o wide`.

Add the following section to this README after you save the screenshots:

```markdown
## Screenshots

### Single-Node Cluster
![Node Ready](screenshots/node-ready.png)

### Initial Three Pods
![Initial Three Pods](screenshots/initial-three-pods.png)

### Flash Sale API
![Service Test](screenshots/service-test.png)

### Scaled to Five Pods
![Five Pods](screenshots/scaled-five-pods.png)

### Pod Self-Healing
![Pod Replaced](screenshots/pod-replaced.png)

### Pod Distribution
![Pod Distribution](screenshots/pod-distribution.png)
```

## Questions and answers

1. **How many replicas are configured initially?** Three.
2. **How many Pods should exist after applying the YAML?** Three, once the Pods are ready.
3. **What happens when the ReplicaSet is scaled to five?** Kubernetes creates two additional Pods to reach the desired count of five.
4. **What happens when one Pod is deleted?** The ReplicaSet notices the missing Pod and creates a replacement to return to five.
5. **How does Kubernetes maintain the desired count?** The ReplicaSet controller continuously compares the actual matching Pods with the configured replica count and creates or removes Pods as needed.
6. **How many nodes are used in this exercise?** One.
7. **Where are the Pods running?** All five Pods run on the single Minikube node.

## Important notes

- A ReplicaSet manages Pods directly. A Deployment is generally preferred for application updates and rollout management, and it manages ReplicaSets for you.
- This app returns simulated purchase responses; it does not process real payments or store orders.
- A single node can run multiple Pods, but scaling Pod count does not add node capacity. The node still has finite CPU and memory.
- The `served_by_pod` field identifies the Pod that served a request.

## Cleanup

Only run these if you want to remove this exercise's resources:

```powershell
kubectl --context=flashsale delete -f flashsale-replicaset.yaml
minikube stop -p flashsale
```

To delete this exercise's separate cluster entirely:

```powershell
minikube delete -p flashsale
```

Do not delete the other Minikube profiles used by your other exercises.

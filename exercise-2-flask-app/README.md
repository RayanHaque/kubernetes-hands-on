# Exercise 2: Deploy a Flask App on Minikube

## Objective
Deploy a small Python Flask application to Kubernetes using Minikube, a Docker image, a Deployment, and a NodePort Service.

## Prerequisites
- Docker Desktop is running.
- Minikube and `kubectl` are installed.
- You have started your Minikube cluster.

> These steps are written for **Windows PowerShell** and the Docker driver. They use `minikube image build` so you do not need the Linux-only `eval $(minikube docker-env)` command from the original instructions.

## Files
- `app.py` — the Flask web application.
- `Dockerfile` — instructions to package the app into an image.
- `flask-deployment.yaml` — Kubernetes Deployment and Service.

## Step 1: Open PowerShell in this folder

From your repository root, run:

```powershell
cd "$HOME\Documents\kubernetes-hands-on\exercise-3-flask-app"
```

If Minikube is not running yet, start it:

```powershell
minikube start --driver=docker
```

Check the cluster:

```powershell
minikube status
kubectl cluster-info
```

## Step 2: Build the Flask Docker image inside Minikube

Run this from the folder containing the `Dockerfile`:

```powershell
minikube image build -t flask-app:latest .
```

Check that the image is available:

```powershell
minikube image ls
```

## Step 3: Deploy the application

Apply the YAML file:

```powershell
kubectl apply -f flask-deployment.yaml
```

Wait for the Deployment to become ready:

```powershell
kubectl rollout status deployment/flask-app
```

Check the Deployment and Pod:

```powershell
kubectl get deployments
kubectl get pods -l app=flask-app
```

The Pod's status should become `Running` and the Deployment should show `1/1` ready.

## Step 4: Inspect the deployment and logs

```powershell
kubectl describe deployment flask-app
```

Get the current Pod name and view its logs:

```powershell
kubectl logs -l app=flask-app
```

You should see Flask starting and listening on port `15000`.

## Step 5: Access the application

Check the Service:

```powershell
kubectl get services
```

In PowerShell, run:

```powershell
minikube service flask-app-service --url
```

Keep that PowerShell window open if Minikube says the tunnel/terminal must remain open. Copy the URL it prints and open it in your browser, or use a **second** PowerShell window to test it. For example, if it prints `http://127.0.0.1:36157`, run:

```powershell
curl.exe http://127.0.0.1:36157
```

Use the URL actually printed on your computer; the port may be different. Expected response:

```text
Hello from Flask on Kubernetes!
```

**Important:** The Service's `port` is `15000`, but a `NodePort` Service is reached through the URL provided by `minikube service ... --url`. Do not assume the app is available at `http://127.0.0.1:15000`.

## How the pieces fit together

- **Flask app:** responds to HTTP requests on port `15000`.
- **Docker image:** packages the Python app and Flask dependency.
- **Deployment:** tells Kubernetes to keep one copy (replica) of the app running.
- **Pod:** runs the app's container.
- **Service:** finds Pods labelled `app: flask-app` and routes traffic to container port `15000`.
- **NodePort + Minikube URL:** provides a way to reach the app from your computer.

## Troubleshooting

### Pod shows `ImagePullBackOff` or `ErrImageNeverPull`
Build the image into the active Minikube cluster:

```powershell
minikube image build -t flask-app:latest .
```

Then restart the Deployment:

```powershell
kubectl rollout restart deployment/flask-app
kubectl get pods -l app=flask-app
```

### Pod is not running
Check the Pod details and logs:

```powershell
kubectl describe pod -l app=flask-app
kubectl logs -l app=flask-app
```

### The browser cannot connect
Make sure the `minikube service flask-app-service --url` command is still running if it asks you to keep the terminal open. Use the exact URL it printed.

## Cleanup

Remove the Kubernetes resources created by this exercise:

```powershell
kubectl delete -f flask-deployment.yaml
```

To stop Minikube without deleting the cluster:

```powershell
minikube stop
```

Only run `minikube delete` if you intend to delete the whole Minikube cluster used by other exercises too.

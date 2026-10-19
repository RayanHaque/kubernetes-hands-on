# Exercise 4: Docker Networking with Multiple Containers

## Objective

Understand Docker bridge networking and configure a multi-container application.

## Scenario

This exercise uses three containers:

- **Flask:** a simple REST API on port 5001.
- **MySQL:** a database container.
- **Redis:** a cache container.

All three containers connect to a custom Docker bridge network named `my-bridge-net`.

## Files

- `app.py` - Flask REST API.
- `requirements.txt` - Python dependencies.
- `Dockerfile` - builds the Flask image.
- `screenshots/` - save your actual screenshots here.

## Task 1: Create a bridge network

Run in PowerShell from this folder:

```powershell
docker network create --driver bridge my-bridge-net
```

If Docker says the network already exists, continue to the next task.

## Task 2: Verify the network

```powershell
docker network ls
```

Find `my-bridge-net` and confirm its driver is `bridge`.

## Task 3: Inspect the network

```powershell
docker network inspect my-bridge-net
```

Initially, the Containers section may be empty. After starting the containers, inspect the network again to see them attached.

## Task 4: Build the Flask image

```powershell
docker build -t flask-api .
```

## Task 5: Start MySQL, Redis, and Flask

Start MySQL with a development password:

```powershell
docker run -d --name mysql --network my-bridge-net -e MYSQL_ROOT_PASSWORD=DevOnly123! mysql:8.0
```

Start Redis:

```powershell
docker run -d --name redis --network my-bridge-net redis:7
```

Start Flask:

```powershell
docker run -d --name flask --network my-bridge-net -p 5001:5001 flask-api
```

These commands use fixed container names. If a container with one of these names already exists, remove that old exercise container before running the command again.

Wait for MySQL to initialize, then check the containers:

```powershell
docker ps
```

All three should show a running status.

## Task 6: Test the Flask API

```powershell
curl.exe http://localhost:5001/about
```

Expected JSON:

```json
{
  "name": "Simple REST API",
  "version": "1.0",
  "description": "This is a simple REST API built with Flask."
}
```

## Task 7: Test container-to-container networking

Open a shell inside the Flask container:

```powershell
docker exec -it flask bash
```

Inside the container, run:

```bash
ping -c 4 mysql
ping -c 4 redis
```

You should see the container names resolve to IP addresses and receive replies.

Exit the container shell:

```bash
exit
```

You can also inspect the network from PowerShell:

```powershell
docker network inspect my-bridge-net
```

The Containers section should now list Flask, MySQL, and Redis.

## Task 8: Check logs

```powershell
docker logs flask
docker logs mysql
docker logs redis
```

## Screenshots

Save your actual screenshots inside this folder:

- `screenshots/network-created.png`
- `screenshots/network-inspect.png`
- `screenshots/containers-running.png`
- `screenshots/flask-api.png`
- `screenshots/ping-connectivity.png`

After taking the screenshots, add this section to display them:

### Custom Bridge Network
![Network Created](screenshots/network-created.png)

### Network Inspection
![Network Inspection](screenshots/network-inspect.png)

### Three Containers Running
![Containers Running](screenshots/containers-running.png)

### Flask API Response
![Flask API](screenshots/flask-api.png)

### Container Connectivity
![Ping Connectivity](screenshots/ping-connectivity.png)

## Questions and Answers

1. **What is a bridge network?** A Docker network that allows connected containers to communicate.
2. **Why use a custom bridge network?** It provides automatic container-name resolution and isolation from unrelated containers.
3. **How does Flask reach MySQL?** By using the hostname `mysql` and the MySQL port, normally 3306.
4. **How does Flask reach Redis?** By using the hostname `redis` and the Redis port, normally 6379.
5. **Why publish port 5001?** The `-p 5001:5001` option allows access to Flask from the host computer.
6. **Does this Flask API query MySQL or Redis?** No. This exercise demonstrates network connectivity only; the API does not yet read from or write to either service.

## Cleanup

Stop and remove the containers:

```powershell
docker stop flask mysql redis
docker rm flask mysql redis
```

Remove the custom network:

```powershell
docker network rm my-bridge-net
```

Only remove this network after removing its connected containers.

# Exercise 5: Docker Security with AppArmor

## Overview

This exercise explores container security using Docker and AppArmor on Ubuntu 24.04. A Flask application runs inside a Docker container while AppArmor provides mandatory access control through security profiles.

## Objectives

- Run a Flask application inside a Docker container.
- Understand how AppArmor restricts application behavior.
- Apply an AppArmor profile to a container.
- Test container behavior under different AppArmor profiles.
- Investigate permission denials using kernel audit logs.

## Tech Stack

- Ubuntu 24.04
- Docker
- AppArmor
- Python 3.12
- Flask
- Linux kernel audit logs

## Project Structure

```text
exercise-5-docker-apparmor/
├── app.py
├── Dockerfile
├── requirements.txt
├── docker-apparmor-demo
└── README.md
```

## Application

The Flask application provides two endpoints:

| Endpoint | Purpose |
|---|---|
| `/` | Returns an application status message |
| `/health` | Returns the health status |

## Build the Docker Image

```bash
docker build -t apparmor-flask-demo .
```

## Run with Docker's Default AppArmor Profile

```bash
sudo docker run -d \
  --security-opt="apparmor=docker-default" \
  -p 5000:5000 \
  --name apparmor-flask \
  apparmor-flask-demo
```

Test the health endpoint:

```bash
curl http://localhost:5000/health
```

Expected response:

```json
{"status":"healthy"}
```

Check the active profile:

```bash
sudo docker exec apparmor-flask cat /proc/self/attr/current
```

Clean up:

```bash
sudo docker rm -f apparmor-flask
```

## Custom AppArmor Profile

The `docker-apparmor-demo` file contains a custom security profile intended to restrict access to sensitive files and privileged operations while allowing the Flask application to run.

Copy the profile into the system configuration directory:

```bash
sudo cp docker-apparmor-demo /etc/apparmor.d/docker-apparmor-demo
```

Load the profile:

```bash
sudo apparmor_parser -r /etc/apparmor.d/docker-apparmor-demo
```

Check AppArmor status:

```bash
sudo aa-status
```

Test the custom profile:

```bash
sudo docker run --rm \
  --security-opt="apparmor=docker-apparmor-demo" \
  apparmor-flask-demo
```

## Troubleshooting and Findings

The Flask application ran successfully using Docker's built-in `docker-default` AppArmor profile.

The custom `docker-apparmor-demo` profile caused `file_mprotect` permission denials when Python and other executables attempted to load shared libraries. Basic executables also failed under the custom profile.

Kernel audit logs were used to identify the denials:

```bash
sudo journalctl -k --no-pager | grep 'profile="docker-apparmor-demo"'
```

**Current status:** The Docker application and built-in AppArmor profile were tested successfully. The custom profile still requires troubleshooting and validation before it can be considered a working security profile.

## Key Learnings

- Docker containers can use AppArmor profiles to restrict application behavior.
- A custom security profile must allow the operations required by the application.
- Kernel audit logs help diagnose AppArmor permission denials.
- A running container does not automatically mean that a custom security policy is correctly configured.

## Future Improvements

- Resolve the custom profile's memory-protection denials.
- Validate that the custom profile blocks the intended restricted operations.
- Reduce permissions to follow the principle of least privilege.
- Add screenshots of successful tests and audit-log findings.



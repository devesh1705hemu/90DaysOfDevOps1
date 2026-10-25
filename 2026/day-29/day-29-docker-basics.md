# Day 29: Introduction to Docker

## Goal

Understand Docker fundamentals, install Docker, run containers from Docker Hub, and practise essential container-management commands.

## 1. What Is Docker?

- **Docker:** A platform for building, shipping, and running applications in containers.
- **Container:** An isolated process with its application and dependencies that shares the host operating system's kernel.
- **Why containers?** They help applications run consistently across development, testing, and production environments.
- **Docker image:** A read-only template used to create containers.
- **Docker container:** A running or stopped instance created from an image.
- **Docker Hub:** A public registry where developers can find and download container images.

### Containers vs. Virtual Machines

| Containers | Virtual Machines |
|---|---|
| Share the host OS kernel. | Run a separate guest OS kernel. |
| Generally lightweight and quick to start. | Generally require more resources. |
| Isolate application processes. | Virtualize hardware and isolate a complete guest OS. |
| Commonly used for microservices and application deployment. | Commonly used when a separate OS environment is required. |

### Docker Architecture

Docker uses a client-server architecture.

- **Docker client:** The command-line interface that sends commands to the Docker daemon.
- **Docker daemon (`dockerd`):** The background service that manages containers, images, networks, and volumes.
- **Docker image:** The template used to create a container.
- **Docker container:** The isolated environment in which an application process runs.
- **Docker registry:** A service that stores and distributes images, such as Docker Hub.

**Architecture flow:**

Docker CLI → Docker daemon → Docker image from a registry, if needed → Container creation and execution → Output returned to the user.

## 2. Install and Verify Docker

Install Docker Desktop for Windows from the [official Docker website](https://www.docker.com/products/docker-desktop/). Start Docker Desktop and wait for the engine to become ready.

| Command | One-line explanation |
|---|---|
| `docker --version` | Displays the installed Docker CLI version. |
| `docker version` | Displays Docker client and server version information. |
| `docker info` | Displays information about the Docker engine and its configuration. |
| `docker run hello-world` | Downloads the image if needed, runs a test container, and prints a success message. |

**Important:** If Docker cannot connect to the daemon, ensure Docker Desktop is running before trying again.

### What Happens When You Run `hello-world`?

1. Docker checks whether the `hello-world` image is available locally.
2. If the image is missing, Docker downloads it from Docker Hub.
3. Docker creates and starts a container from the image.
4. The program prints a success message.
5. The program exits, so the container stops.

## 3. Run Real Containers

### Task 3.1: Run an Nginx Container

```powershell
docker run -d --name my-nginx -p 8080:80 nginx
```

**Explanation:** Starts Nginx in the background, names the container `my-nginx`, and maps host port `8080` to container port `80`.

Open this address in your browser:

http://localhost:8080

You should see the Nginx welcome page.

### Task 3.2: Run an Ubuntu Container

```powershell
docker run -it --name my-ubuntu ubuntu bash
```

**Explanation:** Starts an Ubuntu container and opens an interactive Bash shell.

Run these commands inside the Ubuntu container:

| Command | One-line explanation |
|---|---|
| `cat /etc/os-release` | Displays the Linux distribution and version information. |
| `whoami` | Prints the current username. |
| `pwd` | Displays the current working directory. |
| `ls -la` | Lists files, including hidden files, with details. |
| `apt update` | Refreshes the package index from configured repositories. |
| `apt install -y curl iputils-ping` | Installs curl and ping utilities without prompting for confirmation. |
| `ping -c 4 127.0.0.1` | Sends four test packets to the local loopback address. |
| `curl --version` | Displays the installed curl version. |
| `exit` | Exits the shell and stops the container if that shell is its main process. |

Note: The package installation commands require network access. Changes made inside a container are not automatically saved into the original image.

### Task 3.3: List Running Containers

```powershell
docker ps
```

**Explanation:** Lists containers that are currently running.

### Task 3.4: List All Containers

```powershell
docker ps -a
```

**Explanation:** Lists all containers, including stopped and exited containers.

### Task 3.5: List Local Images

```powershell
docker image ls
```

**Explanation:** Lists the container images stored on your local machine.

### Task 3.6: Stop and Remove a Container

```powershell
docker stop my-nginx
```

**Explanation:** Gracefully stops the running container named `my-nginx`.

```powershell
docker rm my-nginx
```

**Explanation:** Removes the stopped container named `my-nginx`.

```powershell
docker start my-nginx
```

**Explanation:** Starts an existing stopped container named `my-nginx`.

```powershell
docker restart my-nginx
```

**Explanation:** Restarts the container named `my-nginx`.

**Important:** Once a container is removed, you cannot restart that same container. You can create a new container from the image using `docker run`.

## 4. Explore Docker Features

### Task 4.1: Detached Mode

```powershell
docker run -d nginx
```

**Explanation:** Starts an Nginx container in the background and immediately returns control to the terminal.

- `-d` means detached mode.
- The container continues running independently of the terminal prompt.
- Use `docker ps` to check whether it is running.

### Task 4.2: Give a Container a Custom Name

```powershell
docker run -d --name devops-nginx nginx
```

**Explanation:** Creates and starts an Nginx container with the custom name `devops-nginx`.

Container names must be unique among existing containers, including stopped ones.

### Task 4.3: Map a Port

```powershell
docker run -d --name web-server -p 8080:80 nginx
```

**Explanation:** Maps host port `8080` to container port `80`, making the Nginx web server accessible from your computer.

Port mapping syntax:

`-p HOST_PORT:CONTAINER_PORT`

For example, `-p 8080:80` means traffic sent to port `8080` on your computer is forwarded to port `80` inside the container.

Open:

http://localhost:8080

If port `8080` is already in use, use another host port, such as `8081:80`.

### Task 4.4: Check Container Logs

```powershell
docker logs web-server
```

**Explanation:** Displays the output logs captured from the `web-server` container.

```powershell
docker logs -f web-server
```

**Explanation:** Continuously follows new log output until you press `Ctrl+C`.

```powershell
docker logs --tail 20 web-server
```

**Explanation:** Displays only the last 20 lines of container logs.

```powershell
docker logs -t web-server
```

**Explanation:** Displays container log entries with timestamps.

To generate Nginx access logs, open `http://localhost:8080` several times and then inspect the logs.

### Task 4.5: Execute a Command Inside a Running Container

```powershell
docker exec web-server nginx -v
```

**Explanation:** Executes the Nginx version command inside the running `web-server` container.

```powershell
docker exec web-server ps
```

**Explanation:** Runs `ps` inside the container to inspect processes if the utility is available.

```powershell
docker exec -it web-server sh
```

**Explanation:** Opens an interactive shell inside the running container.

Inside the container, try:

```sh
pwd
ls -la
cat /etc/os-release
exit
```

**Explanation:** These commands inspect the current directory, files, and OS information; `exit` closes the additional shell but normally leaves Nginx running.

Note: Minimal container images might not include utilities such as `ps` or `bash`. Use `sh` if available. The `docker exec` command works only when the target container is running.

### Task 4.6: Inspect Port Mapping

```powershell
docker port web-server
```

**Explanation:** Displays the published host-to-container port mappings for `web-server`.

## 5. Quick Docker Command Reference

| Command | One-line explanation |
|---|---|
| `docker --version` | Displays the Docker CLI version. |
| `docker version` | Displays client and server version details. |
| `docker info` | Shows Docker engine information. |
| `docker run IMAGE` | Creates and starts a container from an image. |
| `docker run -it IMAGE sh` | Starts a container with an interactive terminal and shell. |
| `docker run -d IMAGE` | Starts a container in the background. |
| `docker run --name NAME IMAGE` | Creates a container with a custom name. |
| `docker run -p HOST:CONTAINER IMAGE` | Maps a host port to a container port. |
| `docker ps` | Lists running containers. |
| `docker ps -a` | Lists all containers, including stopped containers. |
| `docker image ls` | Lists local images. |
| `docker pull IMAGE` | Downloads an image without starting a container. |
| `docker logs CONTAINER` | Displays captured container output. |
| `docker exec -it CONTAINER sh` | Opens an interactive shell in a running container. |
| `docker stop CONTAINER` | Stops a running container. |
| `docker start CONTAINER` | Starts an existing stopped container. |
| `docker restart CONTAINER` | Restarts an existing container. |
| `docker rm CONTAINER` | Removes a stopped container. |
| `docker inspect CONTAINER` | Displays detailed configuration and state information. |
| `docker port CONTAINER` | Displays published port mappings. |

Replace placeholders such as `IMAGE`, `CONTAINER`, `NAME`, `HOST`, and `CONTAINER_PORT` with actual values. Do not type angle-bracket placeholders literally into PowerShell.

## 6. Complete Practical Workflow

Run these commands one at a time and verify each result.

### Verify Docker and Run Hello World

```powershell
docker --version
docker run hello-world
```

### Start Nginx

```powershell
docker run -d --name web-server -p 8080:80 nginx
```

Open `http://localhost:8080` in your browser.

### Inspect Nginx

```powershell
docker ps
docker logs web-server
docker exec web-server nginx -v
docker port web-server
```

### Explore Ubuntu

```powershell
docker run -it --name my-ubuntu ubuntu bash
```

Inside Ubuntu:

```bash
cat /etc/os-release
whoami
pwd
ls -la
exit
```

### Inspect and Clean Up

Back in PowerShell:

```powershell
docker ps
docker ps -a
docker stop web-server
docker rm web-server
docker ps -a
```

If a container with one of these names already exists, inspect it with `docker ps -a` before reusing the name.


## 8. Why Docker Matters for DevOps

Docker packages applications and their dependencies into portable images, helping teams run applications consistently across development, testing, and production.

Containers are widely used in:

- CI/CD pipelines
- Microservices architectures
- Cloud deployments
- Application testing
- Container orchestration platforms such as Kubernetes

Learning Docker is an important step toward becoming a DevOps engineer because it builds the foundation for containerization, automated deployment, and orchestration.

## 9. Interview Practice

Try answering these questions without looking at the explanations.

1. What is Docker, and why do we use containers?
2. What is the difference between a Docker image and a container?
3. How are containers different from virtual machines?
4. What is the role of the Docker daemon?
5. What happens when you execute `docker run hello-world`?
6. What is the difference between `docker ps` and `docker ps -a`?
7. What does the `-d` flag do?
8. Explain `-p 8080:80`.
9. What is the difference between `docker stop` and `docker rm`?
10. What is the purpose of `docker logs`?
11. What is the difference between `docker run` and `docker exec`?
12. Why does an Nginx container continue running after you exit a shell opened with `docker exec`?




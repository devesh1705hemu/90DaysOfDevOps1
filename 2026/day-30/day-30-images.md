# Day 30 – Docker Images & Container Lifecycle

## Objective

Today's goal is to understand how Docker images and containers work, how image layers and caching improve efficiency, and how to manage the complete container lifecycle.

## Task 1: Docker Images

### 1. Pull Docker Images

```bash
docker pull nginx
docker pull ubuntu
docker pull alpine
```

- `docker pull nginx` – Downloads the NGINX web server image from Docker Hub.
- `docker pull ubuntu` – Downloads the Ubuntu Linux image from Docker Hub.
- `docker pull alpine` – Downloads the lightweight Alpine Linux image from Docker Hub.

### 2. List All Images

```bash
docker images
docker image ls
```

- `docker images` – Lists all Docker images available on the local machine, including their tags, IDs, and sizes.
- `docker image ls` – Displays the same image listing using the newer Docker command syntax.

**Observation:** Record the actual image sizes displayed by your system.

| Image | Purpose |
|---|---|
| NGINX | Web server and reverse proxy |
| Ubuntu | General-purpose Linux environment |
| Alpine | Lightweight Linux environment |

### 3. Compare Ubuntu and Alpine

```bash
docker image inspect ubuntu
docker image inspect alpine
```

| Image  | General Characteristics| 
| ------ | ------------------------------------------ | 
| Ubuntu | Full-featured Linux distribution           | 
| Alpine | Minimal and lightweight Linux distribution |

- `docker image inspect ubuntu` – Displays metadata and configuration details of the Ubuntu image.
- `docker image inspect alpine` – Displays metadata and configuration details of the Alpine image.

**Why is Alpine smaller than Ubuntu?**

- Alpine includes fewer preinstalled packages and utilities.
- Alpine uses BusyBox for many common system utilities.
- Alpine uses musl libc, while Ubuntu commonly uses glibc.
- Alpine is designed to provide a minimal operating system footprint.

**Real-world application:** Smaller images can reduce download time, storage requirements, and deployment overhead. However, application compatibility must be tested before choosing Alpine.

### 4. Inspect an Image

```bash
docker image inspect nginx
docker image inspect nginx --format '{{.Id}}'
docker image inspect nginx --format '{{.Os}}'
docker image inspect nginx --format '{{.Architecture}}'
docker image inspect nginx --format '{{json .Config.Cmd}}'
```

- `docker image inspect nginx` – Displays detailed metadata, configuration, tags, and layer-related information for the NGINX image.
- `docker image inspect nginx --format '{{.Id}}'` – Displays the full image ID.
- `docker image inspect nginx --format '{{.Os}}'` – Displays the operating system associated with the image.
- `docker image inspect nginx --format '{{.Architecture}}'` – Displays the CPU architecture of the image.
- `docker image inspect nginx --format '{{json .Config.Cmd}}'` – Displays the default command configured for the image.

### 5. Remove an Image

```bash
docker images
docker rmi ubuntu
docker images
```

- `docker images` – Lists images before removal.
- `docker rmi ubuntu` – Removes the Ubuntu image if it is not blocked by existing container references.
- `docker images` – Verifies whether the image was removed successfully.

**Note:** Docker may prevent removal if a container still references the image. Remove only resources you no longer need.

---

## Task 2: Image Layers

### 1. View Image History

```bash
docker image history nginx
docker image history --no-trunc nginx
```

- `docker image history nginx` – Displays the image history, including build instructions and the sizes associated with image changes.
- `docker image history --no-trunc nginx` – Displays the full history without truncating long commands.

### 2. Understand Layer Sizes

Some history entries show sizes, while others show `0B`.

- Layers with non-zero sizes commonly contain filesystem changes, such as installed packages or copied files.
- Entries showing `0B` often represent configuration instructions that do not add filesystem data.
- The displayed history is not necessarily a one-to-one list of all unique physical storage layers.

### 3. What Are Docker Image Layers?

Docker image layers are filesystem changes stacked together to form a complete image. Docker uses them to improve storage efficiency, image distribution, and build performance.

**Benefits of image layers:**

1. **Reusability:** Multiple images can share common layers.
2. **Storage efficiency:** Shared layers do not need to be stored repeatedly on the same Docker host.
3. **Build caching:** Docker can reuse unchanged build steps when their inputs remain valid.
4. **Faster downloads:** Docker generally downloads only the layers that are missing locally.
5. **Separation:** Filesystem changes from different image-building steps can be represented independently.

**Real-world example:** Multiple application images can reuse the same base image layers, reducing duplicate storage and download requirements.

---

## Task 3: Container Lifecycle

### 1. Create a Container Without Starting It

```bash
docker create --name lifecycle-demo nginx
docker ps -a
```

- `docker create --name lifecycle-demo nginx` – Creates a container named `lifecycle-demo` without starting its process.
- `docker ps -a` – Lists all containers, including created, running, and stopped containers.

**Expected state:** Created.

### 2. Start the Container

```bash
docker start lifecycle-demo
docker ps -a
```

- `docker start lifecycle-demo` – Starts the previously created container.
- `docker ps -a` – Displays the container's current state.

**Expected state:** Up.

### 3. Pause the Container

```bash
docker pause lifecycle-demo
docker ps -a
```

- `docker pause lifecycle-demo` – Suspends the container's running processes.
- `docker ps -a` – Shows the paused container status.

**Expected state:** Up (Paused).

### 4. Unpause the Container

```bash
docker unpause lifecycle-demo
docker ps -a
```

- `docker unpause lifecycle-demo` – Resumes the suspended processes.
- `docker ps -a` – Confirms that the container is running again.

**Expected state:** Up.

### 5. Stop the Container

```bash
docker stop lifecycle-demo
docker ps -a
```

- `docker stop lifecycle-demo` – Gracefully requests container termination before forcefully terminating it if necessary.
- `docker ps -a` – Shows the container in an exited state.

**Expected state:** Exited.

### 6. Restart the Container

```bash
docker restart lifecycle-demo
docker ps -a
```

- `docker restart lifecycle-demo` – Restarts the container by stopping and starting it.
- `docker ps -a` – Verifies the current container state.

**Expected state:** Up.

### 7. Kill the Container

```bash
docker kill lifecycle-demo
docker ps -a
```

- `docker kill lifecycle-demo` – Immediately sends `SIGKILL` by default to terminate the container's main process.
- `docker ps -a` – Shows the terminated container as exited.

**Expected state:** Exited, commonly with exit code 137.

### 8. Remove the Container

```bash
docker rm lifecycle-demo
docker ps -a
```

- `docker rm lifecycle-demo` – Removes the stopped container.
- `docker ps -a` – Confirms that the container no longer appears in the list.

**Expected state:** Removed.

### Container Lifecycle Summary

| Command | Purpose |
|---|---|
| `docker create` | Creates a container without starting it |
| `docker start` | Starts a stopped container |
| `docker pause` | Suspends container processes |
| `docker unpause` | Resumes suspended processes |
| `docker stop` | Gracefully stops a container |
| `docker restart` | Stops and starts a container |
| `docker kill` | Immediately terminates the main process by default |
| `docker rm` | Removes a stopped container |
| `docker ps` | Lists running containers |
| `docker ps -a` | Lists all containers |

---

## Task 4: Working with Running Containers

### 1. Run NGINX in Detached Mode

```bash
docker run -d --name nginx-lab -p 8080:80 nginx
docker ps
```

- `docker run -d --name nginx-lab -p 8080:80 nginx` – Creates and starts an NGINX container in the background, mapping host port 8080 to container port 80.
- `docker ps` – Lists currently running containers.

Test the web server:

```bash
curl http://localhost:8080
```

- `curl http://localhost:8080` – Sends an HTTP request to the NGINX service through the host's port mapping.

### 2. View Container Logs

```bash
docker logs nginx-lab
```

- `docker logs nginx-lab` – Displays the logs collected from the NGINX container.

Generate a request first if the access log is empty:

```bash
curl http://localhost:8080
docker logs nginx-lab
```

### 3. View Real-Time Logs

```bash
docker logs -f nginx-lab
```

- `docker logs -f nginx-lab` – Continuously follows new log entries generated by the container.

To display the latest 20 lines and continue following:

```bash
docker logs --tail 20 -f nginx-lab
```

- `docker logs --tail 20 -f nginx-lab` – Displays the last 20 log lines and follows new entries.

Press `Ctrl+C` to stop following logs without stopping the container.

### 4. Enter the Container

```bash
docker exec -it nginx-lab /bin/bash
```

- `docker exec -it nginx-lab /bin/bash` – Opens an interactive Bash shell inside the running container.

If Bash is unavailable, use:

```bash
docker exec -it nginx-lab /bin/sh
```

- `docker exec -it nginx-lab /bin/sh` – Opens an interactive shell using `/bin/sh`.

Explore the filesystem from inside the container:

```bash
pwd
ls -la /
ls -la /etc/nginx
ls -la /usr/share/nginx/html
cat /etc/os-release
exit
```

- `pwd` – Displays the current working directory.
- `ls -la /` – Lists files and directories in the root filesystem.
- `ls -la /etc/nginx` – Lists NGINX configuration files.
- `ls -la /usr/share/nginx/html` – Lists the default website files.
- `cat /etc/os-release` – Displays operating system identification information.
- `exit` – Leaves the interactive container shell.

### 5. Execute a Single Command Inside the Container

```bash
docker exec nginx-lab nginx -v
docker exec nginx-lab ls -la /usr/share/nginx/html
docker exec nginx-lab cat /etc/nginx/nginx.conf
```

- `docker exec nginx-lab nginx -v` – Displays the installed NGINX version.
- `docker exec nginx-lab ls -la /usr/share/nginx/html` – Lists the website files without opening an interactive shell.
- `docker exec nginx-lab cat /etc/nginx/nginx.conf` – Displays the main NGINX configuration file.

### 6. Inspect the Container

```bash
docker inspect nginx-lab
```

- `docker inspect nginx-lab` – Displays detailed container metadata, networking information, configuration, and mount details.

**Find the container IP address:**

```bash
docker inspect nginx-lab --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
```

- This command displays the container's IP address on its configured Docker network.

**Find port mappings:**

```bash
docker port nginx-lab
```

- `docker port nginx-lab` – Displays the published host ports mapped to container ports.

Expected mapping example:

```text
80/tcp -> 0.0.0.0:8080
```

**Find mounts:**

```bash
docker inspect nginx-lab --format '{{json .Mounts}}'
```

- This command displays the container's configured mounts, including source paths, destinations, and mount types.

For a readable mount listing:

```bash
docker inspect nginx-lab --format '{{range .Mounts}}{{println .Type .Source "->" .Destination}}{{end}}'
```

- This command prints each mount type, source, and destination in a readable format.

---

## Task 5: Docker Cleanup

**Warning:** These commands can stop applications and remove containers or images. Check what is running before using them, especially on shared or production machines.

### 1. Stop All Running Containers

```bash
docker stop $(docker ps -q)
```

- `docker stop $(docker ps -q)` – Stops all currently running containers.

Safer version when no containers are running:

```bash
docker ps -q | xargs -r docker stop
```

- This command stops running containers and handles an empty container list without passing an empty argument to `docker stop`.

Verify:

```bash
docker ps
```

- `docker ps` – Confirms which containers remain running.

### 2. Remove All Containers

```bash
docker rm $(docker ps -aq)
```

- `docker rm $(docker ps -aq)` – Removes all stopped containers after they have been stopped.

Safer version:

```bash
docker ps -aq | xargs -r docker rm
```

- This removes all containers returned by `docker ps -aq` and safely handles an empty list.

**Important:** This removes every container on the Docker host, not just containers created during today's practice.

### 3. Remove Unused Images

```bash
docker image prune
```

- `docker image prune` – Removes dangling images that are not referenced by containers.

To remove all images not referenced by any existing container:

```bash
docker image prune -a
```

- `docker image prune -a` – Removes all unused images, including tagged images that are not referenced by containers.

**Note:** Run the more aggressive cleanup only when you are certain you no longer need those images.

### 4. Check Docker Disk Usage

```bash
docker system df
docker system df -v
```

- `docker system df` – Displays disk usage for Docker images, containers, local volumes, and build cache.
- `docker system df -v` – Displays detailed Docker disk usage information.

Example output:

```text
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          3         0         250MB     250MB
Containers      0         0         0B        0B
Local Volumes   1         0         20MB      20MB
Build Cache     2         0         50MB      50MB
```

The figures above are illustrative. Record the actual output from your machine.

---

## Key Learnings

1. Docker images are templates used to create containers.
2. Docker image layers improve reuse, storage efficiency, and build caching.
3. Containers can be created, started, paused, resumed, stopped, restarted, killed, and removed.
4. `docker logs` helps troubleshoot application behavior.
5. `docker exec` allows commands to run inside a running container.
6. `docker inspect` provides detailed information about container configuration, networking, and mounts.
7. Docker cleanup commands help manage unused resources and disk space.

## Real-World DevOps Applications

- **Image management:** Pull and inspect images before deploying applications.
- **Container troubleshooting:** Check logs, execute diagnostic commands, and inspect network settings.
- **Lifecycle management:** Restart failed services and stop containers safely during maintenance.
- **Storage optimization:** Remove unused images and monitor Docker disk consumption.
- **Production readiness:** Use health checks, persistent volumes, proper logging, and resource limits when deploying applications.


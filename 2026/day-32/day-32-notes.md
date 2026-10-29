# Day 32: Docker Volumes & Networking

> **#90DaysOfDevOps | Hands-on Practice | Docker Fundamentals**

## 1. Overview

### Objective

The objective of Day 32 is to understand and implement two essential Docker concepts:

1. **Data Persistence:** Store application data so it survives container removal.
2. **Container Networking:** Enable containers to communicate with one another securely and reliably.

### What I Learned

- Why data can be lost when a container is removed.
- How to create and manage Docker named volumes.
- How to use bind mounts to share files between the host and a container.
- How Docker bridge networking works.
- The difference between the default bridge and custom bridge networks.
- How to connect an application container to a database container using container names.

### Prerequisites

- Docker Desktop installed and running.
- Basic knowledge of Docker images and containers.
- A terminal such as PowerShell, Command Prompt, or Bash.
- A web browser for testing the Nginx web server.

**Platform note:** The commands below work in Windows PowerShell, Linux, and macOS unless otherwise mentioned. Use `docker exec` to execute commands inside containers.

---

## 2. Task 1: Understand the Problem of Data Loss

### Objective

Demonstrate what happens to database data when a container is removed without persistent storage.

### Step 1: Run a PostgreSQL Container

```bash
docker run -d --name postgres-temp -e POSTGRES_PASSWORD=devpass123 postgres:16
```

**Command explanation:**

- `docker run`: Creates and starts a container.
- `-d`: Runs the container in the background.
- `--name postgres-temp`: Assigns a name to the container.
- `-e POSTGRES_PASSWORD=devpass123`: Sets the database administrator password.
- `postgres:16`: Uses the PostgreSQL 16 image.

### Step 2: Create a Table and Insert Data

```bash
docker exec -it postgres-temp psql -U postgres
```

Execute these SQL statements inside PostgreSQL:

```sql
CREATE DATABASE devopsdb;
\c devopsdb

CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

INSERT INTO students (name)
VALUES ('Devesh'), ('Docker Learner');

SELECT * FROM students;

\q
```

**Expected result:**

| id | name |
|---:|---|
| 1 | Devesh |
| 2 | Docker Learner |

### Step 3: Stop and Remove the Container

```bash
docker stop postgres-temp
docker rm postgres-temp
```

These commands stop the container and remove the container itself.

### Step 4: Start a New Container

```bash
docker run -d --name postgres-temp -e POSTGRES_PASSWORD=devpass123 postgres:16
```

Connect to PostgreSQL:

```bash
docker exec -it postgres-temp psql -U postgres -d devopsdb
```

Run:

```sql
SELECT * FROM students;
```

**Expected result:** The connection fails because the `devopsdb` database does not exist in the newly initialized container.

### Conclusion

Container removal deletes the container's writable layer. Without a persistent volume, the database's stored data is not preserved for the replacement container.

Docker images are templates, containers are running instances, and persistent storage must be managed separately.

> Note: Data can remain in Docker's internal storage after a container stops. The important distinction is that deleting the container does not preserve its writable layer as reusable storage for a new container.

---

## 3. Task 2: Docker Named Volumes

### Objective

Use a named volume to persist PostgreSQL data across container removal and recreation.

### Step 1: Create a Named Volume

```bash
docker volume create postgres-data
```

List all volumes:

```bash
docker volume ls
```

Inspect the volume:

```bash
docker volume inspect postgres-data
```

**Explanation:**

- `docker volume create`: Creates persistent storage managed by Docker.
- `docker volume ls`: Lists Docker volumes.
- `docker volume inspect`: Displays information about a particular volume.

### Step 2: Run PostgreSQL With the Volume

```bash
docker run -d \
  --name postgres-persistent \
  -e POSTGRES_PASSWORD=devpass123 \
  -e POSTGRES_DB=devopsdb \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:16
```

In PowerShell, use a single line if multiline syntax causes errors:

```powershell
docker run -d --name postgres-persistent -e POSTGRES_PASSWORD=devpass123 -e POSTGRES_DB=devopsdb -v postgres-data:/var/lib/postgresql/data postgres:16
```

**Important options:**

- `-v postgres-data:/var/lib/postgresql/data`: Mounts the named volume at PostgreSQL's data directory.
- `POSTGRES_DB=devopsdb`: Creates the initial database on first initialization.
- The volume retains the database files when the container is removed.

### Step 3: Insert Data

```bash
docker exec -it postgres-persistent psql -U postgres -d devopsdb
```

Execute:

```sql
CREATE TABLE persistence_test (
    id SERIAL PRIMARY KEY,
    message VARCHAR(200)
);

INSERT INTO persistence_test (message)
VALUES ('Data survives container removal');

SELECT * FROM persistence_test;

\q
```

### Step 4: Remove the Container

```bash
docker stop postgres-persistent
docker rm postgres-persistent
```

Do not remove the volume.

### Step 5: Create a New Container Using the Same Volume

```bash
docker run -d --name postgres-restored -e POSTGRES_PASSWORD=devpass123 -e POSTGRES_DB=devopsdb -v postgres-data:/var/lib/postgresql/data postgres:16
```

Wait for PostgreSQL to become ready:

```bash
docker exec postgres-restored pg_isready -U postgres
```

Verify the saved data:

```bash
docker exec -it postgres-restored psql -U postgres -d devopsdb -c "SELECT * FROM persistence_test;"
```

**Expected result:** The original row is still present.

### Step 6: Verify the Volume

```bash
docker volume ls
docker volume inspect postgres-data
docker inspect postgres-restored
```

Inspecting the container shows the volume mounted at `/var/lib/postgresql/data`.

### Conclusion

Named volumes provide persistent storage independently of a container's lifecycle. A replacement container can access the same database files by mounting the same volume.

**Important:** Do not run `docker volume rm postgres-data` or `docker volume prune` during this experiment. Removing the volume can destroy the only copy of your database data.

---

## 4. Task 3: Docker Bind Mounts

### Objective

Serve a host machine's HTML file through an Nginx container and observe live file changes.

### Step 1: Create a Project Folder

In PowerShell:

```powershell
mkdir nginx-site
cd nginx-site
```

Create a file named `index.html` in this folder with the following content:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Day 32 - Docker Networking</title>
</head>
<body>
    <h1>Hello from Docker!</h1>
    <p>This page is served by Nginx using a bind mount.</p>
</body>
</html>
```

### Step 2: Run Nginx With a Bind Mount

From the parent directory containing `nginx-site`, run:

```powershell
docker run -d --name nginx-bind -p 8080:80 --mount "type=bind,source=$($PWD.Path)\nginx-site,target=/usr/share/nginx/html,readonly" nginx:alpine
```

If you are already inside `nginx-site`, return to the parent directory first.

**Explanation:**

- `-p 8080:80`: Maps host port 8080 to container port 80.
- `--mount type=bind`: Mounts a specific host directory.
- `source`: The directory on your computer.
- `target`: The directory served by Nginx.
- `readonly`: Prevents the container from modifying the mounted files.

### Step 3: Access the Website

Open this address in your browser:

http://localhost:8080

**Expected result:** The browser displays the heading "Hello from Docker!"

### Step 4: Edit the Host File

Modify `index.html` on your computer:

```html
<h1>Updated without rebuilding the Docker image!</h1>
<p>This change comes directly from the host filesystem.</p>
```

Save the file and refresh the browser.

**Expected result:** Nginx serves the updated HTML without restarting the container or rebuilding its image.

### Step 5: Verify the Mount

```powershell
docker inspect nginx-bind
```

Check the `Mounts` section for the source directory, destination, and read-only setting.

### Named Volume vs Bind Mount

| Feature | Named Volume | Bind Mount |
|---|---|---|
| Storage location | Managed by Docker | A specified host path |
| Setup | `docker volume create` | Specify a host directory |
| Main use | Databases and persistent application data | Source code, configuration, and local files |
| Host path control | Docker manages the storage location | User chooses the path |
| Portability | Easier to manage through Docker | Depends on host directory structure |

### Conclusion

A named volume is ideal for persistent application data. A bind mount is useful when a container needs direct access to files on the host machine.

---

## 5. Task 4: Docker Networking Basics

### Objective

Explore Docker networks and test communication between containers on the default bridge network.

### Step 1: List Docker Networks

```bash
docker network ls
```

Typical network types include:

- `bridge`: The default network driver for standalone containers.
- `host`: Shares the host network namespace on supported Linux environments.
- `none`: Disables normal external networking for the container.

Docker Desktop may expose different host networking behavior depending on the operating system and configuration.

### Step 2: Inspect the Default Bridge

```bash
docker network inspect bridge
```

This displays connected containers, subnet information, gateway details, and network configuration.

### Step 3: Run Two Containers

```bash
docker run -dit --name bridge-a ubuntu:24.04 bash
docker run -dit --name bridge-b ubuntu:24.04 bash
```

Install ping in both containers:

```bash
docker exec bridge-a bash -c "apt-get update && apt-get install -y iputils-ping"
docker exec bridge-b bash -c "apt-get update && apt-get install -y iputils-ping"
```

### Step 4: Test Communication by Container Name

```bash
docker exec bridge-a ping -c 3 bridge-b
```

**Expected result:** Name resolution generally fails on the default bridge because it does not provide the same automatic container-name DNS resolution as a user-defined bridge network.

### Step 5: Test Communication by IP Address

Get the IP address of `bridge-b`:

```bash
docker inspect -f "{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}" bridge-b
```

Copy the returned IP address and run:

```bash
docker exec bridge-a ping -c 3 <BRIDGE_B_IP>
```

Replace `<BRIDGE_B_IP>` with the actual address, for example `172.17.0.3` if that is what Docker returns.

**Expected result:** The containers can generally communicate by IP address when they are on the same default bridge and connectivity is permitted.

### Conclusion

The default bridge supports IP-based connectivity between attached containers, but it does not provide the automatic container-name DNS resolution available on user-defined bridge networks.

---

## 6. Task 5: Create a Custom Bridge Network

### Objective

Use a custom bridge network to enable communication by container name.

### Step 1: Create a Network

```bash
docker network create my-app-net
```

Verify:

```bash
docker network ls
docker network inspect my-app-net
```

### Step 2: Run Two Containers on the Custom Network

```bash
docker run -dit --name app-a --network my-app-net ubuntu:24.04 bash
docker run -dit --name app-b --network my-app-net ubuntu:24.04 bash
```

Install ping:

```bash
docker exec app-a bash -c "apt-get update && apt-get install -y iputils-ping"
```

### Step 3: Ping the Second Container by Name

```bash
docker exec app-a ping -c 3 app-b
```

**Expected result:** The hostname `app-b` resolves to its container IP address, and the ping succeeds if the environment permits ICMP traffic.

### Step 4: Inspect Network Membership

```bash
docker network inspect my-app-net
```

Check the `Containers` section to confirm both containers are attached.

### Why Does the Custom Network Work?

A user-defined bridge network includes Docker's embedded DNS service, which resolves container names and network aliases to container IP addresses.

The default bridge does not provide the same automatic name resolution for standalone containers. A custom network also makes it easier to isolate groups of services.

### Conclusion

Custom bridge networks simplify application architecture because services can communicate using stable names instead of hard-coded IP addresses.

---

## 7. Task 6: Combine Networking and Persistent Storage

### Objective

Build a small backend infrastructure in which a PostgreSQL database has persistent storage and an application container can reach it by name.

### Architecture

```text
                    Docker Network
                      my-app-net
                          |
             +------------+------------+
             |                         |
      +------v-------+         +-------v------+
      | PostgreSQL   |         | App / Client |
      | Container    |<--------| Container    |
      +------+-------+         +--------------+
             |
      +------v-------+
      | Named Volume |
      | postgres-data|
      +--------------+
```

### Step 1: Create the Network and Volume

```bash
docker network create my-app-net
docker volume create postgres-data
```

If these resources already exist from earlier tasks, reuse them instead of creating duplicates.

### Step 2: Start PostgreSQL

```bash
docker run -d --name app-db --network my-app-net -e POSTGRES_USER=appuser -e POSTGRES_PASSWORD=devpass123 -e POSTGRES_DB=appdb -v postgres-data:/var/lib/postgresql/data postgres:16
```

Wait for the database to become ready:

```bash
docker exec app-db pg_isready -U appuser -d appdb
```

**Expected result:** PostgreSQL reports that it is accepting connections.

> Important: If `postgres-data` already contains a database initialized with different credentials or settings, PostgreSQL will reuse that existing database. Initialization environment variables do not reset an existing database.

### Step 3: Run an Application-Side Client Container

Use the PostgreSQL client tools in a temporary Ubuntu container attached to the same network:

```bash
docker run --rm -it --network my-app-net ubuntu:24.04 bash
```

Inside the Ubuntu container, install the client:

```bash
apt-get update && apt-get install -y postgresql-client
```

Connect using the database container's name:

```bash
psql -h app-db -p 5432 -U appuser -d appdb
```

Enter the password:

```text
devpass123
```

The `-h app-db` option uses Docker's network DNS to resolve the database container name.

### Step 4: Verify Database Communication

Run these SQL commands from the client:

```sql
SELECT current_database(), current_user;

CREATE TABLE IF NOT EXISTS app_health (
    id SERIAL PRIMARY KEY,
    status VARCHAR(50)
);

INSERT INTO app_health (status)
VALUES ('Database connection successful');

SELECT * FROM app_health;

\q
```

Exit the Ubuntu container:

```bash
exit
```

### Step 5: Verify the Network

```bash
docker network inspect my-app-net
docker inspect app-db
docker volume inspect postgres-data
```

Confirm that the database is attached to `my-app-net` and that the named volume is mounted.

### Conclusion

The database container stores its files in a named volume. The client container reaches PostgreSQL through the custom network using `app-db` as the hostname.

**Production note:** This is a client-container demonstration, not a complete application deployment. In a real system, the application would use a database driver and retrieve credentials from a secure secret-management system rather than hard-coding passwords.

---

## 8. Troubleshooting Guide

| Problem | Possible cause | Solution |
|---|---|---|
| Container exits immediately | Startup command or configuration issue | Run `docker logs CONTAINER_NAME` |
| PostgreSQL is not ready | Database is still initializing | Run `docker exec app-db pg_isready -U appuser` |
| Database data disappears | No persistent volume was mounted | Attach a named volume to the database directory |
| Container names do not resolve | Containers are on different networks or using the default bridge | Attach both containers to the same custom network |
| Ping command not found | Ping package is not installed | Install `iputils-ping` in Ubuntu |
| Nginx page does not update | Incorrect mount path, file not saved, or browser cache | Verify the mount and refresh the page |
| Port 8080 is unavailable | Another process is using the port | Choose another host port, such as `-p 8081:80` |
| Container name already exists | An earlier container has the same name | Inspect and remove the old container if no longer needed |

Useful diagnostic commands:

```bash
docker ps -a
docker logs app-db
docker inspect app-db
docker network ls
docker volume ls
docker stats
```

---


---

## 10. Cleanup

After capturing the required screenshots, remove the experimental containers and resources that you no longer need.

### Remove Containers

```bash
docker rm -f postgres-temp postgres-persistent postgres-restored nginx-bind bridge-a bridge-b app-a app-b app-db
```

The command reports errors for containers that do not exist. This is harmless when cleaning up optional experiments.

### Remove the Custom Network

```bash
docker network rm my-app-net
```

The network must have no attached containers before Docker can remove it.

### Remove the Named Volume

```bash
docker volume rm postgres-data
```

**Warning:** This deletes the database files stored in that volume. Only run this command if you no longer need the data.

Remove the host folder separately if you want:

```powershell
Remove-Item -Recurse -Force .\nginx-site
```

Run this PowerShell command from the directory containing `nginx-site`, and only after you have finished using the files.

---

## 11. Real-World Applications

### Where These Concepts Are Used

1. **Databases in production:** Named volumes preserve database files across container recreation. Production systems also need backups, recovery testing, and suitable storage architecture.
2. **Local development:** Bind mounts let developers edit code on their host machines while applications run inside containers.
3. **Microservices:** Custom Docker networks allow services to communicate through service names instead of fixed IP addresses.
4. **Application deployment:** A backend service can connect to a database using its network hostname.
5. **Troubleshooting:** Network and volume inspection commands help identify connectivity and persistence issues.

### Interview Questions

**Q1. What is a Docker volume?**

A Docker-managed storage mechanism that allows data to persist independently of a container's lifecycle.

**Q2. What is the difference between a named volume and a bind mount?**

A named volume is managed by Docker, whereas a bind mount maps a specified host filesystem path into a container.

**Q3. What happens to a named volume when its container is removed?**

The volume normally remains available unless explicitly removed.

**Q4. Why use a custom bridge network?**

It provides automatic DNS-based container-name resolution and network isolation.

**Q5. Why is a container IP address less suitable than a container name?**

Container IP addresses can change when containers are recreated, while network names and aliases provide a more stable way to locate services.

**Q6. Does a Docker volume replace database backups?**

No. Volumes provide persistence, but backups are still required for recovery from corruption, accidental deletion, and other failures.

---

## 12. Final Summary

Today, I learned how Docker volumes and networking solve two fundamental infrastructure problems.

- **Volumes** preserve application data independently of container lifecycles.
- **Bind mounts** make host files accessible inside containers.
- **Default bridge networking** supports IP-based communication but lacks the same automatic container-name DNS resolution.
- **Custom bridge networks** enable name-based communication between connected containers.
- **Combined architecture** allows an application-side client to reach a persistent PostgreSQL database through a custom network.


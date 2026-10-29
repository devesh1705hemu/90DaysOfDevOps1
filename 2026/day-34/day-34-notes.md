# Day 34: Docker Compose Advanced, Real-World Multi-Container Apps

> **#90DaysOfDevOps | Hands-on Practice | Docker, Flask, PostgreSQL, Redis**

## 1. Objective

Today's goal is to build a production-like application stack using Docker Compose.

We will combine a web application, a relational database, and a cache, then configure health checks, service dependencies, restart policies, custom networks, named volumes, and service scaling.

### Learning outcomes

- Build a Flask application using a custom Dockerfile.
- Connect Flask to PostgreSQL and Redis.
- Configure `depends_on` with `condition: service_healthy`.
- Understand Docker restart policies.
- Use named networks, named volumes, and service labels.
- Rebuild an application image after a code change.
- Understand why scaling services with fixed host-port mappings causes conflicts.

## 2. Project Architecture

<box border radius="lg" padding={3} gap={3}>
  <box background="surface-secondary" radius="md" padding={3} align="center">
    <icon name="globe" size="xl"/>
    **Client / Browser**

    <text color="secondary" size="sm">http://localhost:5000</text>
  </box>
  <box align="center" gap={1}>
    <icon name="arrow-down" size="lg"/>
    <text color="secondary" size="xs">HTTP traffic</text>
  </box>
  <box border radius="md" padding={3} align="center" gap={1}>
    <icon name="layers" size="xl"/>
    **Flask Web Application**

    <text color="secondary" size="sm">Custom Dockerfile · Python · Flask</text>
  </box>
  <grid columns={2} gap={2}>
    <grid-item>
      <box align="center" gap={1}>
        <icon name="arrow-down" size="lg"/>
        <text color="secondary" size="xs">SQL queries</text>
      </box>
      <box background="surface" border radius="md" padding={3} gap={1} align="center">
        <icon name="database" size="xl"/>
        **PostgreSQL**
        <text color="secondary" size="xs">Persistent named volume</text>
      </box>
    </grid-item>
    <grid-item>
      <box align="center" gap={1}>
        <icon name="arrow-down" size="lg"/>
        <text color="secondary" size="xs">Cache operations</text>
      </box>
      <box background="surface" border radius="md" padding={3} gap={1} align="center">
        <icon name="zap" size="xl"/>
        **Redis**
        <text color="secondary" size="xs">In-memory cache</text>
      </box>
    </grid-item>
  </grid>
  <text color="secondary" size="sm" textAlign="center">All services communicate through an explicitly defined Docker bridge network.</text>
</box>

## 3. Task 1: Build Your Own Application Stack

### Step 1: Create the project directory

Run these commands in PowerShell:

```powershell
mkdir day-34-compose-advanced
cd day-34-compose-advanced

mkdir app
New-Item app/app.py -ItemType File
New-Item app/requirements.txt -ItemType File
New-Item app/Dockerfile -ItemType File
New-Item docker-compose.yml -ItemType File
New-Item .env -ItemType File
New-Item .dockerignore -ItemType File
New-Item .gitignore -ItemType File
```

**Explanation:** Creates the project folders and files required for the Flask application and Compose configuration.

### Step 2: Create the Flask application

Open `app/app.py` and add:

```python
import os
import time

import psycopg
import redis
from flask import Flask, jsonify

app = Flask(__name__)

DB_HOST = os.getenv("DB_HOST", "db")
DB_NAME = os.getenv("POSTGRES_DB", "devopsdb")
DB_USER = os.getenv("POSTGRES_USER", "devops")
DB_PASSWORD = os.getenv("POSTGRES_PASSWORD")

REDIS_HOST = os.getenv("REDIS_HOST", "cache")

cache = redis.Redis(
    host=REDIS_HOST,
    port=6379,
    decode_responses=True,
    socket_connect_timeout=2,
)

def get_database_connection():
    return psycopg.connect(
        host=DB_HOST,
        dbname=DB_NAME,
        user=DB_USER,
        password=DB_PASSWORD,
        connect_timeout=3,
    )

@app.route("/")
def home():
    try:
        with get_database_connection() as conn:
            with conn.cursor() as cur:
                cur.execute("""
                    CREATE TABLE IF NOT EXISTS visits (
                        id SERIAL PRIMARY KEY,
                        visited_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                    )
                """)
                cur.execute(
                    "INSERT INTO visits DEFAULT VALUES RETURNING id"
                )
                visit_id = cur.fetchone()[0]

                cur.execute("SELECT COUNT(*) FROM visits")
                total_visits = cur.fetchone()[0]

        cache_hits = cache.incr("page_views")

        return jsonify({
            "application": "Day 34 DevOps Stack",
            "status": "success",
            "database": "PostgreSQL connected",
            "database_visit_id": visit_id,
            "total_database_visits": total_visits,
            "cache": "Redis connected",
            "redis_page_views": cache_hits,
        })

    except Exception as exc:
        app.logger.exception("Application dependency check failed")
        return jsonify({
            "status": "error",
            "message": str(exc),
        }), 503

@app.route("/health")
def health():
    return jsonify({"status": "healthy"}), 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

**Explanation:** Flask exposes a web endpoint, writes visit records to PostgreSQL, and increments a Redis counter.

The `/health` endpoint confirms that the Flask HTTP server is responding. The `/` endpoint checks both dependencies by actually using them.

### Step 3: Create `app/requirements.txt`

```text
Flask==3.1.0
psycopg[binary]==3.2.4
redis==5.2.1
```

**Explanation:** Defines the Python packages needed by the application.

### Step 4: Create the Dockerfile

Open `app/Dockerfile`:

```dockerfile
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

RUN useradd --create-home appuser \
    && chown -R appuser:appuser /app

USER appuser

EXPOSE 5000

CMD ["python", "app.py"]
```

**Explanation:**

- `FROM`: Selects the base image.
- `ENV`: Configures Python behavior.
- `WORKDIR`: Sets the application directory.
- `COPY`: Copies dependencies and source code into the image.
- `RUN`: Installs packages and creates a non-root user.
- `USER`: Runs the application without root privileges.
- `EXPOSE`: Documents the application's listening port.
- `CMD`: Starts Flask.

### Step 5: Create `.env`

```dotenv
POSTGRES_DB=devopsdb
POSTGRES_USER=devops
POSTGRES_PASSWORD=ChangeThisToALocalDevPassword123
```

These credentials are for local practice only. Do not reuse them in production.

### Step 6: Create `.dockerignore`

```text
__pycache__/
*.pyc
.env
.git/
.venv/
venv/
```

**Explanation:** Prevents unnecessary files and local secrets from entering the Docker build context.

### Step 7: Create `.gitignore`

```text
.env
__pycache__/
*.pyc
.venv/
venv/
```

**Explanation:** Prevents local configuration and Python-generated files from being committed to Git.

### Step 8: Create `docker-compose.yml`

Open `docker-compose.yml` and add the complete configuration:

```yaml
services:
  web:
    build:
      context: ./app
      dockerfile: Dockerfile
    ports:
      - "5000:5000"
    environment:
      DB_HOST: db
      POSTGRES_DB: ${POSTGRES_DB:?Set POSTGRES_DB in .env}
      POSTGRES_USER: ${POSTGRES_USER:?Set POSTGRES_USER in .env}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?Set POSTGRES_PASSWORD in .env}
      REDIS_HOST: cache
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test:
        - CMD
        - python
        - -c
        - "import urllib.request; urllib.request.urlopen('http://127.0.0.1:5000/health', timeout=3)"
      interval: 15s
      timeout: 5s
      retries: 3
      start_period: 10s
    labels:
      com.example.project: day-34
      com.example.role: web

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: ${POSTGRES_DB:?Set POSTGRES_DB in .env}
      POSTGRES_USER: ${POSTGRES_USER:?Set POSTGRES_USER in .env}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?Set POSTGRES_PASSWORD in .env}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test:
        - CMD-SHELL
        - pg_isready -U "$$POSTGRES_USER" -d "$$POSTGRES_DB"
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    restart: always
    networks:
      - backend
    labels:
      com.example.project: day-34
      com.example.role: database

  cache:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
    restart: unless-stopped
    networks:
      - backend
    labels:
      com.example.project: day-34
      com.example.role: cache

volumes:
  postgres_data:

networks:
  backend:
    driver: bridge
```

### Understanding the configuration

| Configuration | Purpose |
|---|---|
| `build` | Builds the Flask image from your Dockerfile |
| `ports` | Maps host port 5000 to container port 5000 |
| `environment` | Supplies configuration to the application |
| `depends_on` | Defines service startup dependencies |
| `condition: service_healthy` | Waits for a dependency's health check to pass |
| `healthcheck` | Tests whether a service is responding |
| `restart` | Defines container restart behavior |
| `volumes` | Preserves PostgreSQL database files |
| `networks` | Defines the network used for service communication |
| `labels` | Adds searchable metadata to containers |

The `web` service joins the Compose-created default network and the explicitly defined `backend` network. PostgreSQL and Redis are attached to `backend`, allowing the web application to reach them by service name.

The database does not publish port 5432 to the host. Only the Flask application is exposed on port 5000.

### Step 9: Validate and start the stack

```bash
docker compose config
```

**Purpose:** Validates the configuration and resolves environment variables.

```bash
docker compose up -d --build
```

**Purpose:** Builds the Flask image and starts all three services in detached mode.

Check the status:

```bash
docker compose ps
```

View the logs:

```bash
docker compose logs
```

Test the application:

```bash
curl http://localhost:5000/
```

You can also open `http://localhost:5000/` in your browser.

**Expected result:** A JSON response showing successful PostgreSQL and Redis connections, along with visit and cache counters.

Example response:

```json
{
  "application": "Day 34 DevOps Stack",
  "status": "success",
  "database": "PostgreSQL connected",
  "database_visit_id": 1,
  "total_database_visits": 1,
  "cache": "Redis connected",
  "redis_page_views": 1
}
```

Values will change each time you access the endpoint.

---

## 4. Task 2: `depends_on` and Health Checks

### Objective

Ensure that the application starts only after PostgreSQL and Redis are ready to accept connections.

### Step 1: Understand the difference

A basic dependency looks like this:

```yaml
depends_on:
  - db
```

This establishes a startup order, but it does not guarantee that PostgreSQL is ready to accept queries.

The improved configuration uses:

```yaml
depends_on:
  db:
    condition: service_healthy
  cache:
    condition: service_healthy
```

**Explanation:** Compose waits for the health checks of both dependencies to pass before starting the web service.

### Step 2: Verify database health

```bash
docker compose ps
```

```bash
docker inspect $(docker compose ps -q db)
```

The second command uses shell substitution supported in Bash and many Linux terminals. In PowerShell, use:

```powershell
$dbId = docker compose ps -q db
docker inspect $dbId
```

Look for the container's health status.

You can also check PostgreSQL directly:

```bash
docker compose exec db pg_isready
```

### Step 3: Test a fresh startup

```bash
docker compose down
docker compose up -d --build
docker compose ps
docker compose logs web
```

**Expected result:** PostgreSQL and Redis become healthy before Compose starts the web application.

### Important distinction

Health checks establish whether a service meets its defined health criteria. They do not guarantee that a service will never fail later.

The Flask health endpoint in this project checks the HTTP server only. For a production deployment, you may want separate liveness and readiness checks, with readiness also verifying required dependencies.

**Conclusion:** `depends_on` with `service_healthy` is more reliable than startup ordering alone.

---

## 5. Task 3: Restart Policies

### Objective

Understand how Docker handles containers that exit unexpectedly.

### Step 1: Understand the policies

| Policy | Behavior | Typical use |
|---|---|---|
| `no` | Does not automatically restart the container | One-off tasks and debugging |
| `always` | Restarts after exits, including after a Docker daemon restart, subject to Docker's manual-stop behavior | Services expected to remain running |
| `unless-stopped` | Restarts unless explicitly stopped by the operator | Long-running services where intentional stops should be respected |
| `on-failure` | Restarts after a non-zero exit status | Workloads where failures should trigger a retry |

### Step 2: Test `restart: always`

The database service already contains:

```yaml
restart: always
```

Start the stack:

```bash
docker compose up -d
```

Manually kill the database container:

```bash
docker compose kill db
```

Check its state:

```bash
docker compose ps -a
```

Check its logs:

```bash
docker compose logs db
```

Docker should restart the database container according to its restart policy. It may take a short time to become healthy again.

Verify:

```bash
docker compose exec db pg_isready
```

### Step 3: Test `restart: on-failure`

Change the database service's restart policy in `docker-compose.yml`:

```yaml
restart: on-failure:3
```

This allows up to three restart attempts after qualifying failures.

Apply the configuration:

```bash
docker compose up -d
```

For a clean experiment, recreate the database container if needed:

```bash
docker compose up -d --force-recreate db
```

Then test:

```bash
docker compose kill db
docker compose ps -a
```

A forced kill generally produces a non-zero exit status, so the policy should trigger a restart.

**Important:** `docker compose stop db` is not a valid test for automatic restart behavior. Docker suppresses automatic restarts when a container is intentionally stopped by the operator.

### Step 4: Restore the normal configuration

Set the database service back to:

```yaml
restart: always
```

Apply it:

```bash
docker compose up -d
```

### Conclusion

Restart policies help recover from process failures, but they do not fix application bugs or guarantee application availability. Production systems also require monitoring, alerting, backups, and appropriate recovery procedures.

---

## 6. Task 4: Custom Dockerfiles and Rebuilding

### Objective

Modify the Flask application, rebuild its image, and restart the service.

### Step 1: Make a code change

Open `app/app.py` and change the application name from:

```python
"application": "Day 34 DevOps Stack",
```

to:

```python
"application": "Day 34 Advanced Compose Platform",
```

Save the file.

### Step 2: Rebuild and restart

```bash
docker compose up -d --build
```

**Explanation:** Rebuilds services configured with `build:` when necessary and recreates containers when their image or configuration changes.

### Step 3: Verify the change

```bash
curl http://localhost:5000/
```

**Expected result:** The JSON response contains the updated application name.

Check the image and container status:

```bash
docker compose images
docker compose ps
```

### Why is this important?

In a containerized workflow, source code is packaged into an image. Rebuilding ensures that the running container uses the updated application code.

**Best practice:** Pin dependency versions, use small trusted base images, run as a non-root user, and scan images before deploying to production.

---

## 7. Task 5: Named Networks, Volumes, and Labels

### Objective

Organize services explicitly and persist database data independently of the database container.

### Step 1: Verify the named volume

```bash
docker volume ls
docker volume inspect day-34-compose-advanced_postgres_data
```

The exact volume name depends on the Compose project name. Use `docker volume ls` to find the actual name.

### Step 2: Verify the network

```bash
docker network ls
docker network inspect day-34-compose-advanced_backend
```

Again, use the actual name returned by Docker if it differs.

### Step 3: Inspect labels

```bash
docker inspect $(docker compose ps -q web)
```

In PowerShell:

```powershell
$webId = docker compose ps -q web
docker inspect $webId
```

Look for the `Labels` section.

### Step 4: Verify persistence

Insert data by requesting the application:

```bash
curl http://localhost:5000/
```

Stop and remove the stack:

```bash
docker compose down
```

Start it again:

```bash
docker compose up -d --build
```

Request the application again:

```bash
curl http://localhost:5000/
```

**Expected result:** The PostgreSQL visit count continues from the previous value because the named volume persists. The Redis counter resets because Redis is not configured with persistent storage in this exercise.

### Why use explicit networks?

- They make the network design visible and easier to maintain.
- They isolate services from unrelated containers.
- They provide service-name DNS resolution.
- They allow services to be attached to multiple networks when necessary.

### Why use labels?

Labels make containers easier to identify and filter by project, service role, environment, or team.

For example:

```bash
docker ps --filter "label=com.example.project=day-34"
```

**Conclusion:** Explicit resources make Compose configurations easier to understand, inspect, and maintain.

---

## 8. Task 6: Scaling the Web Application (Bonus)

### Objective

Understand why scaling a service with a fixed published host port creates a conflict.

### Step 1: Try scaling to three replicas

The web service currently publishes:

```yaml
ports:
  - "5000:5000"
```

Run:

```bash
docker compose up -d --scale web=3
```

**Expected result:** Compose will generally report a port allocation error because multiple replicas cannot all bind the same host port `5000`.

### Why does it fail?

Each replica can listen on port 5000 inside its own container. However, all replicas cannot independently claim the same host IP address and host port combination.

This is a port publishing conflict, not a limitation on how many containers Docker can run.

### Step 2: Understand the solution

A common production design uses a reverse proxy or load balancer in front of multiple web replicas.

```text
                Browser
                   |
              Host Port 5000
                   |
             +-----v------+
             |  Reverse   |
             |   Proxy    |
             +-----+------+
                   |
           +-------+-------+
           |       |       |
         Web 1   Web 2   Web 3
           |       |       |
           +-------+-------+
                   |
            PostgreSQL / Redis
```

The reverse proxy listens on the host port and distributes requests to the application containers through the Docker network.

### Step 3: Clean up

If the scaling command failed, check the current state:

```bash
docker compose ps -a
```

Restore the original configuration and run:

```bash
docker compose up -d --build
```

**Conclusion:** Scaling a service requires an appropriate traffic-routing strategy. Multiple replicas alone do not provide load balancing.

---

## 9. Troubleshooting Guide

| Problem | Possible cause | Solution |
|---|---|---|
| Flask cannot connect to PostgreSQL | Database is not ready or credentials are incorrect | Check `docker compose logs db` and verify `.env` |
| Redis connection fails | Redis is not healthy or the hostname is wrong | Check `docker compose ps` and verify `REDIS_HOST=cache` |
| Port 5000 is unavailable | Another application is using the port | Change the host port mapping |
| Application changes do not appear | Old image or container is running | Run `docker compose up -d --build` |
| Container restarts repeatedly | Application error or incorrect configuration | Inspect logs and health status |
| Database data disappears | The volume was removed or the mount was incorrect | Verify the named volume and its mount |
| Scaling fails | Multiple replicas compete for the same published host port | Use a reverse proxy or a suitable routing architecture |
| `.env` variables are missing | File is in the wrong directory or variable names differ | Check the project directory and run `docker compose config` |

Useful diagnostic commands:

```bash
docker compose ps -a
docker compose logs --tail=100 web
docker compose logs --tail=100 db
docker compose logs --tail=100 cache
docker compose config
docker compose top
docker stats
docker volume ls
docker network ls
```

---



## 11. Real-World Applications

### Where these concepts are used

1. **Web applications:** Flask or Node.js services communicate with databases and caches.
2. **Database reliability:** Health checks and restart policies help services recover from common failures.
3. **Performance optimization:** Redis can reduce repeated database queries when used with an appropriate caching strategy.
4. **Infrastructure management:** Named networks, volumes, and labels help organize application resources.
5. **Continuous integration and deployment:** Custom images can be rebuilt and tested when application code changes.
6. **Horizontal scaling:** Multiple application replicas can handle additional requests when paired with suitable load balancing and shared-state design.

### Production improvements to consider

- Use a dedicated secret-management solution for credentials.
- Pin image versions and scan images for vulnerabilities.
- Add proper application readiness checks and structured logs.
- Configure PostgreSQL backups and test restoration.
- Set resource limits and monitor CPU, memory, and application errors.
- Use a reverse proxy or load balancer before scaling web replicas.
- Avoid relying on Compose restart policies as a replacement for monitoring and recovery planning.

---

## 12. Interview Questions

**Q1. What is the purpose of Docker Compose?**

It defines and manages multi-container applications through a declarative YAML configuration.

**Q2. What is the difference between `depends_on` and `condition: service_healthy`?**

Basic `depends_on` controls startup ordering. The health condition allows Compose to wait until the dependency passes its configured health check.

**Q3. What is a Docker health check?**

A command or test used by Docker to classify a container's health as starting, healthy, or unhealthy.

**Q4. What is the difference between `always` and `on-failure`?**

`always` restarts containers after exits, subject to Docker's manual-stop behavior. `on-failure` restarts them after non-zero exit statuses, optionally with a retry limit.

**Q5. Why use a named volume for PostgreSQL?**

It preserves database files independently of the database container's lifecycle.

**Q6. Why can a service fail to scale with a fixed port mapping?**

Multiple replicas cannot all publish the same host IP and port combination.

**Q7. Why use Redis in an application stack?**

Redis can provide fast in-memory caching, counters, and other data structures, reducing repeated work when used appropriately.

**Q8. Does `docker compose up -d --build` always rebuild every image?**

It builds services with build configurations as needed. Docker may reuse cached layers, and services using only prebuilt images are not rebuilt from a local Dockerfile.

---

## 13. Cleanup

Stop and remove containers and the Compose-created network:

```bash
docker compose down
```

This preserves the named PostgreSQL volume by default.

To remove the PostgreSQL data volume as well, only after you have finished the experiment and no longer need the data:

```bash
docker compose down -v
```

**Warning:** The second command deletes Compose-managed named volumes and can permanently remove your local database data.

Verify remaining resources:

```bash
docker ps -a
docker volume ls
docker network ls
```

# Day 34: Docker Compose Advanced
## Complete Command Reference

### 1. Project Setup

| Command | One-line explanation |
|---|---|
| `mkdir day-34-compose-advanced` | Creates the main project directory. |
| `cd day-34-compose-advanced` | Moves into the project directory. |
| `mkdir app` | Creates a directory for the Flask application. |
| `New-Item app/app.py -ItemType File` | Creates the Flask application file in PowerShell. |
| `New-Item app/requirements.txt -ItemType File` | Creates the Python dependencies file. |
| `New-Item app/Dockerfile -ItemType File` | Creates the Dockerfile for building the custom image. |
| `New-Item docker-compose.yml -ItemType File` | Creates the Docker Compose configuration file. |
| `New-Item .env -ItemType File` | Creates a file for local environment variables. |
| `New-Item .dockerignore -ItemType File` | Creates a file specifying which files to exclude from the Docker build context. |
| `New-Item .gitignore -ItemType File` | Creates a file specifying which files Git should ignore. |

### 2. Docker and Compose Verification

| Command | One-line explanation |
|---|---|
| `docker --version` | Displays the installed Docker CLI version. |
| `docker compose version` | Displays the Docker Compose version. |
| `docker info` | Displays Docker Engine configuration and status information. |
| `docker compose --help` | Lists available Docker Compose commands. |
| `docker compose config` | Validates and displays the resolved Compose configuration. |

### 3. Build and Start the Application Stack

| Command | One-line explanation |
|---|---|
| `docker compose up` | Creates and starts the services in the foreground. |
| `docker compose up -d` | Starts all services in detached mode. |
| `docker compose up -d --build` | Builds configured application images and starts or recreates services as needed. |
| `docker compose ps` | Displays the status of the project's containers. |
| `docker compose ps -a` | Displays the project's containers, including stopped containers. |
| `docker compose images` | Lists images associated with the Compose project. |
| `docker compose top` | Displays the processes running inside the project's containers. |

### 4. Application Testing

| Command | One-line explanation |
|---|---|
| `curl http://localhost:5000/` | Sends an HTTP request to the Flask application and displays its response. |
| `curl http://localhost:5000/health` | Checks the Flask application's health endpoint. |
| `docker compose logs` | Displays logs from all Compose services. |
| `docker compose logs web` | Displays logs from the Flask application service. |
| `docker compose logs db` | Displays logs from PostgreSQL. |
| `docker compose logs cache` | Displays logs from Redis. |
| `docker compose logs --tail=100 web` | Displays the last 100 lines of Flask logs. |
| `docker compose logs -f web` | Continuously follows the Flask application's logs until interrupted. |

### 5. Health Checks and Dependencies

| Command | One-line explanation |
|---|---|
| `docker compose exec db pg_isready` | Checks whether PostgreSQL is accepting connections. |
| `docker compose ps` | Shows the service state and available health information. |
| `docker compose up -d --build` | Starts the stack after building the application image when required. |
| `docker compose down` | Stops and removes the project's containers and default network. |

### 6. Restart Policy Testing

| Command | One-line explanation |
|---|---|
| `docker compose kill db` | Forcefully terminates the database container's main process to test restart behavior. |
| `docker compose ps -a` | Checks whether the database container is running, restarting, or stopped. |
| `docker compose logs db` | Displays database logs to help diagnose restarts or startup failures. |
| `docker compose up -d` | Applies Compose configuration changes and starts services as needed. |
| `docker compose up -d --force-recreate db` | Recreates the database container even when its configuration has not changed. |
| `docker compose exec db pg_isready` | Verifies that PostgreSQL is ready after a restart. |

**Restart policies practiced:** `no`, `always`, `unless-stopped`, and `on-failure:3`.

### 7. Custom Dockerfile and Rebuilding

| Command | One-line explanation |
|---|---|
| `docker compose build` | Builds images for services configured with `build:`. |
| `docker compose up -d --build` | Rebuilds configured application images and starts the updated services. |
| `docker compose images` | Shows the images used by the Compose project. |
| `docker compose ps` | Verifies the state of the updated containers. |
| `curl http://localhost:5000/` | Checks whether the updated application code is running. |

### 8. Named Networks and Volumes

| Command | One-line explanation |
|---|---|
| `docker volume ls` | Lists Docker volumes available on the host. |
| `docker volume inspect day-34-compose-advanced_postgres_data` | Displays details of the PostgreSQL named volume if that is its actual name. |
| `docker network ls` | Lists Docker networks. |
| `docker network inspect day-34-compose-advanced_backend` | Displays network details if that is the actual Compose-generated network name. |
| `docker inspect $(docker compose ps -q web)` | Inspects the Flask container in Bash to view its configuration, labels, networks, and mounts. |
| `$webId = docker compose ps -q web` | Stores the Flask container ID in a PowerShell variable. |
| `docker inspect $webId` | Inspects the Flask container using the PowerShell variable. |
| `docker ps --filter "label=com.example.project=day-34"` | Lists containers carrying the specified project label. |
| `docker stats` | Displays live CPU, memory, network, and block I/O usage for containers. |

### 9. Scaling the Web Application

| Command | One-line explanation |
|---|---|
| `docker compose up -d --scale web=3` | Attempts to run three replicas of the web service. |
| `docker compose ps -a` | Checks the state of the web replicas after a scaling attempt. |
| `docker compose up -d --build` | Restores the normal stack configuration and starts the services. |

**Important:** Scaling may fail when every replica attempts to publish the same fixed host port, such as `5000:5000`.

### 10. Stop, Remove, and Clean Up

| Command | One-line explanation |
|---|---|
| `docker compose stop` | Stops services without removing their containers. |
| `docker compose start` | Starts existing stopped containers. |
| `docker compose down` | Removes the project's containers and default network while retaining named volumes by default. |
| `docker compose down -v` | Removes containers, the default network, and Compose-managed named volumes. |
| `docker ps -a` | Lists all containers, including stopped containers. |
| `docker volume ls` | Checks which Docker volumes remain after cleanup. |
| `docker network ls` | Checks which Docker networks remain after cleanup. |

**Warning:** `docker compose down -v` can permanently delete the PostgreSQL data stored in the project's named volume.

---

## 11. Essential Day 34 Workflow

Use these commands in order for the normal development workflow:

```bash
# 1. Validate the Compose configuration
docker compose config

# 2. Build the application and start all services
docker compose up -d --build

# 3. Check the service status
docker compose ps

# 4. Check application logs
docker compose logs -f web

# 5. Test the Flask endpoint
curl http://localhost:5000/

# 6. Check PostgreSQL readiness
docker compose exec db pg_isready

# 7. Inspect the database volume
docker volume ls

# 8. Stop and remove containers and the default network
docker compose down
```



---


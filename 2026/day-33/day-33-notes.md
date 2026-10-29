# Day 33: Docker Compose, Multi-Container Basics

> **#90DaysOfDevOps | Hands-on Practice | Docker Compose**

## 1. Objective

Today's goal is to run and manage multi-container applications using a single command with Docker Compose.

Previously, containers, networks, and volumes were created manually. Docker Compose simplifies this process by defining services, networks, volumes, ports, and environment variables in a YAML configuration file.

### Learning Outcomes

- Verify Docker Compose installation.
- Run an Nginx container using a Compose file.
- Deploy WordPress and MySQL together.
- Understand service discovery and container communication.
- Manage application lifecycle using Compose commands.
- Configure environment variables through YAML and `.env` files.
- Understand persistent storage in multi-container applications.

## 2. Prerequisites

- Docker Desktop or Docker Engine installed and running.
- Docker Compose V2 available.
- A terminal such as PowerShell, Command Prompt, or Bash.
- A web browser.

Verify the installation:

```bash
docker --version
docker compose version
docker info
```

**Explanation:**

- `docker --version`: Displays the Docker CLI version.
- `docker compose version`: Verifies the Compose plugin.
- `docker info`: Displays Docker Engine information.

Expected Compose output:

```text
Docker Compose version v2.x.x
```

The exact version depends on your installation.

---

## 3. Task 1: Install and Verify

### Step 1: Check Docker Compose Availability

```bash
docker compose version
```

If a version is displayed, Docker Compose is available.

### Step 2: Check Available Commands

```bash
docker compose --help
```

This displays commands such as `up`, `down`, `ps`, `logs`, `stop`, and `build`.

### Step 3: Record the Results

| Check | Command | Expected result |
|---|---|---|
| Docker CLI | `docker --version` | Docker version displayed |
| Compose | `docker compose version` | Compose version displayed |
| Docker Engine | `docker info` | Engine information displayed |

**Result:** Docker Compose is ready for the multi-container exercises if the commands execute successfully.

---

## 4. Task 2: Create Your First Compose File

### Objective

Run an Nginx web server using a Compose YAML file instead of manually creating a container.

### Step 1: Create the Project Directory

In PowerShell:

```powershell
mkdir compose-basics
cd compose-basics
```

### Step 2: Create `docker-compose.yml`

Create a file named `docker-compose.yml` inside `compose-basics`.

```yaml
services:
  web:
    image: nginx:alpine
    container_name: compose-nginx
    ports:
      - "8080:80"
    restart: unless-stopped
```

### Explanation

- `services`: Defines the application's containers.
- `web`: The Compose service name.
- `image`: Specifies the Nginx image.
- `container_name`: Sets a fixed container name for this exercise.
- `ports`: Maps host port `8080` to container port `80`.
- `restart`: Restarts the container unless it has been explicitly stopped.

### Step 3: Validate the Compose File

```bash
docker compose config
```

This checks the Compose configuration and displays the resolved configuration.

### Step 4: Start the Application

```bash
docker compose up
```

Docker downloads the Nginx image if needed and starts the service.

To start it in the background instead:

```bash
docker compose up -d
```

### Step 5: Verify the Container

```bash
docker compose ps
```

Test the web server:

```bash
curl http://localhost:8080
```

Alternatively, open this URL in your browser:

http://localhost:8080

**Expected result:** The default Nginx welcome page appears.

### Step 6: Stop and Remove the Application

```bash
docker compose down
```

This removes the project's containers and its default network.

### Expected Project Structure

```text
compose-basics/
└── docker-compose.yml
```

**Conclusion:** A single Compose file replaces several manual container-management commands.

---

## 5. Task 3: WordPress and MySQL Multi-Container Setup

### Objective

Deploy WordPress and MySQL together, connect them through a shared Compose network, and preserve database data with a named volume.

### Architecture

```text
                 Browser
                    |
                 Port 8080
                    |
             +------v------+
             |  WordPress  |
             |   Service   |
             +------+------+
                    |
             Compose Network
                    |
             +------v------+
             |    MySQL    |
             |   Service   |
             +------+------+
                    |
             +------v------+
             | Named Volume|
             |  db_data    |
             +-------------+
```

### Step 1: Create the Project Directory

Return to the parent directory and create a new project:

```powershell
cd ..
mkdir wordpress-compose
cd wordpress-compose
```

### Step 2: Create `docker-compose.yml`

```yaml
services:
  db:
    image: mysql:8.4
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: ${MYSQL_PASSWORD:?Set MYSQL_PASSWORD in .env}
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:?Set MYSQL_ROOT_PASSWORD in .env}
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-uroot", "-p$${MYSQL_ROOT_PASSWORD}"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 30s

  wordpress:
    image: wordpress:latest
    restart: unless-stopped
    ports:
      - "8081:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD:?Set MYSQL_PASSWORD in .env}
      WORDPRESS_DB_NAME: wordpress
    depends_on:
      db:
        condition: service_healthy

volumes:
  db_data:
```

### Explanation

**MySQL service:**

- `image`: Uses MySQL 8.4.
- `MYSQL_DATABASE`: Creates the initial WordPress database.
- `MYSQL_USER`: Creates the database user.
- `MYSQL_PASSWORD`: Sets the application user's password.
- `MYSQL_ROOT_PASSWORD`: Sets the MySQL root password.
- `db_data`: Stores database files persistently.
- `healthcheck`: Checks whether MySQL is responding.

**WordPress service:**

- `ports`: Publishes WordPress on host port `8081`.
- `WORDPRESS_DB_HOST: db:3306`: Uses the Compose service name `db` to locate MySQL.
- `depends_on`: Waits for the database health check to pass before starting WordPress.

**Important:** Compose automatically creates a project network and attaches both services to it. Therefore, WordPress can reach MySQL using `db` as the hostname without manually configuring a network.

### Step 3: Create the `.env` File

Create a file named `.env` in the same directory as `docker-compose.yml`.

```dotenv
MYSQL_PASSWORD=ChangeThisAppPassword123
MYSQL_ROOT_PASSWORD=ChangeThisRootPassword456
```

These are demonstration credentials. Use stronger, unique credentials for any deployment beyond local practice.

### Step 4: Validate the Configuration

```bash
docker compose config
```

This resolves environment-variable references and validates the Compose configuration.

Be careful when sharing the output because resolved configuration can expose passwords.

### Step 5: Start Both Services

```bash
docker compose up -d
```

Check their status:

```bash
docker compose ps
```

Check the logs:

```bash
docker compose logs
```

Wait until MySQL becomes healthy and WordPress starts.

### Step 6: Open WordPress

Open this URL:

http://localhost:8081

Follow the WordPress setup wizard:

1. Select your preferred language.
2. Enter a site title.
3. Create a WordPress administrator account.
4. Set a strong administrator password.
5. Complete the installation.

**Expected result:** The WordPress website is accessible, and its database connection works.

### Step 7: Verify Database Persistence

Create a test post in the WordPress dashboard and publish it.

Stop and remove the containers:

```bash
docker compose down
```

Start them again:

```bash
docker compose up -d
```

Wait for the services to become ready, then open:

http://localhost:8081

Check whether the published post and WordPress settings remain available.

**Expected result:** The post and settings remain because the MySQL database is stored in the `db_data` named volume.

Verify the volume:

```bash
docker volume ls
docker volume inspect wordpress-compose_db_data
```

The volume's exact name depends on the Compose project name. If needed, use `docker volume ls` to identify it.

### Important Notes

- `docker compose down` removes containers and the default network, but retains named volumes.
- `docker compose down -v` also removes Compose-managed named volumes. Avoid this command if you want to preserve the database.
- The WordPress image and MySQL image are separate services, but Compose manages them as one application.
- The `.env` file supplies configuration values to Compose. Add `.env` to `.gitignore` to avoid committing credentials.

**Conclusion:** WordPress and MySQL can run as independent containers while communicating through the Compose network and preserving data through a named volume.

---

## 6. Task 4: Docker Compose Commands

Use these commands from the directory containing the relevant Compose file.

| Task | Command | Purpose |
|---|---|---|
| Start services | `docker compose up` | Starts services in the foreground |
| Start in detached mode | `docker compose up -d` | Starts services in the background |
| View running services | `docker compose ps` | Shows service and container status |
| View all service logs | `docker compose logs` | Displays logs from all services |
| Follow logs continuously | `docker compose logs -f` | Streams logs in real time |
| View one service's logs | `docker compose logs wordpress` | Displays WordPress logs |
| View database logs | `docker compose logs db` | Displays MySQL logs |
| Stop services | `docker compose stop` | Stops containers without removing them |
| Start stopped services | `docker compose start` | Starts existing stopped containers |
| Remove containers and network | `docker compose down` | Stops and removes project containers and network |
| Rebuild images | `docker compose build` | Rebuilds images for services with build configurations |
| Rebuild and start | `docker compose up -d --build` | Builds configured images and starts services |
| Validate configuration | `docker compose config` | Resolves and validates Compose configuration |

### Practical Exercise

Run the following commands one by one:

```bash
docker compose up -d
docker compose ps
docker compose logs
docker compose logs -f wordpress
```

Press `Ctrl+C` to stop following the logs. This does not stop the containers.

Then execute:

```bash
docker compose stop
docker compose ps -a
docker compose start
docker compose ps
docker compose down
```

### Rebuilding an Image

The `docker compose build` command applies to services that contain a `build:` configuration.

For example:

```yaml
services:
  app:
    build: .
    ports:
      - "3000:3000"
```

If the Dockerfile or source used during the image build changes, rebuild and restart:

```bash
docker compose up -d --build
```

**Important:** Changing only an environment variable or a mounted configuration file does not always require rebuilding the image. Restart or recreate the affected service when appropriate.

---

## 7. Task 5: Environment Variables

### Objective

Learn how to configure application settings using environment variables in Compose.

### Method 1: Define Variables Directly in YAML

Example:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8082:80"
    environment:
      APP_ENV: development
      APP_NAME: compose-demo
```

Start the service:

```bash
docker compose up -d
```

Verify the variables inside the container:

```bash
docker compose exec web sh -c 'printf "%s\n" "$APP_ENV" "$APP_NAME"'
```

**Expected output:**

```text
development
compose-demo
```

This demonstrates that the variables were passed to the container.

### Method 2: Use a `.env` File

Create a `.env` file in the same directory:

```dotenv
APP_ENV=development
APP_NAME=compose-demo
WEB_PORT=8083
```

Update `docker-compose.yml`:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "${WEB_PORT}:80"
    environment:
      APP_ENV: ${APP_ENV}
      APP_NAME: ${APP_NAME}
```

Start the application:

```bash
docker compose up -d
```

Verify the configuration:

```bash
docker compose config
```

Verify the values inside the running container:

```bash
docker compose exec web sh -c 'printf "%s\n" "$APP_ENV" "$APP_NAME"'
```

**Expected output:**

```text
development
compose-demo
```

Open the Nginx server:

http://localhost:8083

### Difference Between `.env` and `environment`

- `.env` supplies values for variable substitution in the Compose file.
- `environment` passes specified variables into the container.
- A variable in `.env` is not automatically available inside a container unless the Compose configuration passes it through or uses another supported mechanism.

### Protect Credentials

Create a `.gitignore` file in your project directory:

```gitignore
.env
```

Do not commit real passwords, API keys, or other secrets to GitHub.

For production deployments, use an appropriate secrets-management solution rather than treating a plain `.env` file as secure secret storage.

---

## 8. Troubleshooting Guide

| Problem | Likely cause | Solution |
|---|---|---|
| `docker compose` is unavailable | Compose plugin is missing or Docker installation needs attention | Verify Docker installation and Compose plugin |
| Port is already allocated | Another process uses the host port | Choose a different host port |
| WordPress cannot connect to MySQL | Incorrect credentials or database not ready | Check `.env`, health status, and database logs |
| WordPress setup disappears | Database volume was removed or persistence was not configured correctly | Verify the named volume and mount |
| Environment variable is empty | Variable is missing or not passed into the container | Check `.env`, YAML references, and container environment |
| YAML validation fails | Incorrect indentation or invalid configuration | Run `docker compose config` |
| Changes are not reflected | Existing container uses old configuration or image | Recreate the service or rebuild the image as appropriate |

Useful diagnostic commands:

```bash
docker compose ps -a
docker compose logs db
docker compose logs wordpress
docker compose config
docker volume ls
docker network ls
```

---


---

## 10. Real-World Applications

Docker Compose is useful for:

1. **Local development:** Start a frontend, backend, and database together.
2. **Testing environments:** Reproduce a consistent application setup for debugging.
3. **Database-backed applications:** Connect application services to persistent databases.
4. **Service configuration:** Manage ports, environment variables, and health checks in one file.
5. **Developer onboarding:** Allow team members to start a project using a documented command.

Compose is especially useful for development, testing, and smaller deployments. Larger production environments may require orchestration platforms such as Kubernetes, depending on availability, scaling, and reliability requirements.

---

## 11. Interview Questions

**Q1. What is Docker Compose?**

Docker Compose is a tool for defining and managing multi-container applications using a YAML configuration file.

**Q2. What is the difference between `docker compose up` and `docker compose down`?**

`up` creates and starts services. `down` stops and removes the project's containers and default network.

**Q3. What does detached mode mean?**

The `-d` option runs containers in the background and returns control of the terminal.

**Q4. How do WordPress and MySQL communicate in this setup?**

They communicate through the Compose network, with WordPress connecting to the database using `db:3306`.

**Q5. Does `docker compose down` delete named volumes?**

Not by default. Use `docker compose down -v` to remove Compose-managed named volumes, which can delete persistent data.

**Q6. What is the purpose of a `.env` file?**

It supplies variable values for Compose configuration. Those values must be explicitly passed to containers when the application needs them.

**Q7. What is the difference between `docker compose stop` and `docker compose down`?**

`stop` stops existing containers without removing them. `down` removes the project's containers and default network.

---

## 12. Final Summary

In Day 33, I learned to manage multi-container applications using Docker Compose.

- Verified Docker Compose installation.
- Created and ran an Nginx container using YAML.
- Deployed WordPress and MySQL together.
- Used a named volume for database persistence.
- Practiced Compose lifecycle and troubleshooting commands.
- Configured and verified environment variables.
- Learned why service names are useful for container communication.


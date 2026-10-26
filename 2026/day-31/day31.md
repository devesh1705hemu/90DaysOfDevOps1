# Day 31: Dockerfile, Build Your Own Images

## Objective

Learn to create custom Docker images using Dockerfiles, understand Dockerfile instructions, compare `CMD` and `ENTRYPOINT`, use `.dockerignore`, and optimize builds with caching.

Environment: Ubuntu EC2 instance  
Working directory: `~/practice-docker`

---

# Task 1: Your First Dockerfile

## 1. Create the project directory

```bash
mkdir my-first-image
cd my-first-image
pwd
```

- `mkdir my-first-image` creates the project directory.
- `cd my-first-image` moves into the project directory.
- `pwd` displays the current working directory.

## 2. Create the Dockerfile

Command:

```bash
nano Dockerfile
```

**Dockerfile:**

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

CMD ["echo", "Hello from my custom image!"]
```

### Dockerfile instruction uses

- `FROM ubuntu:latest` selects Ubuntu as the base image.
- `RUN apt-get update` refreshes the package index during the build.
- `apt-get install -y curl` installs curl without interactive confirmation.
- `rm -rf /var/lib/apt/lists/*` removes cached package index files to reduce image size.
- `CMD ["echo", "Hello from my custom image!"]` defines the default command that prints a message when the container starts.

## 3. Build the image

```bash
docker build -t my-ubuntu:v1 .
```

- `docker build -t my-ubuntu:v1 .` builds an image named `my-ubuntu` with tag `v1` using the current directory as the build context.

## 4. Run the image

```bash
docker run --rm my-ubuntu:v1
```

- `docker run --rm my-ubuntu:v1` runs a container and automatically removes it after the command finishes.

**Expected output:**

```text
Hello from my custom image!
```

## 5. Verify curl installation

```bash
docker run --rm my-ubuntu:v1 curl --version
```

- `docker run --rm my-ubuntu:v1 curl --version` overrides the default `CMD` and verifies that curl is installed.

---

# Task 2: Dockerfile Instructions

## 1. Create the project directory

```bash
cd ~/practice-docker
mkdir -p dockerfile-instructions
cd dockerfile-instructions
```

- `cd ~/practice-docker` moves to the main practice directory.
- `mkdir -p dockerfile-instructions` creates the project directory if it does not already exist.
- `cd dockerfile-instructions` moves into the project directory.

## 2. Create the HTML file

```bash
nano index.html
```

**File: `index.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>My Docker App</title>
</head>
<body>
    <h1>Hello from my Docker container!</h1>
    <p>Built using Dockerfile instructions.</p>
</body>
</html>
```

- `nano index.html` creates or edits the HTML file that will be copied into the image.

## 3. Create the Dockerfile

```bash
nano Dockerfile
```

**File: `Dockerfile`**

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y python3 && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY index.html .

EXPOSE 8000

CMD ["python3", "-m", "http.server", "8000", "--bind", "0.0.0.0"]
```

### Instruction uses

- `FROM ubuntu:latest` selects Ubuntu as the base image.
- `RUN ...` installs Python 3 during the image build.
- `WORKDIR /app` sets `/app` as the working directory for subsequent instructions.
- `COPY index.html .` copies the HTML file into `/app` inside the image.
- `EXPOSE 8000` documents the port the application is intended to listen on.
- `CMD [...]` starts Python's HTTP server on port 8000 when the container runs.

## 4. Build the image

```bash
docker build -t dockerfile-demo:v1 .
```

- `docker build -t dockerfile-demo:v1 .` builds and tags the custom image as `dockerfile-demo:v1`.

## 5. Run the container

```bash
docker run -d --name dockerfile-web -p 8080:8000 dockerfile-demo:v1
```

- `docker run -d --name dockerfile-web -p 8080:8000 dockerfile-demo:v1` starts the container in the background and maps host port 8080 to container port 8000.

## 6. Test the application

```bash
docker ps
curl http://localhost:8080
```

- `docker ps` lists running containers.
- `curl http://localhost:8080` sends an HTTP request to the application through the published port.

**Expected result:** The response contains the HTML from `index.html`.

## 7. Inspect the container

```bash
docker logs dockerfile-web
docker exec dockerfile-web pwd
docker exec dockerfile-web ls -l /app
docker inspect dockerfile-web
```

- `docker logs dockerfile-web` displays the container's logs.
- `docker exec dockerfile-web pwd` displays the container's current working directory.
- `docker exec dockerfile-web ls -l /app` lists files in the application directory.
- `docker inspect dockerfile-web` displays detailed container configuration and networking information.

## 8. Clean up

```bash
docker stop dockerfile-web
docker rm dockerfile-web
```

- `docker stop dockerfile-web` gracefully stops the container.
- `docker rm dockerfile-web` removes the stopped container but keeps its image.

---

# Task 3: CMD vs ENTRYPOINT

## 1. CMD example

Create the directory:

```bash
cd ~/practice-docker
mkdir -p cmd-demo
cd cmd-demo
nano Dockerfile
```

- `cd ~/practice-docker` moves to the practice directory.
- `mkdir -p cmd-demo` creates the CMD experiment directory if needed.
- `cd cmd-demo` enters the directory.
- `nano Dockerfile` creates or edits the Dockerfile.

**File: `cmd-demo/Dockerfile`**

```dockerfile
FROM ubuntu:latest
CMD ["echo", "hello"]
```

Build and run:

```bash
docker build -t cmd-demo:v1 .
docker run --rm cmd-demo:v1
docker run --rm cmd-demo:v1 echo "Hello from Docker"
```

- `docker build -t cmd-demo:v1 .` builds the CMD demonstration image.
- `docker run --rm cmd-demo:v1` executes the default command and prints `hello`.
- `docker run --rm cmd-demo:v1 echo "Hello from Docker"` replaces the default command and prints the custom message.

## 2. ENTRYPOINT example

Create the directory:

```bash
cd ~/practice-docker
mkdir -p entrypoint-demo
cd entrypoint-demo
nano Dockerfile
```

- `cd ~/practice-docker` moves to the practice directory.
- `mkdir -p entrypoint-demo` creates the ENTRYPOINT experiment directory if needed.
- `cd entrypoint-demo` enters the directory.
- `nano Dockerfile` creates or edits the Dockerfile.

**File: `entrypoint-demo/Dockerfile`**

```dockerfile
FROM ubuntu:latest
ENTRYPOINT ["echo"]
```

Build and run:

```bash
docker build -t entrypoint-demo:v1 .
docker run --rm entrypoint-demo:v1
docker run --rm entrypoint-demo:v1 "Hello from Docker"
docker run --rm entrypoint-demo:v1 DevOps Day 31
```

- `docker build -t entrypoint-demo:v1 .` builds the ENTRYPOINT demonstration image.
- `docker run --rm entrypoint-demo:v1` runs `echo` without arguments.
- `docker run --rm entrypoint-demo:v1 "Hello from Docker"` passes the message to the configured `echo` executable.
- `docker run --rm entrypoint-demo:v1 DevOps Day 31` passes the words as arguments to `echo`.

## 3. Combine ENTRYPOINT and CMD

**File: `combined-demo/Dockerfile`**

```dockerfile
FROM ubuntu:latest
ENTRYPOINT ["echo"]
CMD ["Hello from Docker"]
```

Build and run from the directory containing this Dockerfile:

```bash
docker build -t combined-demo:v1 .
docker run --rm combined-demo:v1
docker run --rm combined-demo:v1 "Custom message"
```

- `docker build -t combined-demo:v1 .` builds the combined demonstration image.
- `docker run --rm combined-demo:v1` prints the default CMD argument.
- `docker run --rm combined-demo:v1 "Custom message"` replaces the default arguments while retaining the ENTRYPOINT executable.

### CMD vs ENTRYPOINT notes

| CMD | ENTRYPOINT |
|---|---|
| Provides a default command or arguments. | Defines the main executable. |
| Runtime arguments can replace the default command. | Runtime arguments are appended to the executable by default. |
| Useful when users should easily override the default behavior. | Useful for dedicated applications and command-line tools. |

**Key takeaway:** `ENTRYPOINT` defines what runs, while `CMD` provides default behavior or arguments.

---

# Task 4: Build a Simple Web App Image

## 1. Create the project directory

```bash
cd ~/practice-docker
mkdir -p my-website
cd my-website
```

- `cd ~/practice-docker` moves to the practice directory.
- `mkdir -p my-website` creates the website project directory if needed.
- `cd my-website` enters the project directory.

## 2. Create the HTML file

```bash
nano index.html
```

**File: `my-website/index.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Docker Website</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: #101827;
            color: white;
            text-align: center;
            padding: 80px 20px;
        }
        h1 {
            color: #55d6be;
        }
    </style>
</head>
<body>
    <h1>Hello from Docker!</h1>
    <p>My first website is running inside an NGINX container.</p>
    <p>Day 31 | #90DaysOfDevOps</p>
</body>
</html>
```

- `nano index.html` creates or edits the static webpage.

## 3. Create the Dockerfile

```bash
nano Dockerfile
```

**File: `my-website/Dockerfile`**

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
```

- `FROM nginx:alpine` selects the lightweight NGINX Alpine image.
- `COPY index.html /usr/share/nginx/html/index.html` copies the webpage into NGINX's default document directory.

## 4. Build the website image

```bash
docker build -t my-website:v1 .
docker images
```

- `docker build -t my-website:v1 .` builds and tags the website image as `my-website:v1`.
- `docker images` lists local images and their sizes.

## 5. Run the website

```bash
docker run -d --name my-website-container -p 8080:80 my-website:v1
docker ps
curl http://localhost:8080
```

- `docker run -d --name my-website-container -p 8080:80 my-website:v1` starts the website in the background and publishes host port 8080 to container port 80.
- `docker ps` verifies that the container is running.
- `curl http://localhost:8080` verifies the HTTP response from the website.

To access it externally, visit `http://YOUR_EC2_PUBLIC_IP:8080` after allowing inbound TCP port 8080 from your own public IP in the EC2 security group.

## 6. Inspect the website container

```bash
docker logs my-website-container
docker exec my-website-container ls -l /usr/share/nginx/html
docker inspect my-website-container
```

- `docker logs my-website-container` displays NGINX logs.
- `docker exec my-website-container ls -l /usr/share/nginx/html` verifies that the HTML file exists inside the container.
- `docker inspect my-website-container` displays detailed container configuration.

---

# Task 5: .dockerignore

## 1. Create the ignore file

```bash
cd ~/practice-docker/my-website
nano .dockerignore
```

- `cd ~/practice-docker/my-website` moves to the website project.
- `nano .dockerignore` creates or edits the Docker build-context ignore file.

**File: `my-website/.dockerignore`**

```text
node_modules
.git
*.md
.env
```

### Uses of each entry

- `node_modules` excludes the Node.js dependency directory from the build context.
- `.git` excludes Git repository metadata.
- `*.md` excludes Markdown files.
- `.env` excludes files named `.env` at the applicable path.

## 2. Create temporary test files

```bash
mkdir -p node_modules .git
touch README.md notes.md .env
ls -la
```

- `mkdir -p node_modules .git` creates test directories if they do not already exist.
- `touch README.md notes.md .env` creates empty test files.
- `ls -la` lists files, including hidden files.

**Caution:** Do not create or modify `.git` if it already contains your actual repository metadata.

## 3. Create a diagnostic Dockerfile

```bash
nano Dockerfile.test
```

**File: `my-website/Dockerfile.test`**

```dockerfile
FROM alpine:latest
WORKDIR /app
COPY . .
CMD ["sh", "-c", "find /app -maxdepth 2 -print"]
```

- `FROM alpine:latest` selects Alpine Linux as the base image.
- `WORKDIR /app` sets the working directory.
- `COPY . .` copies the non-ignored build-context files into `/app`.
- `CMD [...]` lists copied files when the container runs.

## 4. Build and verify

```bash
docker build --no-cache -f Dockerfile.test -t dockerignore-test:v1 .
docker run --rm dockerignore-test:v1
```

- `docker build --no-cache -f Dockerfile.test -t dockerignore-test:v1 .` builds the diagnostic image without reusing cached build steps.
- `docker run --rm dockerignore-test:v1` lists files copied into the image.

**Expected observation:** The output should not list `README.md`, `notes.md`, `.env`, `node_modules`, or `.git` when these entries match the `.dockerignore` rules and exist in the build context.

## 5. Remove temporary test resources

```bash
rm -f Dockerfile.test README.md notes.md .env
rm -rf node_modules
docker image rm dockerignore-test:v1
```

- `rm -f Dockerfile.test README.md notes.md .env` removes the temporary files created for testing.
- `rm -rf node_modules` removes the temporary test directory.
- `docker image rm dockerignore-test:v1` removes the diagnostic image.

**Important:** Do not run `rm -rf .git` if it contains your actual Git repository metadata.

### Key notes

- `.dockerignore` filters files from the build context before they are sent to the builder.
- It helps reduce build context size and avoid accidentally copying sensitive files.
- It does not remove files independently created inside the image.
- It does not replace proper secret management.

---

# Task 6: Build Optimization

## 1. Build the image

```bash
cd ~/practice-docker/my-website
docker build -t my-website:v3 .
```

- `cd ~/practice-docker/my-website` moves to the website project.
- `docker build -t my-website:v3 .` builds and tags the image as `my-website:v3`.

## 2. Rebuild without changing files

```bash
docker build -t my-website:v3 .
```

- `docker build -t my-website:v3 .` rebuilds the image and may reuse cached results for unchanged instructions and inputs.

**Observation:** With the legacy builder, the output may display `Using cache` for eligible cached steps.

## 3. Change the HTML file

```bash
nano index.html
```

- `nano index.html` edits the website content to test cache invalidation.

For example, change the heading to:

```html
<h1>Hello from Docker Day 31!</h1>
```

Rebuild:

```bash
docker build -t my-website:v3 .
```

- `docker build -t my-website:v3 .` rebuilds the image, rerunning the `COPY` step because its source file changed.

## 4. Understand layer order

**Less efficient Dockerfile example:**

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]
```

If source code changes, the `COPY` instruction changes, which can invalidate the subsequent dependency installation step.

**Optimized Dockerfile example:**

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
```

- `FROM node:22-alpine` selects a Node.js base image.
- `WORKDIR /app` sets the application working directory.
- `COPY package*.json ./` copies dependency manifests first.
- `RUN npm install` installs dependencies during the image build.
- `COPY . .` copies the application source after dependency installation.
- `CMD ["npm", "start"]` defines the default application startup command.

This optimization works when dependency manifests remain unchanged. The project must contain valid package manifests and an appropriate lockfile.

### Why does layer order matter?

1. Docker executes build instructions in order.
2. Unchanged instructions and inputs can reuse cached results.
3. Changes to earlier instructions can invalidate subsequent cache entries.
4. Stable dependencies should generally be installed before frequently changing source code is copied.
5. `.dockerignore` helps prevent unnecessary files from affecting the build context.

**Real-world application:** Good cache design reduces image build times in CI/CD pipelines and speeds up application deployments.

---

# Day 31: Important Commands Summary

| Command | One-line use |
|---|---|
| `mkdir my-first-image` | Creates a project directory. |
| `cd my-first-image` | Changes the current directory. |
| `pwd` | Displays the current working directory. |
| `nano Dockerfile` | Creates or edits a Dockerfile. |
| `docker build -t my-ubuntu:v1 .` | Builds and tags a custom image. |
| `docker run --rm my-ubuntu:v1` | Runs a container and removes it after exit. |
| `docker images` | Lists local Docker images and sizes. |
| `docker image inspect nginx` | Displays image metadata and configuration. |
| `docker run -d --name dockerfile-web -p 8080:8000 dockerfile-demo:v1` | Starts a container in the background and publishes its port. |
| `docker ps` | Lists running containers. |
| `docker logs dockerfile-web` | Displays container logs. |
| `docker exec dockerfile-web pwd` | Executes a command inside a running container. |
| `docker inspect dockerfile-web` | Displays detailed container metadata. |
| `docker stop dockerfile-web` | Gracefully stops a container. |
| `docker rm dockerfile-web` | Removes a stopped container. |
| `docker logs my-website-container` | Displays the website container's logs. |
| `curl http://localhost:8080` | Tests the website's HTTP response locally. |
| `docker build --no-cache -t my-website:v2 .` | Builds an image without reusing cached build steps. |
| `docker build -f Dockerfile.test -t dockerignore-test:v1 .` | Builds using a specific Dockerfile. |
| `docker run --rm dockerignore-test:v1` | Runs the diagnostic image and removes the container afterward. |
| `docker image rm dockerignore-test:v1` | Removes the diagnostic image. |
| `ls -la` | Lists files, including hidden files. |
| `cat .dockerignore` | Displays the Docker ignore rules. |

---

# Key Learnings

1. Dockerfiles define the instructions used to build custom images.
2. `FROM`, `RUN`, `COPY`, `WORKDIR`, `EXPOSE`, and `CMD` serve different purposes during image construction and container startup.
3. `CMD` provides default behavior, while `ENTRYPOINT` defines the executable that runs by default.
4. NGINX Alpine is useful for lightweight static website containers.
5. `.dockerignore` excludes unnecessary files from the build context.
6. Docker caching improves build efficiency when instructions and inputs remain unchanged.
7. Correct Dockerfile ordering reduces unnecessary rebuilds.
8. Container logs, inspection, and port mapping are essential troubleshooting skills.

# Real-World DevOps Applications

- Build custom images for applications.
- Package static websites for consistent deployment.
- Improve CI/CD pipeline performance with build caching.
- Prevent unnecessary files from entering build contexts.
- Reduce image size by choosing appropriate base images.
- Troubleshoot container startup, networking, and application behavior.

# Interview Questions

1. What is a Dockerfile?
2. What is the difference between `RUN` and `CMD`?
3. What is the difference between `CMD` and `ENTRYPOINT`?
4. What is the purpose of `WORKDIR`?
5. Does `EXPOSE` publish a port automatically?
6. What is `.dockerignore`, and why is it important?
7. How does Docker build caching work?
8. Why should dependency files be copied before frequently changing source files?
9. What is the difference between a Docker image and a container?
10. Why would you choose `nginx:alpine` for a static website?
## Workflow Overview with Commands

### 1. Create the project

```bash
mkdir -p ~/practice-docker/my-website
cd ~/practice-docker/my-website
touch index.html Dockerfile .dockerignore
```

**Purpose:** Create the project directory and required files.

### 2. Configure the Dockerfile

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

**Purpose:** Use a lightweight Nginx image and copy the website into its default web directory.

### 3. Build a Docker image

```bash
docker build -t my-website:v1 .
docker images
```

**Purpose:** Build the image from the current directory and list available images.

### 4. Run the container

```bash
docker run -d --name my-website-container -p 8080:80 my-website:v1
docker ps
```

**Purpose:** Start the website in the background, map host port `8080` to container port `80`, and verify the running container.

### 5. Test the application

```bash
curl http://localhost:8080
docker logs my-website-container
docker inspect my-website-container
```

**Purpose:** Check the website response, view container logs, and inspect container configuration.

### 6. Optimize the build

```bash
docker build -t my-website:v2 .
docker build --no-cache -t my-website:v3 .
```

**Purpose:** Build using Docker's layer cache, then rebuild without using cached layers for comparison.

### 7. Clean up resources

```bash
docker stop my-website-container
docker rm my-website-container
docker image rm my-website:v1 my-website:v2 my-website:v3
```

**Purpose:** Stop and remove the container, then remove the listed images when they are no longer needed.

**Note:** Run cleanup commands only after finishing your tests. If an image is still used by another container, Docker may refuse to remove it.

### Complete Workflow

**Create Files → Write Dockerfile → Build Image → Run Container → Test Application → Optimize Build → Clean Up**

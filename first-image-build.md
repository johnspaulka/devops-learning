# 🐳 Dockerfile Deep Dive for Freshers: Building Your First Container Images!

> **Target Audience:** Freshers | **Session Goal:** Understand the structure and core commands of a Dockerfile to build a simple application image.

---

## 🚀 Session Introduction: Why Dockerfiles?

Imagine a chef needing a **recipe** to cook a meal. A **Dockerfile** is that recipe for your software! It's a simple text file that contains all the commands a user could call on the command line to assemble an image.

- **Goal:** Ensure your application runs the same way everywhere (on your laptop, a server, or in the cloud).
- **Analogy:** A Dockerfile is the **blueprint**; the **Docker Image** is the house built from that blueprint; the **Docker Container** is the running instance of the house.

---

## 🛠️ Section 1: Dockerfile Fundamentals

| Command | Description | Example/Analogy |
| :--- | :--- | :--- |
| **`FROM`** | **The Base!** Specifies the **base image** your image will be built upon. All Dockerfiles *must* start with `FROM`. | Think of this as the *operating system* (e.g., Ubuntu, Alpine) or a pre-configured environment (e.g., Python, Node.js). |
| **`WORKDIR`** | **The Folder!** Sets the working directory for any subsequent commands like `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, etc. | This is like using the `cd` command inside the container to make sure you're in the right project folder. |
| **`COPY`** | **The Transfer!** Copies files or directories from your local host machine into the filesystem of the image. | *Example:* `COPY . /app` (Copy everything in the current host directory to `/app` in the image). |
| **`RUN`** | **The Executor!** Executes any command in a new layer on top of the current image and commits the results. Used for installing packages or building the app. | *Example:* `RUN apt-get update && apt-get install -y git`. This command is executed *during the image build*. |
| **`EXPOSE`** | **The Door!** Informs Docker that the container listens on the specified network ports at runtime. This is *documentation only*; you still need to map the port when running the container. | *Example:* `EXPOSE 8080` (Telling the world this app uses port 8080). |
| **`CMD`** | **The Default!** Provides defaults for an executing container. **Only the last `CMD` instruction in a Dockerfile will be executed.** | This is the command that **runs when the container starts** (e.g., starting your web server). |

---

## ⚙️ Section 2: Code Example - A Simple Python App

### 1. The Application Code (app.py)

Let's create a minimal Python script that runs a basic web server.

```python
# app.py
import http.server
import socketserver

PORT = 8080
Handler = http.server.SimpleHTTPRequestHandler

with socketserver.TCPServer(("", PORT), Handler) as httpd:
    print("serving at port", PORT)
    httpd.serve_forever()
```

### 2. The Requirements File (requirements.txt)

This file lists any Python packages needed (none in this case, but good practice!).

```text
# requirements.txt
# This app uses built-in Python modules, so this file is empty.
```

### 3. The Dockerfile

This is the **recipe** to containerize the app.

```dockerfile
# Dockerfile

# 1. Start from an official Python image (lightweight Alpine is often preferred)
FROM python:3.9-slim-buster

# 2. Set the working directory inside the container
WORKDIR /usr/src/app

# 3. Copy the requirements file (good practice for layer caching)
COPY requirements.txt ./

# 4. RUN a command to install dependencies (if any)
# This command is executed during the IMAGE BUILD
RUN pip install --no-cache-dir -r requirements.txt

# 5. Copy the rest of the application code
COPY . .

# 6. Inform Docker that the container will listen on this port
EXPOSE 8080

# 7. Define the command to run when the container starts
# The format ['executable', 'param1', 'param2'] is preferred (exec form)
CMD ["python", "app.py"]
```

---

## 🚀 Section 3: Building and Running the Image (Hands-on Steps)

### Step 1: Create the Project Structure

Create a folder named `docker-session` and place the `app.py`, `requirements.txt`, and `Dockerfile` inside it.

```bash
# Command to check the structure
$ ls docker-session
Dockerfile  app.py  requirements.txt
```

### Step 2: Build the Docker Image

The `docker build` command reads the Dockerfile and creates a new image.

| Command | Description |
| :--- | :--- |
| **`docker build -t session-app:v1 .`** | **`-t`**: Tag the image (`session-app`) and give it a version (`v1`). |
| | **`.`**: Tells Docker to look for the `Dockerfile` in the *current directory*. |

**Expected Output (Focus on Layers!):** You'll see each `FROM`, `RUN`, `COPY` command creating a **layer**. This is key to fast builds and small images!

```text
$ docker build -t session-app:v1 .
Sending build context to Docker daemon  3.072kB
Step 1/7 : FROM python:3.9-slim-buster
 ---> abc123def456
Step 2/7 : WORKDIR /usr/src/app
 ---> Running in ghi789jkl012
 ---> mno345pqr678
Step 3/7 : COPY requirements.txt ./
 ---> stu901vwx234
Step 4/7 : RUN pip install --no-cache-dir -r requirements.txt
 ---> Running in yz567abc890
 ---> def123ghi456
Step 5/7 : COPY . .
 ---> jkl789mno012
Step 6/7 : EXPOSE 8080
 ---> pqr345stu678
Step 7/7 : CMD ["python", "app.py"]
 ---> vwx901yz234
Successfully built vwx901yz234
Successfully tagged session-app:v1
```

### Step 3: Verify the Image

Check if your newly built image is present in your local Docker registry.

| Command | Description |
| :--- | :--- |
| **`docker images`** | Lists all local images. |

```text
$ docker images
REPOSITORY     TAG     IMAGE ID       CREATED         SIZE
session-app    v1      vwx901yz234    2 minutes ago   125MB
python         3.9-slim-buster  abc123def456  2 weeks ago    115MB
```

### Step 4: Run a Container from the Image

The `docker run` command creates and starts a container from the image.

| Command | Description |
| :--- | :--- |
| **`docker run -d -p 80:8080 session-app:v1`** | **`-d`**: Run the container in **detached mode** (in the background). |
| | **`-p 80:8080`**: **Port mapping!** Maps port **80** on your host machine to port **8080** (the `EXPOSE`d port) inside the container. |

**Expected Output:**
```bash
$ docker run -d -p 80:8080 session-app:v1
abc123def456789...  # Container ID
```

**Final Check:** Open your browser and navigate to **`http://localhost`** to see the default directory listing served by the Python app inside the container! 🎉

### Step 5: Verify the Running Container

```bash
# Check running containers
$ docker ps
CONTAINER ID   IMAGE           COMMAND           CREATED         STATUS         PORTS                  NAMES
abc123def456   session-app:v1  "python app.py"   1 minute ago    Up 1 minute    0.0.0.0:80->8080/tcp   amazing_tesla

# Check container logs
$ docker logs abc123def456
serving at port 8080
```

---

## 🧠 Section 4: Advanced Tips (For Better Dockerfiles)

### The `.dockerignore` File

- **Purpose:** Lists files and folders that **should NOT be copied** into the image.
- **Benefit:** Keeps the image smaller and build times faster.
- **Must-Haves:** `*.pyc`, `venv/`, `.git/`, `__pycache__/`, `node_modules/` (depending on the app type).

**Example `.dockerignore`:**
```text
*.pyc
__pycache__/
.git/
.gitignore
README.md
venv/
.env
```

### Minimizing Layers and Size

- **Rule of Thumb:** Combine related `RUN` commands using `&&` and the line continuation character (`\`).

**❌ Bad (Creates 3 layers):**
```dockerfile
RUN apt-get update
RUN apt-get install -y git
RUN rm -rf /var/lib/apt/lists/*
```

**✅ Good (Creates 1 layer and cleans up):**
```dockerfile
RUN apt-get update && \
    apt-get install -y git && \
    rm -rf /var/lib/apt/lists/*
```

### Layer Caching Best Practice

**Order matters!** Place instructions that change frequently at the bottom:

```dockerfile
FROM python:3.9-slim-buster
WORKDIR /usr/src/app

# Copy requirements first (changes less frequently)
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# Copy source code last (changes most frequently)
COPY . .

EXPOSE 8080
CMD ["python", "app.py"]
```

---

## 🎯 Session Summary

**What You've Accomplished:**

1. ✅ **Created** a simple Python web application
2. ✅ **Written** your first Dockerfile with essential commands
3. ✅ **Built** a Docker image from the Dockerfile
4. ✅ **Ran** a container from your custom image
5. ✅ **Learned** best practices for efficient Dockerfiles

**Key Commands to Remember:**
```bash
# Build an image
docker build -t my-app:v1 .

# List images
docker images

# Run a container
docker run -d -p 8080:80 my-app:v1

# List running containers
docker ps

# View container logs
docker logs <container-id>
```

**Next Steps:**
- Try modifying the Python app and rebuilding
- Experiment with different base images (e.g., `python:3.9-alpine`)
- Add more dependencies to `requirements.txt` and observe the build process

---

*Happy containerizing! 🐳*

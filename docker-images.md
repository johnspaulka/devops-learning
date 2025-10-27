# 🐳 Docker Images: Building Your Own Containers

> **Session Plan: Diving into the Dockerfile** - Master the art of creating custom Docker images from scratch

---

## 🎯 **What You'll Learn**

By the end of this guide, you'll confidently:
- ✅ Understand what a Dockerfile is and how it differs from shell scripts
- ✅ Write Dockerfiles using proper naming conventions and structure
- ✅ Use the FROM command to select appropriate base images
- ✅ Create efficient image layers with the RUN command
- ✅ Apply best practices for layer optimization and cleanup
- ✅ Expose ports properly and understand the difference between EXPOSE and port mapping

---

## 🐳 **Page 1: Understanding Docker Images and Tags**

<div class="images-section">

### 🖼️ **What is a Docker Image?**

A Docker image is a **read-only template** that contains everything needed to run an application:

- **Operating System:** Base OS (Linux distributions like Ubuntu, Alpine, Debian)
- **Runtime Environment:** Programming languages (Python, Node.js, Java)
- **Application Code:** Your application files and dependencies
- **Configuration:** Environment variables, startup commands, and settings
- **Metadata:** Information about the image (author, version, ports)

Think of a Docker image as a **snapshot** of a complete application environment that can be replicated anywhere Docker runs.

### 🏷️ **Docker Image Tags: Versioning and Identification**

Docker image tags are **labels** that help identify and version your images. They follow this format:

```bash
# Basic format
[REGISTRY/][NAMESPACE/]REPOSITORY:TAG

# Examples
nginx:latest                    # Official Nginx image, latest version
nginx:1.28.0-alpine             # Official nginx image, version 1.28.0-alpine

# Breaking down the full format with examples:
nginx, equivalent to docker.io/library/nginx:latest
# This pulls an image from the docker.io registry, the library namespace (contains official images), 
# the nginx image repository, and the latest tag

docker/welcome-to-docker, equivalent to docker.io/docker/welcome-to-docker:latest
# This pulls an image from the docker.io registry, the docker namespace, 
# the welcome-to-docker image repository, and the latest tag

ghcr.io/dockersamples/example-voting-app-vote:pr-311
# This pulls an image from the GitHub Container Registry, the dockersamples namespace, 
# the example-voting-app-vote image repository, and the pr-311 tag
```

### 📋 **Understanding Tag Components**

| Component | Purpose | Example |
|-----------|---------|---------|
| **Registry** | Where the image is stored | `docker.io`, `my-registry.com` |
| **Repository** | The image name/namespace | `nginx`, `my-app`, `company/webapp` |
| **Tag** | Version or variant identifier | `latest`, `v1.0`, `alpine`, `dev` |

### 🎯 **Common Tag Patterns**

```bash
# Version numbers
nginx:1.21.6
my-app:v2.1.0

# Architecture variants
node:18-alpine    # Alpine Linux variant (smaller)
python:3.11-slim  # Slim variant (minimal packages)

# Special tags
nginx:latest      # Most recent stable version
nginx:stable      # Stable release
```

### 🔍 **Default Tags and Behavior**

```bash
# When you don't specify a tag, Docker assumes 'latest'
docker pull nginx          # Same as: docker pull nginx:latest
docker run ubuntu          # Same as: docker run ubuntu:latest

# But 'latest' might not always be what you expect!
# It's just the default tag name, not necessarily the newest version
``` 

## 📄 **Page 2: Introduction to the Dockerfile**

<div class="intro-section">

### 🍳 **What is a Dockerfile?**

Think of a Dockerfile as a **recipe for creating a Docker image**. Just like a cooking recipe tells you the ingredients and steps to make a dish, a Dockerfile tells Docker the components and steps to build your custom image.

**🚨 Important:** A Dockerfile is **NOT** a shell script (like Bash). It's a unique language defined by Docker with its own syntax and rules.

### 📝 **Naming Convention**

```bash
# Default name (recommended)
Dockerfile

# Custom filename (requires -f flag)
Dockerfile.dev
my-app.Dockerfile # Preferred custom name as per convention
```

**💡 Best Practice:** Use the default name `Dockerfile` (with capital "D") unless you have multiple Dockerfiles in the same directory.

### 🏗️ **Using Custom Filenames**

When you need a different filename, use the `-f` flag:

```bash
# Build with custom filename
docker build -f Dockerfile.dev -t my-app:dev .
docker build -f my-app.dockerfile -t my-app:prod .
```

### 🎯 **The FROM Command: Your Starting Point**

Every Dockerfile **must** start with a `FROM` command - it's required!

```dockerfile
# Using a specific Linux distribution
FROM debian:jessie

# Using a minimal distribution (recommended)
FROM alpine:3.18

# Using a language-specific base
FROM node:18-alpine
FROM python:3.11-slim
```

**🎯 Purpose:** The `FROM` command specifies the **base image** for your new image.

**💡 Best Practice:** Choose minimal distributions like Alpine or slim variants to keep your final image size small.

</div>

---

## 🏗️ **Page 3: Structure, Layers, and the RUN Command**

<div class="structure-section">

**⚠️ Order Matters:** Docker processes the Dockerfile line-by-line, building up the image sequentially.

### 🧅 **Understanding Layers**

Each instruction in your Dockerfile creates a separate **read-only layer**:

```dockerfile
# Layer 1: Base image (always first)
FROM debian:bookworm-slim   

LABEL maintainer="johnhonai@harihar.nagar"

# Layer 2: Package index update
RUN apt-get update          

# Layer 3: Nginx installation
RUN apt-get install -y nginx  

# Layer 4: Application files
COPY index.html /var/www/html/

# Layer 5: Port configuration
EXPOSE 80

# Layer 6: Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

**🎯 Benefits of Layers:**
- **Caching:** Unchanged layers are reused between builds
- **Efficiency:** Multiple images can share common layers
- **Debugging:** You can inspect each layer individually

### 🏃 **The RUN Command: Executing Inside the Container**

The `RUN` command executes any commands available in the base image:

```dockerfile
FROM debian:jessie

# Since base is Debian, we can use Debian tools
RUN apt-get update
RUN apt-get install -y nginx wget curl
RUN wget https://example.com/config.conf -O /etc/nginx/nginx.conf
```

**🔍 Context:** Because our base image is Debian, `RUN` has access to all Debian binaries like `apt-get`, `wget`, `curl`, etc.

</div>

---

## ⚡ **Page 4: Best Practices and Essential Instructions**

<div class="best-practices-section">

### 🔗 **Layer Optimization: Chain Commands Together**

**❌ Inefficient (creates multiple layers):**
```dockerfile
RUN apt-get update
RUN apt-get install -y nginx
RUN apt-get install -y curl
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*
```

**✅ Optimized (single layer):**
```dockerfile
RUN apt-get update && \
    apt-get install -y nginx curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**🎯 Benefits:**
- Fewer layers = smaller image size
- Faster builds due to better caching
- Self-contained cleanup in the same layer

### 🚪 **The EXPOSE Command: Documentation, Not Action**

```dockerfile
# Document that the app listens on port 80
EXPOSE 80

# Multiple ports
EXPOSE 80 443 8080
```

**🚨 Critical Understanding:**
- `EXPOSE` is **documentation only**
- It does **NOT** automatically open ports to the host
- It tells other developers which ports the app uses

### 🔌 **Actually Opening Ports: The -p Flag**

To make ports accessible from the host, use `-p` when running:

```bash
# Map host port 8080 to container port 80
docker container run -p 8080:80 my-nginx-image

# Map host port 443 to container port 443
docker container run -p 443:443 my-nginx-image

# Map multiple ports
docker container run -p 8080:80 -p 8443:443 my-nginx-image
```

### 📝 **Images -> Containers**

**🚀 Build and run:**
```bash
# Build the image
docker build -t my-nginx .

# Run with port mapping
docker container run -d -p 8080:80 --name web-server my-nginx

# Test it works
curl http://localhost:8080
```

</div>

---

## 📚 **Quick Reference Summary**

| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image (required) | `FROM alpine:3.18` |
| `RUN` | Execute commands during build | `RUN apt-get update && apt-get install -y nginx` |
| `COPY` | Copy files from host to image | `COPY app.js /usr/src/app/` |
| `WORKDIR` | Set working directory | `WORKDIR /usr/src/app` |
| `EXPOSE` | Document ports (metadata only) | `EXPOSE 80 443` |
| `CMD` | Default command when container starts | `CMD ["nginx", "-g", "daemon off;"]` |

**🎯 Remember:** 
- Order matters in Dockerfiles
- Chain related commands to minimize layers
- Always clean up in the same RUN instruction
- Use `-p` flag to actually map ports when running containers

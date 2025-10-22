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

## 📄 **Page 1: Introduction to the Dockerfile**

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
my-app.dockerfile
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

## 🏗️ **Page 2: Structure, Layers, and the RUN Command**

<div class="structure-section">

### 📋 **Dockerfile Structure and Order**

```dockerfile
# 1. Base image (always first)
FROM debian:jessie

# 2. Metadata and configuration
LABEL maintainer="your-email@example.com"

# 3. Install dependencies
RUN apt-get update && apt-get install -y nginx

# 4. Copy application files
COPY . /app

# 5. Set working directory
WORKDIR /app

# 6. Expose ports
EXPOSE 80

# 7. Define startup command
CMD ["nginx", "-g", "daemon off;"]
```

**⚠️ Order Matters:** Docker processes the Dockerfile line-by-line, building up the image sequentially.

### 🧅 **Understanding Layers**

Each instruction in your Dockerfile creates a separate **read-only layer**:

```dockerfile
FROM debian:jessie          # Layer 1: Base OS
RUN apt-get update          # Layer 2: Package index update
RUN apt-get install nginx   # Layer 3: Nginx installation
COPY app.html /var/www/     # Layer 4: Application files
EXPOSE 80                   # Layer 5: Port configuration
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

### 🌐 **Real Example: Installing Nginx**

```dockerfile
FROM debian:jessie

# Add official Nginx repository key
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys ABF5BD827BD9BF62

# Add Nginx repository
RUN echo "deb http://nginx.org/packages/debian/ jessie nginx" > /etc/apt/sources.list.d/nginx.list

# Update package index and install latest Nginx
RUN apt-get update && apt-get install -y nginx

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

</div>

---

## ⚡ **Page 3: Best Practices and Essential Instructions**

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

### 🧹 **Self-Correction and Cleanup Pattern**

Always clean up in the same `RUN` instruction where you install packages:

```dockerfile
RUN apt-get update && \
    apt-get install -y \
        nginx \
        curl \
        wget && \
    # Configure application
    echo "server_name example.com;" > /etc/nginx/conf.d/server.conf && \
    # Clean up package cache
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

### 📊 **Log Forwarding Best Practice**

Point application logs to STDOUT and STDERR so Docker can manage them:

```dockerfile
# Forward Nginx logs to Docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log
```

**🎯 Why:** This allows Docker to capture and manage container logs through `docker container logs`.

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

### 📝 **Complete Example: Nginx Web Server**

```dockerfile
FROM debian:jessie

# Install Nginx with cleanup in single layer
RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Forward logs to Docker
RUN ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log

# Copy custom configuration
COPY nginx.conf /etc/nginx/nginx.conf
COPY index.html /var/www/html/

# Document the port
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

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

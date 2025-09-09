# Build, Ship, Run: The Core of Modern Application Delivery

<div class="workflow-hero">
  <h1>🚀 Build, Ship, Run</h1>
  <div class="workflow-icons">
    <span class="workflow-step">🔨</span>
    <span class="workflow-arrow">→</span>
    <span class="workflow-step">🚢</span>
    <span class="workflow-arrow">→</span>
    <span class="workflow-step">🏃‍♂️</span>
  </div>
  <h2>How Containers Revolutionize Software Development</h2>
  <p class="context-note">
    <strong>Context:</strong> An essential workflow for Cloud-Native environments, 
    championed by the <em>CNCF (Cloud Native Computing Foundation)</em>
  </p>
</div>

---

## 🔨 The "Build" Phase: Creating the Universal App Package

<div class="build-section">

### 📦 Concept: The Container Image

<div class="concept-highlight">
  <h4>🎯 What it is:</h4>
  <p>A <strong>single, universal package</strong> for your application</p>
  <p>Think of it as a <em>self-contained bundle</em></p>
</div>

<div class="cross-platform-grid">
  <div class="platform-feature">
    <h4>🌐 Cross-OS</h4>
    <p>Works on Linux and Windows</p>
  </div>
  <div class="platform-feature">
    <h4>🔧 Cross-Platform</h4>
    <p>x86, ARM, cloud, on-premise</p>
  </div>
  <div class="platform-feature">
    <h4>💻 Language-Agnostic</h4>
    <p>Python, Java, Node.js, Go, etc.</p>
  </div>
</div>

### 📋 What's Inside the Package

<div class="package-contents">
  <div class="content-item">
    <span class="content-icon">💾</span>
    <strong>Application Code</strong>
    <p>Your source code and compiled binaries</p>
  </div>
  <div class="content-item">
    <span class="content-icon">⚙️</span>
    <strong>Runtime Environment</strong>
    <p>Python interpreter, JVM, Node.js, etc.</p>
  </div>
  <div class="content-item">
    <span class="content-icon">🔧</span>
    <strong>System Tools</strong>
    <p>Essential OS utilities and commands</p>
  </div>
  <div class="content-item">
    <span class="content-icon">📚</span>
    <strong>Libraries & Dependencies</strong>
    <p>All required packages and frameworks</p>
  </div>
  <div class="content-item">
    <span class="content-icon">⚙️</span>
    <strong>Configuration</strong>
    <p>Environment variables and settings</p>
  </div>
</div>

<div class="key-takeaway">
  <h4>🎯 Key Takeaway:</h4>
  <p><strong>One package that runs reliably anywhere</strong></p>
</div>

### 💻 Hands-On: Working with Images

Let's see this in action! Here are some basic Docker commands to explore images:

#### 📥 Pulling an Image
```bash
docker pull nginx:latest
```

**Expected Output:**
```
latest: Pulling from library/nginx
a2abf6c4d29d: Pull complete
a9edb18cadd1: Pull complete
589b7251471a: Pull complete
186b1aaa4aa6: Pull complete
b4df32aa5a72: Pull complete
a0bcbecc962e: Pull complete
Digest: sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

#### 📋 Listing Images
```bash
docker images
```

**Expected Output:**
```
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    605c77e624dd   2 weeks ago   141MB
```

#### 🔍 Inspecting an Image
```bash
docker inspect nginx:latest
```

**Expected Output (abbreviated):**
```json
[
    {
        "Id": "sha256:605c77e624dd...",
        "RepoTags": ["nginx:latest"],
        "Created": "2023-12-01T00:00:00.000000000Z",
        "Size": 141234567,
        "Architecture": "amd64",
        "Os": "linux"
    }
]
```

#### 🚀 Running a Container from Image
```bash
docker run -d -p 8080:80 --name my-nginx nginx:latest
```

**Expected Output:**
```
f1e568eed137c4e7c6177f5c2410af999e09d87ab74a5573c917a4cfd75051dc
```

#### 📊 Checking Running Containers
```bash
docker ps
```

**Expected Output:**
```
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                  NAMES
f1e568eed137   nginx:latest   "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp   my-nginx
```

</div>

---

## 📝 The "Build" Blueprint: The Dockerfile

<div class="dockerfile-section">

### 🧾 Concept: The Recipe for Container Images

<div class="dockerfile-concept">
  <p>The <strong>Dockerfile</strong> is the recipe for creating a container image</p>
  <p>A simple text file with a set of <em>top-down instructions</em></p>
</div>

### 🏗️ Essential First Step

<div class="essential-step">
  <h4>📍 Every Dockerfile must start with:</h4>
  <div class="code-example">
    <code>FROM python:3.9</code>
  </div>
  <p>The <strong>FROM</strong> instruction specifies the base image</p>
</div>

### 🔧 Key Instructions (Stanzas)

<div class="dockerfile-instructions">
  <div class="instruction-card">
    <h4><code>FROM</code></h4>
    <p><strong>The starting point or foundation</strong></p>
    <div class="instruction-example">
      <code>FROM node:18-alpine</code>
    </div>
  </div>

  <div class="instruction-card">
    <h4><code>WORKDIR</code></h4>
    <p><strong>Sets the current directory inside the image</strong></p>
    <div class="instruction-example">
      <code>WORKDIR /app</code>
    </div>
  </div>

  <div class="instruction-card">
    <h4><code>COPY</code></h4>
    <p><strong>Adds your application files into the image</strong></p>
    <div class="instruction-example">
      <code>COPY . /app</code>
    </div>
  </div>

  <div class="instruction-card">
    <h4><code>RUN</code></h4>
    <p><strong>Executes commands to install dependencies</strong></p>
    <div class="instruction-example">
      <code>RUN pip install -r requirements.txt</code>
    </div>
  </div>

  <div class="instruction-card">
    <h4><code>CMD</code></h4>
    <p><strong>The default command when container starts</strong></p>
    <div class="instruction-example">
      <code>CMD ["python", "app.py"]</code>
    </div>
  </div>
</div>

### 💻 Hands-On: Creating Your First Dockerfile

Let's create a simple Python web application and containerize it:

#### 📝 Sample Application Files

**app.py:**
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return '<h1>Hello from Docker!</h1>'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**requirements.txt:**
```
Flask==2.3.3
```

#### 🏗️ Complete Dockerfile Example
```dockerfile
# Start with Python base image
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy requirements first (for better caching)
COPY requirements.txt .

# Install dependencies
RUN pip install -r requirements.txt

# Copy application code
COPY app.py .

# Expose port
EXPOSE 5000

# Define startup command
CMD ["python", "app.py"]
```

#### 🔨 Building the Image
```bash
docker build -t my-python-app .
```

**Expected Output:**
```
[+] Building 45.2s (10/10) FINISHED
 => [internal] load build definition from Dockerfile                    0.0s
 => => transferring dockerfile: 234B                                    0.0s
 => [internal] load .dockerignore                                       0.0s
 => => transferring context: 2B                                         0.0s
 => [internal] load metadata for docker.io/library/python:3.9-slim     2.1s
 => [1/5] FROM docker.io/library/python:3.9-slim@sha256:...            15.2s
 => [internal] load build context                                       0.0s
 => => transferring context: 156B                                       0.0s
 => [2/5] WORKDIR /app                                                  0.1s
 => [3/5] COPY requirements.txt .                                       0.0s
 => [4/5] RUN pip install -r requirements.txt                          25.8s
 => [5/5] COPY app.py .                                                 0.0s
 => exporting to image                                                  2.0s
 => => exporting layers                                                 2.0s
 => => writing image sha256:abc123...                                   0.0s
 => => naming to docker.io/library/my-python-app                       0.0s
```

#### 🚀 Running the Container
```bash
docker run -d -p 5000:5000 --name my-app my-python-app
```

**Expected Output:**
```
a1b2c3d4e5f6789012345678901234567890123456789012345678901234567890
```

#### 🌐 Testing the Application
```bash
curl http://localhost:5000
```

**Expected Output:**
```html
<h1>Hello from Docker!</h1>
```

</div>

---

## 🏗️ How the "Build" Works: Understanding Layers

<div class="layers-section">

### 🧱 Concept: Each Instruction Creates a New "Layer"

<div class="layer-process">
  <h4>📋 The Build Process:</h4>
  <ol class="build-steps">
    <li><strong>Starts with the base image</strong> (FROM instruction)</li>
    <li><strong>Each subsequent command</strong> adds a new layer on top</li>
    <li><strong>Each layer contains only the changes</strong> from that command</li>
    <li><strong>New files, installed packages, modifications</strong> are captured</li>
  </ol>
</div>

### ⚡ Efficiency: Layer Caching

<div class="caching-benefits">
  <div class="cache-feature">
    <h4>🚀 Speed</h4>
    <p>Unchanged layers are <strong>reused</strong> in next builds</p>
  </div>
  <div class="cache-feature">
    <h4>💾 Storage</h4>
    <p>Shared layers <strong>save disk space</strong></p>
  </div>
  <div class="cache-feature">
    <h4>🔄 Efficiency</h4>
    <p>Only <strong>modified layers</strong> are rebuilt</p>
  </div>
</div>

### 📦 Portability: Easy Distribution

<div class="portability-info">
  <p><strong>Layers are packaged together</strong> and can be easily moved around as <em>tarballs</em></p>
  <p>This makes sharing and deploying images incredibly efficient</p>
</div>

### 🎯 Layer Visualization

<div class="layer-stack">
  <div class="layer layer-cmd">
    <strong>Layer 4:</strong> CMD ["python", "app.py"]
  </div>
  <div class="layer layer-run">
    <strong>Layer 3:</strong> RUN pip install -r requirements.txt
  </div>
  <div class="layer layer-copy">
    <strong>Layer 2:</strong> COPY . /app
  </div>
  <div class="layer layer-workdir">
    <strong>Layer 1:</strong> WORKDIR /app
  </div>
  <div class="layer layer-from">
    <strong>Base Layer:</strong> FROM python:3.9
  </div>
</div>

### 💻 Hands-On: Exploring Layers

Let's examine how layers work in practice:

#### 🔍 Viewing Image History
```bash
docker history my-python-app
```

**Expected Output:**
```
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
abc123def456   5 minutes ago   CMD ["python" "app.py"]                        0B        buildkit.dockerfile.v0
<missing>      5 minutes ago   COPY app.py . # buildkit                        1.23kB    buildkit.dockerfile.v0
<missing>      5 minutes ago   RUN /bin/sh -c pip install -r requirements.txt 15.2MB    buildkit.dockerfile.v0
<missing>      5 minutes ago   COPY requirements.txt . # buildkit              58B       buildkit.dockerfile.v0
<missing>      5 minutes ago   WORKDIR /app                                    0B        buildkit.dockerfile.v0
<missing>      2 weeks ago     /bin/sh -c #(nop)  CMD ["python3"]             0B
<missing>      2 weeks ago     /bin/sh -c set -eux;  wget -O get-pip.py "$P…  10.5MB
...
```

#### 🧱 Understanding Layer Caching
Let's modify our app and rebuild to see caching in action:

**Modified app.py:**
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return '<h1>Hello from Docker! Version 2.0</h1>'  # Changed this line

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

#### 🔨 Rebuilding with Cache
```bash
docker build -t my-python-app:v2 .
```

**Expected Output (notice the cache usage):**
```
[+] Building 2.1s (10/10) FINISHED
 => [internal] load build definition from Dockerfile                    0.0s
 => => transferring dockerfile: 234B                                    0.0s
 => [internal] load .dockerignore                                       0.0s
 => => transferring context: 2B                                         0.0s
 => [internal] load metadata for docker.io/library/python:3.9-slim     0.0s
 => [1/5] FROM docker.io/library/python:3.9-slim                       0.0s
 => => CACHED                                                           0.0s
 => [internal] load build context                                       0.0s
 => => transferring context: 156B                                       0.0s
 => [2/5] WORKDIR /app                                                  0.0s
 => => CACHED                                                           0.0s
 => [3/5] COPY requirements.txt .                                       0.0s
 => => CACHED                                                           0.0s
 => [4/5] RUN pip install -r requirements.txt                          0.0s
 => => CACHED                                                           0.0s
 => [5/5] COPY app.py .                                                 0.1s
 => exporting to image                                                  2.0s
 => => exporting layers                                                 2.0s
 => => writing image sha256:def789...                                   0.0s
 => => naming to docker.io/library/my-python-app:v2                    0.0s
```

Notice how much faster it is! Only the `COPY app.py .` layer was rebuilt because that's the only file that changed.

#### 📊 Comparing Image Sizes
```bash
docker images my-python-app
```

**Expected Output:**
```
REPOSITORY      TAG       IMAGE ID       CREATED          SIZE
my-python-app   v2        def789abc123   2 minutes ago    126MB
my-python-app   latest    abc123def456   10 minutes ago   126MB
```

Both images share the same base layers, so they don't take up double the space!

</div>

---

### 🧹 Cleanup Commands

After your demonstration, clean up the resources:

#### 🛑 Stop and Remove Containers
```bash
# Stop running containers
docker stop my-nginx my-app

# Remove containers
docker rm my-nginx my-app
```

#### 🗑️ Remove Images
```bash
# Remove our custom images
docker rmi my-python-app:latest my-python-app:v2

# Remove the nginx image
docker rmi nginx:latest
```

#### 🧽 System Cleanup
```bash
# Remove all unused containers, networks, images, and build cache
docker system prune -a
```

**Expected Output:**
```
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all images without at least one container associated to them
  - all build cache

Are you sure you want to continue? [y/N] y
Deleted Containers:
...
Deleted Images:
...
Total reclaimed space: 1.2GB
```

---

<div class="speaker-notes">
  <h4>📝 Speaker Notes</h4>
  <ul>
    <li><strong>Emphasize universality:</strong> One image works everywhere - development, testing, production</li>
    <li><strong>Dockerfile analogy:</strong> Like a recipe that anyone can follow to create the exact same dish</li>
    <li><strong>Layer efficiency:</strong> Compare to Git commits - only changes are stored, not entire copies</li>
    <li><strong>CNCF context:</strong> This workflow is fundamental to cloud-native applications and microservices</li>
    <li><strong>Best practice:</strong> Order Dockerfile instructions from least to most frequently changing for optimal caching</li>
    <li><strong>Hands-on tips:</strong> Encourage audience to follow along with the commands during presentation</li>
    <li><strong>Layer caching demo:</strong> The rebuild demonstration clearly shows the power of Docker's layer caching</li>
    <li><strong>Real-world relevance:</strong> These commands are what developers use daily in their Docker workflow</li>
    <li><strong>Troubleshooting:</strong> If commands fail, check Docker is running and user has proper permissions</li>
  </ul>
</div>

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

</div>

---

<div class="speaker-notes">
  <h4>📝 Speaker Notes</h4>
  <ul>
    <li><strong>Emphasize universality:</strong> One image works everywhere - development, testing, production</li>
    <li><strong>Dockerfile analogy:</strong> Like a recipe that anyone can follow to create the exact same dish</li>
    <li><strong>Layer efficiency:</strong> Compare to Git commits - only changes are stored, not entire copies</li>
    <li><strong>CNCF context:</strong> This workflow is fundamental to cloud-native applications and microservices</li>
    <li><strong>Best practice:</strong> Order Dockerfile instructions from least to most frequently changing for optimal caching</li>
  </ul>
</div>

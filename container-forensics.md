# 🕵️ Docker Container Forensics: Peeking Inside the Black Box

> **Hands-On Investigation:** Master the essential commands to monitor, configure, and troubleshoot what's happening *inside* your running containers

---

## 🔄 **Prerequisites & Quick Review**

<div class="prerequisites-section">

### ✅ **What You Should Know**
- Basic container operations: `docker run <image>`
- Container lifecycle management: start, stop, remove

### 🔍 **External Investigation Tools (Quick Recap)**
Before we go inside, remember these external tools:

```bash
# See running containers
docker container ls

# View container logs
docker container logs <container_name>

# Inspect configuration
docker container inspect <container_name>
```

</div>

---

## 🎯 **What You'll Learn**

By the end of this guide, you'll confidently:
- ✅ Launch containers for inspection and monitoring
- ✅ View active processes running inside containers
- ✅ Inspect complete container configuration and metadata
- ✅ Monitor live performance metrics and resource usage
- ✅ Debug containers using logs and process inspection

---

## 🚀 **Setup: Getting Our Investigation Targets Running**

<div class="setup-section">

Before we dive into forensics, let's launch the two containers we'll be examining. You should have two terminals open for this session: one for running the commands and one for viewing the live statistics.

### 🌐 **Step 1: Launch Nginx (Web Server)**

This is a simple, standard web server that usually runs two processes.

```bash
docker container run -d --name nginx-web nginx
```

**What this does:**
- **`-d`**: Runs in detached mode (background)
- **`--name nginx-web`**: Gives the container a memorable name
- **`nginx`**: Uses the official Nginx image

### 🗄️ **Step 2: Launch MySQL (Database Server)**

This container requires an environment variable to set the root password. We'll use a specific option to make it generate a **random one** on first boot.

```bash
docker container run -d --name mysql-db -e MYSQL_RANDOM_ROOT_PASSWORD=true mysql
```

**What this does:**
- **`-e MYSQL_RANDOM_ROOT_PASSWORD=true`**: Tells MySQL to generate a random root password
- **`--name mysql-db`**: Names the container for easy reference

### ✅ **Step 3: Verify Running Containers**

Always check your work! We should see both containers running.

```bash
docker container ls
```

**Expected output:**
```
CONTAINER ID   IMAGE   COMMAND                  CREATED         STATUS         PORTS     NAMES
a1b2c3d4e5f6   nginx   "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   80/tcp    nginx-web
b2c3d4e5f6a1   mysql   "docker-entrypoint.s…"   1 minute ago    Up 1 minute    3306/tcp  mysql-db
```

</div>

---

## 🔎 **Tool 1: `docker container top` - Process Inspector**

<div class="tool-section">

### 🎯 **Purpose: View Active Processes**

Think of this as running the Linux `ps aux` or `top` command specifically on the container's isolated OS environment. It's the simplest way to confirm the main application is operational.

### 💻 **Hands-On: Inspecting Processes**

#### 🌐 **Nginx Process Inspection**
```bash
docker container top nginx-web
```

**Expected output:**
```
PID    USER   TIME     COMMAND
1234   root   0:00     nginx: master process nginx -g daemon off;
1235   nginx  0:00     nginx: worker process
```

**🔍 What you're seeing:**
- **Master process**: Controls the worker processes and handles configuration
- **Worker process(es)**: Handle actual HTTP requests efficiently

#### 🗄️ **MySQL Process Inspection**
```bash
docker container top mysql-db
```

**Expected output:**
```
PID    USER   TIME     COMMAND
2345   mysql  0:02     mysqld
```

**🔍 What you're seeing:**
- **mysqld**: The main MySQL daemon process
- **Authentication plugin**: Shows the security configuration being used

### 🎯 **Key Insights**

<div class="insights-grid">
  <div class="insight-card">
    <h4>✅ Health Check</h4>
    <p>Quickly verify the main application is running</p>
  </div>
  <div class="insight-card">
    <h4>🔧 Architecture</h4>
    <p>Understand how the application structures its processes</p>
  </div>
  <div class="insight-card">
    <h4>🐛 Debugging</h4>
    <p>Spot unexpected processes or missing services</p>
  </div>
</div>

</div>

---

## 🧠 **Tool 2: `docker container inspect` - Deep Configuration Dive**

<div class="tool-section">

### 🎯 **Purpose: Complete Configuration and Metadata**

This command dumps a massive **JSON array** containing every detail about the container—how it was started, its network settings, environmental variables, resource limits, and more. This is your go-to for deep configuration checks and debugging.

### 💻 **Hands-On: Deep Container Inspection**

#### 🔍 **Full Container Inspection**
```bash
docker container inspect mysql-db
```

**Expected output (abbreviated):**
```json
[
    {
        "Id": "b2c3d4e5f6a1...",
        "Created": "2024-01-15T10:30:00.000000000Z",
        "Path": "docker-entrypoint.sh",
        "Args": ["mysqld"],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 2345,
            "StartedAt": "2024-01-15T10:30:01.000000000Z"
        },
        "Config": {
            "Env": [
                "MYSQL_RANDOM_ROOT_PASSWORD=true",
                "MYSQL_MAJOR=8.0",
                "MYSQL_VERSION=8.0.35"
            ]
        },
        "NetworkSettings": {
            "IPAddress": "172.17.0.3",
            "Gateway": "172.17.0.1"
        }
    }
]
```

### 🎯 **Key Data Points to Hunt For**

<div class="data-points-grid">
  <div class="data-point">
    <h4>🌍 Environment Variables</h4>
    <p>Search for the <strong>"Env"</strong> section</p>
    <code>"MYSQL_RANDOM_ROOT_PASSWORD=true"</code>
  </div>
  <div class="data-point">
    <h4>🌐 Network Configuration</h4>
    <p>Find the <strong>"IPAddress"</strong> field</p>
    <code>"IPAddress": "172.17.0.3"</code>
  </div>
  <div class="data-point">
    <h4>🚀 Startup Command</h4>
    <p>Look for <strong>"Path"</strong> and <strong>"Args"</strong></p>
    <code>"Path": "docker-entrypoint.sh"</code>
  </div>
  <div class="data-point">
    <h4>📊 Container State</h4>
    <p>Check the <strong>"State"</strong> section</p>
    <code>"Status": "running"</code>
  </div>
</div>

### 🔧 **Filtering Specific Information**

For easier reading, you can filter specific sections:

```bash
# Get just the environment variables
docker container inspect mysql-db --format='{{.Config.Env}}'

# Get just the IP address
docker container inspect mysql-db --format='{{.NetworkSettings.IPAddress}}'

# Get the container state
docker container inspect mysql-db --format='{{.State.Status}}'
```

</div>

---

## 📈 **Tool 3: `docker container stats` - Live Performance Monitor**

<div class="tool-section">

### 🎯 **Purpose: Real-Time Performance Metrics**

This is your real-time **Task Manager** for Docker. It provides a dynamic view of resource usage, allowing you to quickly spot containers that are hogging CPU or memory.

### 💻 **Hands-On: Live Performance Monitoring**

#### 📊 **Monitor All Running Containers**
```bash
docker container stats
```

**Expected output (live updating):**
```
CONTAINER ID   NAME        CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS
a1b2c3d4e5f6   nginx-web   0.00%     3.2MiB / 7.775GiB     0.04%     1.23kB / 648B     0B / 0B           2
b2c3d4e5f6a1   mysql-db    0.15%     372.1MiB / 7.775GiB   4.68%     648B / 1.23kB     45.1MB / 12.3MB   37
```

#### 🎯 **Monitor Specific Container**
```bash
# Monitor just the MySQL container
docker container stats mysql-db

# Monitor with no streaming (single snapshot)
docker container stats --no-stream
```

### 📊 **Understanding the Metrics**

<div class="metrics-grid">
  <div class="metric-card">
    <h4>🖥️ CPU %</h4>
    <p>Percentage of host CPU currently being used</p>
    <div class="metric-example">
      <strong>Normal:</strong> 0-5% for idle services<br>
      <strong>High:</strong> 50%+ indicates heavy processing
    </div>
  </div>
  <div class="metric-card">
    <h4>💾 MEM USAGE / LIMIT</h4>
    <p>RAM consumed vs. total allocated</p>
    <div class="metric-example">
      <strong>Nginx:</strong> ~3MB (very lightweight)<br>
      <strong>MySQL:</strong> ~370MB (database cache)
    </div>
  </div>
  <div class="metric-card">
    <h4>🌐 NET I/O</h4>
    <p>Network traffic input/output</p>
    <div class="metric-example">
      <strong>Format:</strong> Received / Sent<br>
      <strong>Spikes:</strong> Indicate active connections
    </div>
  </div>
  <div class="metric-card">
    <h4>💿 BLOCK I/O</h4>
    <p>Disk read/write activity</p>
    <div class="metric-example">
      <strong>MySQL:</strong> High during startup<br>
      <strong>Nginx:</strong> Minimal for static content
    </div>
  </div>
</div>

### 💡 **Performance Insights**

<div class="performance-insights">
  <div class="insight-highlight">
    <h4>🚀 MySQL Startup Behavior</h4>
    <p>When MySQL first starts, you'll observe:</p>
    <ul>
      <li><strong>Memory spike:</strong> Loading database engine and cache</li>
      <li><strong>High BLOCK I/O:</strong> Reading configuration and data files</li>
      <li><strong>CPU activity:</strong> Initializing database structures</li>
    </ul>
    <p><em>This is completely normal and expected!</em></p>
  </div>
</div>

### 🔧 **Advanced Monitoring**

```bash
# Format output for easier parsing
docker container stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Monitor and log to file
docker container stats --no-stream > container_performance.log
```

</div>

---


## 📝 **Forensics Toolkit Summary**

<div class="summary-section">

You've successfully mastered the three pillars of Docker container forensics!

### 🛠️ **The Essential Toolkit**

<div class="toolkit-grid">
  <div class="tool-summary">
    <h4>🔎 <code>docker container top</code></h4>
    <p><strong>What's running?</strong> (Process list)</p>
    <div class="use-case">Quick health check to see if the service is alive</div>
  </div>
  <div class="tool-summary">
    <h4>🧠 <code>docker container inspect</code></h4>
    <p><strong>How was it configured?</strong> (Metadata)</p>
    <div class="use-case">Deep debugging, network settings, startup environment</div>
  </div>
  <div class="tool-summary">
    <h4>📈 <code>docker container stats</code></h4>
    <p><strong>How is it performing?</strong> (Live metrics)</p>
    <div class="use-case">Real-time monitoring and resource bottleneck detection</div>
  </div>
</div>

### 🎯 **When to Use Each Tool**

| Scenario | Recommended Tool | Why |
|----------|------------------|-----|
| **Service won't start** | `top` → `inspect` | Check if processes are running, then examine configuration |
| **Performance issues** | `stats` | Monitor CPU, memory, and I/O in real-time |
| **Network connectivity** | `inspect` | Get IP addresses and network configuration |
| **Environment debugging** | `inspect` | Check environment variables and startup commands |
| **Resource planning** | `stats` | Understand actual resource consumption patterns |

</div>

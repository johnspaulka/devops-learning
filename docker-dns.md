# 💻 Docker Networking and DNS: Naming Your Containers

> **Session Goal:** Understand why DNS/naming is crucial in a dynamic container environment and master container name resolution.

---

## 🎯 Session Objectives

By the end of this session, freshers will be able to:

1. Understand why **DNS/naming** is crucial in a dynamic container environment.
2. Differentiate how **Default Bridge** and **Custom Networks** handle DNS resolution.
3. Use the `docker network` and `docker container` commands to explore networking features.
4. Implement proper container communication using names instead of IP addresses.

---

## 1. 💡 The Core Problem: Why Can't We Trust IP Addresses?

### The Dynamic Container Challenge

**Analogy:** Think of an IP address like a **temporary apartment number** in a massive, busy city (your Docker environment).

**The Issue:** Containers are **dynamic**. They are constantly launched, stopped, moved, and restarted.

- If you stop a container and restart it (or if it fails and Docker restarts it), it might get a **completely different IP address**.
- If you configure one container (e.g., a PHP app) to talk to another container (e.g., a MySQL database) using its IP, that connection will **break** the moment the database container restarts.

**The Solution:** We need a **permanent name** that is consistent regardless of the underlying IP address. This is where **DNS Naming** comes in.
- Docker uses the **container name** as the hostname/DNS entry.

### 🛠️ Hands-On Demo: The IP Address Problem

Let's see this problem in action:

```bash
# Start a container and note its IP
docker run -d --name web-server nginx
docker inspect -f '{{.NetworkSettings.IPAddress}}' web-server
# Output: 172.17.0.2

# Stop and restart the container
docker stop web-server
docker start web-server
docker inspect -f '{{.NetworkSettings.IPAddress}}' web-server
# Output: 172.17.0.2 (might be the same, but could change)

# Remove and recreate the container
docker rm -f web-server
docker run -d --name web-server nginx
docker inspect -f '{{.NetworkSettings.IPAddress}}' web-server
# Output: 172.17.0.3 (likely different!)
```

> **🎯 Crucial Takeaway:** In a dynamic microservices environment, **rely on names, not IPs.**

---

## 2. 🌉 DNS in Default vs. Custom Networks

This is the most critical difference to understand. Docker handles naming differently based on the network type.

| Feature | Default `bridge` Network | Custom User-Defined Network |
| :--- | :--- | :--- |
| **DNS Resolution** | **NO** (Containers cannot find each other by name). | **YES** (Automatic resolution by container name). |
| **Isolation** | Low (All containers are on the same network). | High (Isolated environment). |
| **Recommended Use** | Testing/Single-container scenarios. | **Production**, Multi-container applications. |

### 🛠️ Hands-On Part A: Default Bridge Network (Failure Case)

Let's see the failure state first.

| Step | Command to Run | Expected Output/Notes |
| :--- | :--- | :--- |
| **1. Run Container 1** | `docker container run -d --name web_one nginx` | Should start the Nginx container in the background. |
| **2. Run Container 2** | `docker container run -it --name client_two busybox sh` | Starts a shell in a lightweight **busybox** container. |
| **3. Test DNS** | `ping web_one` (inside `client_two` shell) | **FAIL:** `ping: bad address 'web_one'` |
| **4. Check /etc/hosts** | `cat /etc/hosts` (inside `client_two` shell) | No entry for `web_one` |
| **5. Exit and Clean Up** | `exit`, then `docker container rm -f web_one client_two` | Removes both containers. |

**Expected Terminal Session:**
```bash
$ docker container run -d --name web_one nginx
abc123def456...

$ docker container run -it --name client_two busybox sh
/ # ping web_one
ping: bad address 'web_one'
/ # cat /etc/hosts
127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
fe00::0         ip6-localnet
ff00::0         ip6-mcastprefix
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters
172.17.0.3      client_two
/ # exit

$ docker container rm -f web_one client_two
```

### 🛠️ Hands-On Part B: Custom Network (Success Case)

Now, let's create a proper network.

| Step | Command to Run | Expected Output/Notes |
| :--- | :--- | :--- |
| **1. Create Network** | `docker network create my-app-net` | A new **custom bridge network** is created. |
| **2. Run Container 1** | `docker container run -d --net my-app-net --name web_db redis` | Run a **Redis** (as an example backend) on the custom network. |
| **3. Run Container 2** | `docker container run -it --net my-app-net --name web_app busybox sh` | Starts the **busybox** container on the *same* custom network. |
| **4. Test DNS** | `ping web_db` (inside `web_app` shell) | **SUCCESS!** Ping will start, showing the IP address resolution. |
| **5. Test nslookup** | `nslookup web_db` (inside `web_app` shell) | Shows DNS resolution details. |
| **6. Exit and Clean Up** | `exit`, then cleanup commands | Remove all containers and the network. |

**Expected Terminal Session:**
```bash
$ docker network create my-app-net
def456ghi789...

$ docker container run -d --net my-app-net --name web_db redis
ghi789jkl012...

$ docker container run -it --net my-app-net --name web_app busybox sh
/ # ping -c 3 web_db
PING web_db (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.123 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.098 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.087 ms

/ # nslookup web_db
Server:         127.0.0.11
Address:        127.0.0.11:53

Name:   web_db
Address: 172.18.0.2

/ # exit

$ docker container rm -f web_db web_app
$ docker network rm my-app-net
```

**🎯 Key Insight:** The **custom network** provides built-in DNS through Docker's embedded DNS server at `127.0.0.11`!

---

## 3. 🔗 The `--link` Option: Bridging the Default Gap (Legacy)

### Understanding Docker Links

- **The Problem:** What if you *must* use the default bridge network, but still need DNS?
- **The Answer (Legacy):** Docker provides the `--link` option. This option manually injects the linked container's name and IP address into the **hosts file** (`/etc/hosts`) of the new container.
- **Note:** This is considered a **legacy feature**. It only works one way (Container B links to Container A, but Container A cannot find B). **Custom networks** are the modern, recommended solution.

### 🛠️ Hands-On Part C: Using `--link`

| Step | Command to Run | Expected Output/Notes |
| :--- | :--- | :--- |
| **1. Run Container 1** | `docker container run -d --name target_db redis` | Run the target container on the default bridge network. |
| **2. Run Container 2 (with link)** | `docker container run -it --name app_client --link target_db:database_name busybox sh` | Note the format: `--link <target_name>:<alias>`. The alias is what you will use for pinging. |
| **3. Test DNS** | `ping database_name` (inside `app_client` shell) | **SUCCESS!** The link added a DNS entry to `/etc/hosts`. |
| **4. Check /etc/hosts** | `cat /etc/hosts` (inside `app_client` shell) | Shows the manual DNS entry. |
| **5. Exit and Clean Up** | `exit`, then `docker container rm -f target_db app_client` | Remove all containers. |

**Expected Terminal Session:**
```bash
$ docker container run -d --name target_db redis
jkl012mno345...

$ docker container run -it --name app_client --link target_db:database_name busybox sh
/ # ping -c 2 database_name
PING database_name (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.156 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.098 ms

/ # cat /etc/hosts
127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
fe00::0         ip6-localnet
ff00::0         ip6-mcastprefix
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters
172.17.0.2      database_name target_db
172.17.0.3      app_client

/ # exit

$ docker container rm -f target_db app_client
```

**🎯 Notice:** The `/etc/hosts` file contains the manual entry for `database_name` pointing to the Redis container's IP.

---

## 4. Advanced DNS Features

### 4.1 Docker's Embedded DNS Server

Docker runs an embedded DNS server at `127.0.0.11:53` in custom networks:

```bash
# Inside a container on a custom network
/ # cat /etc/resolv.conf
nameserver 127.0.0.11
options ndots:0
```

### 4.2 DNS Round Robin (Load Balancing)

Multiple containers with the same network alias can provide load balancing:

```bash
# Create network
docker network create lb-net

# Run multiple web servers with same alias
docker run -d --net lb-net --net-alias web nginx
docker run -d --net lb-net --net-alias web nginx
docker run -d --net lb-net --net-alias web nginx

# Test from client - will round-robin between the three containers
docker run -it --net lb-net busybox sh
/ # nslookup web
# Shows multiple IP addresses
```

### 4.3 Custom DNS Configuration

```bash
# Custom DNS servers
docker run -it --dns=8.8.8.8 --dns=1.1.1.1 busybox sh

# Custom DNS search domains
docker run -it --dns-search=example.com busybox sh

# Add extra hosts entries
docker run -it --add-host=myhost:192.168.1.100 busybox sh
```

---

## 5. Best Practices and Troubleshooting

### 🎯 DNS Best Practices

| Practice | Reason | Example |
| :--- | :--- | :--- |
| **Use Custom Networks** | Automatic DNS resolution | `docker network create app-net` |
| **Meaningful Container Names** | Easy identification and debugging | `--name web-frontend` vs `--name container1` |
| **Network Aliases** | Multiple names for same service | `--net-alias api --net-alias backend` |
| **Avoid --link** | Legacy feature, use custom networks | Use custom networks instead |

### 🔧 DNS Troubleshooting Commands

```bash
# Check DNS resolution inside container
docker exec container nslookup target-container

# Check /etc/hosts file
docker exec container cat /etc/hosts

# Check DNS configuration
docker exec container cat /etc/resolv.conf

# Test connectivity
docker exec container ping target-container

# Check network configuration
docker network inspect network-name
```

### Common DNS Issues and Solutions

| Issue | Cause | Solution |
| :--- | :--- | :--- |
| **Name not resolving** | Using default bridge network | Create and use custom network |
| **Connection refused** | Service not running on expected port | Check container logs and port configuration |
| **Intermittent resolution** | DNS caching issues | Restart containers or use `--dns` flag |

---

## 📝 Summary and Key Takeaways

### 🎯 Recap

- **Container Names** are more reliable than **IP Addresses** because IPs are volatile in dynamic environments.
- **Custom Networks** automatically handle DNS resolution (the modern and best way).
- The **Default Bridge** network does **not** provide automatic DNS.
- The **`--link`** option is a **legacy** workaround for the default bridge network.
- Docker's embedded DNS server (`127.0.0.11`) handles name resolution in custom networks.

### Decision Matrix

| Scenario | Recommended Approach | Command Example |
| :--- | :--- | :--- |
| **Single Container** | Default bridge is fine | `docker run -d nginx` |
| **Multi-Container App** | Custom network | `docker network create app-net` |
| **Legacy System** | Use --link (not recommended) | `docker run --link db:database app` |
| **Load Balancing** | Network aliases | `docker run --net-alias web nginx` |

### 🚀 Next Steps

In the next session, we'll dive deeper into:
- Advanced Docker networking scenarios
- Port mapping and network drivers
- Docker Compose networking
- Production networking patterns

---

*Master container naming, and your microservices will thank you! 🐳*

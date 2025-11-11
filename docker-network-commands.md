# 🐳 Docker Network Management: The Command Line Deep Dive

> **Purpose:** Practical, hands-on management of Docker networks using the Command Line Interface (CLI). We will treat container networking like managing **virtual network cards** in a running machine.

---

## 1. Network Visibility: Inspecting the Landscape

Before creating anything new, we need to know what networks already exist on our Docker Host.

### 1.1 `docker network ls` (List)

This command is your first look at the networks created by Docker and any custom networks you or others have added.

| Command | Description |
| :--- | :--- |
| `docker network ls` | Lists all Docker networks on the host. |

**🛠️ Hands-on Demo: List Default Networks**

Try this command to see the three main default networks:

```bash
docker network ls
```

**Expected Output & Key Concepts:**

```
NETWORK ID     NAME      DRIVER    SCOPE
abc123def456   bridge    bridge    local
ghi789jkl012   host      host      local
mno345pqr678   none      null      local
```

| NAME | DRIVER | SCOPE | Concept |
| :--- | :--- | :--- | :--- |
| **bridge** | `bridge` | `local` | **Default network**. Used by containers unless a specific network is chosen. It provides NAT and access to the outside world. |
| **host** | `host` | `local` | **No isolation**. The container shares the host's network stack directly. |
| **none** | `null` | `local` | **No networking**. The container is completely isolated. |

### 1.2 `docker network inspect` (Detail)

The `inspect` command is crucial for seeing the detailed configuration of a network, including the containers currently attached to it, their IPs, and the subnet.

| Command | Description |
| :--- | :--- |
| `docker network inspect <NETWORK_NAME>` | Shows a detailed JSON output of the network's configuration. |

**🛠️ Hands-on Demo: Inspect the Default Bridge**

1. Run a simple container:
   ```bash
   docker run -d --name webhost_orig nginx:alpine
   ```

2. Now inspect the default `bridge` network to see your container:
   ```bash
   docker network inspect bridge
   ```

**Points to Note:**

- Look for the **`Containers`** section in the JSON output. Your `webhost_orig` container will be listed, along with its automatically assigned IP address (likely in the **172.17.x.x** range).
- Look for the **`IPAM`** (IP Address Management) section, which defines the network's **Subnet** and **Gateway**.

**Example Output (Key Sections):**
```json
{
    "IPAM": {
        "Config": [
            {
                "Subnet": "172.17.0.0/16",
                "Gateway": "172.17.0.1"
            }
        ]
    },
    "Containers": {
        "abc123...": {
            "Name": "webhost_orig",
            "IPv4Address": "172.17.0.2/16"
        }
    }
}
```

---

## 2. Network Creation: Building Custom Networks

Custom networks are a **best practice**! They isolate your applications from unrelated containers and enable automatic service discovery (containers can talk to each other by name, not just by IP).

### 2.1 `docker network create`

This command creates a new, **user-defined** bridge network by default.

| Command | Description |
| :--- | :--- |
| `docker network create <NETWORK_NAME>` | Creates a new network using the default `bridge` driver. |
| `docker network create --driver <DRIVER> <NETWORK_NAME>` | Creates a network using a specific driver (e.g., `overlay` for Swarm). |

**🛠️ Hands-on Demo: Create a New App Network**

Let's create a dedicated network for our sample application:

```bash
docker network create my_app_net
```

**Verification:**

```bash
docker network ls
```

You should now see `my_app_net` with the `bridge` driver.

**Expected Output:**
```
NETWORK ID     NAME         DRIVER    SCOPE
abc123def456   bridge       bridge    local
ghi789jkl012   host         host      local
mno345pqr678   none         null      local
stu901vwx234   my_app_net   bridge    local
```

### 2.2 Advanced Network Creation Options

| Command | Description | Use Case |
| :--- | :--- | :--- |
| `docker network create --subnet=192.168.1.0/24 mynet` | Create network with specific subnet | Control IP addressing |
| `docker network create --gateway=192.168.1.1 mynet` | Set custom gateway | Network routing control |
| `docker network create --driver=host mynet` | Use host networking | Performance-critical apps |

**Key Difference (Custom vs. Default Bridge):**

| Feature | Default `bridge` (`docker0`) | User-Defined `bridge` (`my_app_net`) |
| :--- | :--- | :--- | :--- |
| **Name Resolution** | Containers **cannot** talk by name. Must use IP. | Containers **can** talk by name (e.g., `web` can talk to `db`). |
| **Isolation** | All containers attach here by default, leading to less isolation. | Only containers you specify attach here, providing better isolation. |

---

## 3. Network Connection: The Virtual NIC

The core of flexible Docker networking is the ability to connect containers to networks at creation time or even when they are running live.

### 3.1 Connecting at Creation (`docker run --network`)

The easiest way to connect a new container is at the moment you run it.

| Command | Description |
| :--- | :--- |
| `docker run -d --name <NAME> --network <NET> <IMAGE>` | Starts a container and attaches it to the specified network. |

**🛠️ Hands-on Demo: Connect New Container**

1. Create a second container and connect it to your new network:
   ```bash
   docker run -d --name new_web_host --network my_app_net nginx:alpine
   ```

2. Verify the connection:
   ```bash
   docker network inspect my_app_net
   ```
   *You should see `new_web_host` listed under containers.*

### 3.2 Connecting a Running Container (`docker network connect`)

This command simulates "plugging in a new network card" to a running container. The container will instantly be connected to the new network.

| Command | Description |
| :--- | :--- |
| `docker network connect <NETWORK_NAME> <CONTAINER_NAME>` | Connects a **running** container to an existing network. |

**🛠️ Hands-on Demo: Connect Live Container**

1. Connect your original container (`webhost_orig`) to the new network:
   ```bash
   docker network connect my_app_net webhost_orig
   ```

2. **Verify Dual-Homing:**
   ```bash
   docker inspect webhost_orig
   ```
   *Search the output for the **`Networks`** section. You will see **two** entries: `bridge` and `my_app_net`, each with its own IP address!*

**Example Dual-Homed Container:**
```json
"Networks": {
    "bridge": {
        "IPAddress": "172.17.0.2"
    },
    "my_app_net": {
        "IPAddress": "172.18.0.3"
    }
}
```

### 3.3 Disconnecting a Running Container (`docker network disconnect`)

To remove the container from a network, you use the `disconnect` command. This is like "unplugging a network card."

| Command | Description |
| :--- | :--- |
| `docker network disconnect <NETWORK_NAME> <CONTAINER_NAME>` | Disconnects a **running** container from a network. |

**🛠️ Hands-on Demo: Disconnect Live Container**

1. Disconnect the `webhost_orig` from the custom network:
   ```bash
   docker network disconnect my_app_net webhost_orig
   ```

2. **Verification:**
   ```bash
   docker inspect webhost_orig
   ```
   *The `my_app_net` entry should be gone, and only the `bridge` network remains.*

---

## 4. Advanced Network Commands

### 4.1 Network Filtering and Formatting

| Command | Description | Use Case |
| :--- | :--- | :--- |
| `docker network ls --filter driver=bridge` | List only bridge networks | Find specific network types |
| `docker network ls --format "table {{.Name}}\t{{.Driver}}"` | Custom output format | Clean, readable output |
| `docker network inspect --format='{{.IPAM.Config}}' mynet` | Extract specific info | Scripting and automation |

### 4.2 Container Network Information

| Command | Description | Output |
| :--- | :--- | :--- |
| `docker inspect -f '{{.NetworkSettings.IPAddress}}' container` | Get container IP | `172.17.0.2` |
| `docker inspect -f '{{.NetworkSettings.Networks}}' container` | All network connections | Full network details |
| `docker exec container hostname -i` | IP from inside container | Container's perspective |

### 4.3 Network Troubleshooting Commands

```bash
# Test connectivity between containers
docker exec container1 ping container2

# Check DNS resolution
docker exec container nslookup container2

# View network interfaces inside container
docker exec container ip addr show

# Check routing table
docker exec container ip route show

# Test port connectivity
docker exec container telnet container2 80
```

---

## 5. Cleanup and Best Practices 🧹

### 5.1 Network Cleanup

When you are finished with your custom network, you can remove it.

| Command | Description |
| :--- | :--- |
| `docker network rm <NETWORK_NAME>` | Removes a network (only works if no containers are attached). |
| `docker network prune` | Removes all unused networks. |

**🛠️ Hands-on Demo: Cleanup**

1. Stop and remove the containers:
   ```bash
   docker rm -f webhost_orig new_web_host
   ```

2. Remove the custom network:
   ```bash
   docker network rm my_app_net
   ```

3. Clean up all unused networks:
   ```bash
   docker network prune
   ```

### 5.2 Key Security & Design Takeaway

| **Concept** | **Analogy** | **Benefit** |
| :--- | :--- | :--- |
| **User-Defined Networks** | A private, isolated **local area network (LAN)** switch for your application team. | **Security:** Only containers on that network can talk to each other. **DNS/Discovery:** Containers can find each other by name (e.g., `web` container talks to `database`). |
| **Connecting/Disconnecting** | Hot-swapping a **network card**. | **Flexibility:** Change a container's communication path without downtime or a restart. |
| **Port Exposure (`-p`)** | Opening a single, locked **door** in the LAN firewall. | **Security:** You only expose the **minimum necessary port** to the outside host, keeping all other container ports protected behind the virtual network's firewall. |

---

## 📚 Quick Reference

### Essential Network Commands

```bash
# List networks
docker network ls

# Create network
docker network create mynet

# Inspect network
docker network inspect mynet

# Connect container to network
docker network connect mynet container

# Disconnect container from network
docker network disconnect mynet container

# Remove network
docker network rm mynet

# Clean up unused networks
docker network prune
```

### Network Drivers

| Driver | Use Case | Scope |
| :--- | :--- | :--- |
| **bridge** | Single-host networking | local |
| **host** | No isolation, use host network | local |
| **none** | No networking | local |
| **overlay** | Multi-host networking (Swarm) | swarm |
| **macvlan** | Assign MAC addresses to containers | local |

---

*Master these commands, and you'll have complete control over Docker networking! 🐳*

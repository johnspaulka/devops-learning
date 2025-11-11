# 🌐 Docker Networking: Concepts & Hands-On

---

## 1. Networking Refresher 🌐

Before diving into Docker, let's quickly align on key networking terms.

| Concept | Description | Analogy |
| :--- | :--- | :--- |
| **IP Address** | A unique address for a device on a network (e.g., `192.168.1.5`) | Your **House Address**. |
| **Subnet** | A range of IPs on the same local network (e.g., `192.168.1.0/24`) | Your **Neighborhood/Street**. |
| **Port** | A specific application-level endpoint on an IP (e.g., `80` for web, `3306` for MySQL) | A specific **Room or Apartment Number** in your house. |
| **Firewall (NAT)** | A security system that controls traffic and often translates private IPs to a public IP. | A **Gatekeeper/Mailroom** that changes the internal house numbers to the public street address. |

**🎯 Key Takeaway:** A container needs an **IP** and its services run on a **Port**.

---

## 2. The `-p` Flag: Exposing Services 🚪

The `-p` flag in `docker run` is how you expose a container's internal port to your host machine's external network.

- **Syntax:** `-p <Host_Port>:<Container_Port>`

| Parameter | What is it? | Example |
| :--- | :--- | :--- |
| **Host Port** | The port on your **laptop** (host OS) that receives external traffic. | `8080` |
| **Container Port** | The port the application inside the container is **actually listening on**. | `80` (for Nginx) |

### 🛠️ Hands-On Practice: Exposing a Web Server

#### Step 1: Run Nginx (without exposing the port)

```bash
docker run -d --name webhost-a nginx
```

**🔍 Observation:** You can't access it via your browser at `http://localhost`.

#### Step 2: Run Nginx (with port exposure)

```bash
docker run -d --name webhost-b -p 8080:80 nginx
```

**🔍 Observation:** You can now access it via your browser at `http://localhost:8080`.

#### Step 3: Verify the Port Mapping

```bash
docker container port webhost-b
```

**Expected Output:**
```
80/tcp -> 0.0.0.0:8080
```

### 🎯 Understanding the Flow

```
Internet → Your Laptop:8080 → Docker Bridge → Container:80 → Nginx
```

---

## 3. Docker's Default Network: Bridge 🌉

When you don't specify a network, Docker automatically connects your containers to the **default `bridge` network**.

### How it Works:

1. **Virtual Switch:** Docker creates a virtual network interface on your host machine called **`docker0`**, which acts like a virtual switch (a network bridge).

2. **Subnet & IPs:** Docker gives the bridge a private **Subnet** (e.g., `172.17.0.0/16`) and assigns each container an **IP address** from this range (e.g., `172.17.0.2`, `172.17.0.3`).

3. **NAT & Firewall:** The Docker daemon automatically configures **NAT (Network Address Translation)** rules on the host's firewall (**iptables**).
   - **Outgoing Traffic:** Containers can reach the outside world because the NAT translates their private IP to the host's public IP.
   - **Incoming Traffic:** All incoming traffic from the external world is **blocked by default** unless a port is explicitly opened using the `-p` flag.

### 🛠️ Hands-On Practice: Inspecting the Default Network

#### Step 1: List all networks

Notice the default `bridge` network.

```bash
docker network ls
```

**Expected Output:**
```
NETWORK ID     NAME      DRIVER    SCOPE
abc123def456   bridge    bridge    local
ghi789jkl012   host      host      local
mno345pqr678   none      null      local
```

#### Step 2: Inspect a container's IP (The "Inspect-Filter" Trick)

This shows the container has a different IP than your host.

```bash
# Get the IP address of the container you ran earlier
docker inspect -f '{{.NetworkSettings.IPAddress}}' webhost-a
```

**Expected Output:**
```
172.17.0.2
```

#### Step 3: Test container-to-container communication

```bash
# Run another container
docker run -d --name webhost-c nginx

# Get its IP
docker inspect -f '{{.NetworkSettings.IPAddress}}' webhost-c

# Test ping from one container to another
docker exec -it webhost-a ping -c 3 172.17.0.3
```

**🎯 Isolation:** Containers on the *same* default bridge network **can talk to each other** using their IP addresses, but they **cannot talk to containers on a *different* network**.

---

## 4. User-Defined Networks: Isolation & Discovery 🔗

The default bridge network is simple, but for complex applications (like a web app + database), it's best practice to use a **user-defined bridge network**.

### Advantages of User-Defined Networks:

- **Service Discovery:** Containers can talk to each other using their **container names** (like a built-in DNS), not just their IP addresses.
- **Isolation:** Only related containers are connected to the network, providing a better security boundary.
- **Decoupling:** You can change a container's IP/subnet without affecting others.

### 🛠️ Hands-On Practice: Creating an Isolated App Network

**Goal:** Run a **Web App** (Apache) and a **Database** (MySQL) so they can communicate privately, but only the Web App is exposed to the outside world.

#### Step 1: Create a User-Defined Network

```bash
docker network create my-app-network
```

Use `docker network ls` again to see the new network.

#### Step 2: Run the MySQL Container (Database) - *No `-p` needed*

We only attach it to the custom network and use an environment variable for the root password.

```bash
docker run -d --name my-db --network my-app-network -e MYSQL_ROOT_PASSWORD=mypass mysql:5.7
```

#### Step 3: Run the Web Container (Client) - *Expose only the web port*

We attach it to the same network and expose the web port.

```bash
docker run -d --name my-web --network my-app-network -p 8081:80 httpd
```

#### Step 4: Verify Communication (Service Discovery)

The web container can "ping" the database container using its **container name** (`my-db`).

```bash
docker exec -it my-web ping -c 3 my-db
```

**🔍 Expected Observation:** The ping is **successful**, proving they can communicate by name!

#### Step 5: Test DNS Resolution

```bash
# Check if the web container can resolve the database name
docker exec -it my-web nslookup my-db
```

### 🎯 Network Architecture Comparison

**Default Bridge Network:**
```
Container A (172.17.0.2) ←→ Container B (172.17.0.3)
     ↓                              ↓
         Default Bridge Network
                  ↓
              Host Machine
```

**User-Defined Network:**
```
my-web (my-db resolves to IP) ←→ my-db (my-web resolves to IP)
           ↓                              ↓
              my-app-network
                    ↓
               Host Machine
```

---

## 5. Review & Cleanup 🧹

### Command Reference

| Concept | Command / Flag | Purpose |
| :--- | :--- | :--- |
| **Expose Port** | `-p 8080:80` | Maps Host Port to Container Port. |
| **Inspect Port** | `docker container port <name>` | Shows the port mapping for a container. |
| **List Networks** | `docker network ls` | Displays all networks (default `bridge` and custom). |
| **Create Network** | `docker network create <name>` | Creates a user-defined network for isolation/DNS. |
| **Run on Network** | `--network <name>` | Connects a container to a specific network. |
| **Check IP** | `docker inspect -f '{{.NetworkSettings.IPAddress}}' <name>` | Gets the container's IP address. |
| **Network Details** | `docker network inspect <name>` | Shows detailed network configuration. |

### 🧹 Cleanup Commands

```bash
# Stop all containers
docker container stop webhost-a webhost-b webhost-c my-db my-web

# Remove all containers
docker container rm webhost-a webhost-b webhost-c my-db my-web

# Remove the custom network
docker network rm my-app-network
```

### 🎯 Key Takeaways

1. **Default Bridge:** Good for simple, single-container applications
2. **User-Defined Networks:** Best practice for multi-container applications
3. **Port Mapping:** Use `-p` to expose services to the outside world
4. **Service Discovery:** Container names work as hostnames in user-defined networks
5. **Security:** Only expose ports you need, keep internal communication on custom networks

### 🚀 Next Steps

- Explore Docker Compose for managing multi-container applications
- Learn about other network drivers (host, overlay, macvlan)
- Practice with real applications (WordPress + MySQL, Node.js + Redis)
- Understand Docker Swarm networking for production clusters

---

*Master the network, master the containers! 🐳*

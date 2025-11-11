# 🌐 Networking Fundamentals for Docker

> **Purpose:** Essential networking concepts you need to understand before diving into Docker networking.

---

## 🎯 What You'll Learn

This guide covers the core networking concepts that Docker builds upon:
- **IP Addresses & Subnets** - How devices get unique addresses
- **NAT (Network Address Translation)** - How private networks connect to the internet
- **Ports & Services** - How applications communicate
- **Docker's Network Magic** - How Docker makes it all work together

---

## 1. IP Addresses & Subnets 📍

### IP Addresses: Your Network Identity

An **IP address** is like a postal address for devices on a network.

| Type | Format | Example | Use Case |
| :--- | :--- | :--- | :--- |
| **IPv4** | `xxx.xxx.xxx.xxx` | `192.168.1.100` | Most common, what we'll focus on |
| **IPv6** | `xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx` | `2001:db8::1` | Newer standard, more addresses |

### Private vs Public IP Addresses

| Type | Range | Example | Purpose |
| :--- | :--- | :--- | :--- |
| **Private** | `192.168.x.x`, `10.x.x.x`, `172.16-31.x.x` | `192.168.1.5` | Internal network use only |
| **Public** | Everything else | `8.8.8.8` | Internet-routable addresses |

**🏠 Home Network Example:**
```
Internet (Public IP: 203.0.113.1)
    ↓
Router/Modem
    ↓
Home Network (Private Subnet: 192.168.1.0/24)
    ├── Your Laptop: 192.168.1.100
    ├── Your Phone: 192.168.1.101
    └── Smart TV: 192.168.1.102
```

### Subnets: Organizing IP Addresses

A **subnet** defines a range of IP addresses that belong to the same network segment.

**CIDR Notation:** `192.168.1.0/24`
- `192.168.1.0` = Network address
- `/24` = First 24 bits are the network portion
- Remaining 8 bits = Host portion (256 possible addresses)

| CIDR | Network Mask | Usable IPs | Example Range |
| :--- | :--- | :--- | :--- |
| `/24` | `255.255.255.0` | 254 | `192.168.1.1` - `192.168.1.254` |
| `/16` | `255.255.0.0` | 65,534 | `192.168.0.1` - `192.168.255.254` |
| `/8` | `255.0.0.0` | 16,777,214 | `10.0.0.1` - `10.255.255.254` |

**🎯 Docker Default:** Docker typically uses `172.17.0.0/16` for its default bridge network.

---

## 2. NAT (Network Address Translation) 🔄

### What is NAT?

**NAT** allows multiple devices with private IP addresses to share a single public IP address when accessing the internet.

### How NAT Works

```
Internal Request:
Your Laptop (192.168.1.100:45678) → Router → Internet (203.0.113.1:45678)

Internet Response:
Internet (203.0.113.1:45678) → Router → Your Laptop (192.168.1.100:45678)
```

**NAT Translation Table:**

| Internal IP:Port | External IP:Port | Destination |
| :--- | :--- | :--- |
| `192.168.1.100:45678` | `203.0.113.1:45678` | `8.8.8.8:53` |
| `192.168.1.101:45679` | `203.0.113.1:45679` | `1.1.1.1:53` |

### Types of NAT

| Type | Description | Docker Usage |
| :--- | :--- | :--- |
| **SNAT** (Source NAT) | Changes source IP of outgoing packets | Container → Internet traffic |
| **DNAT** (Destination NAT) | Changes destination IP of incoming packets | Internet → Container traffic (port mapping) |
| **Masquerading** | Dynamic SNAT using the interface's IP | Docker's default outbound NAT |

**🐳 Docker Context:** When you use `-p 8080:80`, Docker creates DNAT rules to forward traffic from host port 8080 to container port 80.

---

## 3. Ports & Services 🚪

### Understanding Ports

**Ports** are numbered endpoints that allow multiple services to run on the same IP address.

| Port Range | Type | Purpose | Examples |
| :--- | :--- | :--- | :--- |
| **1-1023** | Well-known/System | Standard services | 80 (HTTP), 443 (HTTPS), 22 (SSH) |
| **1024-49151** | Registered | Application-specific | 3306 (MySQL), 5432 (PostgreSQL) |
| **49152-65535** | Dynamic/Private | Temporary connections | Client-side connections |

### Common Service Ports

| Service | Port | Protocol | Docker Usage |
| :--- | :--- | :--- | :--- |
| **HTTP** | 80 | TCP | Web servers (nginx, apache) |
| **HTTPS** | 443 | TCP | Secure web servers |
| **SSH** | 22 | TCP | Remote shell access |
| **MySQL** | 3306 | TCP | Database connections |
| **PostgreSQL** | 5432 | TCP | Database connections |
| **Redis** | 6379 | TCP | Cache/database |
| **MongoDB** | 27017 | TCP | NoSQL database |

### Port States

| State | Description | Example |
| :--- | :--- | :--- |
| **LISTEN** | Service waiting for connections | Web server ready for requests |
| **ESTABLISHED** | Active connection | Browser ↔ Web server |
| **CLOSED** | No service on port | Nothing running on port 8080 |

### Checking Port Usage

```bash
# Show listening ports
netstat -tlnp

# Show specific port
netstat -tlnp | grep :80

# Modern alternative
ss -tlnp

# Check if port is open
telnet localhost 8080
```

---

## 4. Docker's Network Magic: How It All Works Together 🐳

### Docker Networking Simplified

Docker takes care of all the complex networking details for you! Here's what happens behind the scenes:

| What Docker Does | Why It Matters | What You See |
| :--- | :--- | :--- |
| **Creates Virtual Networks** | Isolates containers from each other | `docker network ls` shows your networks |
| **Assigns IP Addresses** | Each container gets its own address | Containers can talk to each other |
| **Handles Port Mapping** | Makes container services accessible | `-p 8080:80` maps host port to container |
| **Manages DNS** | Containers can find each other by name | `ping web-server` works in custom networks |

### Simple Docker Network Flow

```
Your Browser → Your Computer:8080 → Docker Magic → Container:80 → Web Server
```

**What Docker handles for you:**
1. **Traffic Routing** - Gets requests to the right container
2. **Network Isolation** - Keeps containers secure and separated
3. **Name Resolution** - Lets containers find each other by name
4. **Port Management** - Maps external ports to internal services

### Docker Network Types (Simplified)

| Network Type | When to Use | Example |
| :--- | :--- | :--- |
| **Default Bridge** | Single containers, testing | `docker run nginx` |
| **Custom Bridge** | Multi-container apps | `docker network create myapp` |
| **Host** | Maximum performance needed | `docker run --network host nginx` |
| **None** | Complete isolation | `docker run --network none alpine` |

---

## 🎯 What You Need to Remember for Docker

### The Essentials

1. **IP Addresses** - Containers get automatic IP addresses (you usually don't need to worry about them)
2. **Subnets** - Docker creates private networks for your containers (like `172.17.0.0/16`)
3. **NAT** - Docker handles internet access and port mapping automatically
4. **Ports** - Use `-p host_port:container_port` to expose services
5. **Names** - Use container names instead of IP addresses (more reliable)

### Key Docker Networking Commands

```bash
# See what networks exist
docker network ls

# Create a custom network
docker network create myapp-network

# Run container on specific network
docker run --network myapp-network --name web nginx

# Map ports to access from outside
docker run -p 8080:80 nginx

# Check container's network details
docker inspect container-name
```

### Next Steps

Now that you understand these fundamentals, you're ready to:
- ✅ Understand Docker's default bridge network
- ✅ Create and manage custom Docker networks
- ✅ Use container names for communication
- ✅ Troubleshoot basic connectivity issues

---

## 📚 Quick Reference

### Essential Concepts

| Concept | Simple Explanation | Docker Example |
| :--- | :--- | :--- |
| **IP Address** | Container's network address | `172.17.0.2` |
| **Subnet** | Range of IP addresses | `172.17.0.0/16` |
| **Port** | Service endpoint | `80` (web), `3306` (MySQL) |
| **NAT** | Internet access magic | Docker handles this |
| **DNS** | Name to IP translation | `web-server` → `172.17.0.2` |

### Troubleshooting Commands

```bash
# Check if container is running
docker ps

# See container's IP address
docker inspect -f '{{.NetworkSettings.IPAddress}}' container-name

# Test connectivity between containers
docker exec container1 ping container2

# Check what ports are exposed
docker port container-name
```

---

*Master these fundamentals, and Docker networking will make perfect sense! 🐳*

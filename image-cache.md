# 🐳 Docker Image Building 101: Your First Custom Container

> **Session Goal:** Learn how to build a custom Docker image using a `Dockerfile` and understand the concept of build caching.

---

## 1. The Magic of Docker Caching ✨ (The Fast Run)

Docker is *intelligent*! It saves the result of each `Dockerfile` instruction as a **layer** in a **build cache**. If an instruction hasn't changed, it simply reuses the cached layer, making subsequent builds *lightning fast*!

### Step 1: Create a Dockerfile and build the image
```dockerfile
FROM debian:bookworm-slim   
LABEL maintainer="johnhonai@harihar.nagar"
RUN apt-get update && apt-get install -y nginx  
COPY index.html /var/www/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Step 2: Making a Small Change

Edit the `Dockerfile` to add a new exposed port. This change is near the end of the file.

| Action | Description |
| :--- | :--- |
| **Edit `Dockerfile`** | Add a new line for port `8080`. (e.g., `EXPOSE 8080`) |

**Updated Dockerfile:**
```dockerfile
FROM debian:bookworm-slim   
LABEL maintainer="johnhonai@harihar.nagar"
RUN apt-get update && apt-get install -y nginx  
COPY index.html /var/www/html/
EXPOSE 80
EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
```

### Step 3: Re-Building with Caching

Run the exact same build command again. Observe the output closely!

| Command | Description | Key Insight |
| :--- | :--- | :--- |
| `$ docker image build -t custom-nginx .` | Re-runs the build. | **Watch the steps!** Steps 1-4 will say **"Using cache"**. Step 5 (the change) and any subsequent steps will be **re-executed**. |

**Expected Output with Caching:**
```bash
Sending build context to Docker daemon  2.048kB
Step 1/6 : FROM debian:bookworm-slim
 ---> Using cache
 ---> def456ghi789
Step 2/6 : RUN apt-get update && apt-get install -y nginx
 ---> Using cache
 ---> abc123def456
Step 3/6 : RUN ln -sf /dev/stdout /var/log/nginx/access.log
 ---> Using cache
 ---> ghi789jkl012
Step 4/6 : RUN ln -sf /dev/stderr /var/log/nginx/error.log
 ---> Using cache
 ---> mno345pqr678
Step 5/6 : EXPOSE 80
 ---> Using cache
 ---> stu901vwx234
Step 6/7 : EXPOSE 8080
 ---> Running in yz567abc890  # ← This step runs because it's new
 ---> def123ghi456
Step 7/7 : CMD ["nginx", "-g", "daemon off;"]
 ---> Running in jkl789mno012  # ← This step runs because previous step changed
 ---> pqr345stu678
Successfully built pqr345stu678
Successfully tagged custom-nginx:latest
```

### Understanding the Cache Behavior

| Output Snippet | Explanation |
| :--- | :--- |
| `Step 2/6 : RUN apt-get update...` **`---> Using cache`** | This instruction hasn't changed, so Docker reuses the existing layer instantly. |
| `Step 6/7 : EXPOSE 8080` | This line *changed*, so Docker runs it. |
| `Step 7/7 : CMD ["nginx", "-g", "daemon off;"]` | This line *didn't change*, **but** since a previous line (Step 6) changed, this layer **MUST BE REBUILT** to ensure consistency. |

---

## 2. Key Takeaway: The Caching Golden Rule 🥇

The location of your instructions in the `Dockerfile` **matters a lot** for build time!

### The Caching Rule

| **Concept** | **Why it Matters** | **Best Practice** |
| :--- | :--- | :--- |
| **Order of Instructions** | Docker builds layers sequentially. **A change to any instruction invalidates the cache for all subsequent instructions.** | **Place instructions that change the LEAST at the TOP** (e.g., `FROM`, installing OS dependencies). |
| **Code Copying Example** | If you copy your application source code (`COPY . /app`) at the top, *every time you change a single source file*, the entire subsequent build cache is invalidated, leading to long build times. | **Place instructions that change the MOST at the BOTTOM** (e.g., `COPY` application code, `CMD`). |

### Optimized Dockerfile Structure

```dockerfile
# ✅ GOOD: Stable instructions at the top
FROM debian:jessie                          # Rarely changes
RUN apt-get update && apt-get install -y nginx  # Rarely changes
EXPOSE 80                                   # Rarely changes

# ✅ GOOD: Frequently changing instructions at the bottom
COPY ./app /var/www/html/                   # Changes often
CMD ["nginx", "-g", "daemon off;"]         # May change during development
```

### ❌ Anti-Pattern Example

```dockerfile
# ❌ BAD: Frequently changing instruction at the top
FROM debian:jessie                          
COPY ./app /var/www/html/                   # Changes often - invalidates all cache below!
RUN apt-get update && apt-get install -y nginx
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## 🎯 Session Summary

**What You've Learned:**

1. **Dockerfile Basics**: How to write a simple Dockerfile with `FROM`, `RUN`, `EXPOSE`, and `CMD`
2. **Image Building**: Using `docker image build -t <name> .` to create custom images
3. **Docker Caching**: How Docker reuses layers to speed up builds
4. **Optimization Strategy**: Placing stable instructions at the top and changing instructions at the bottom

**Next Steps:**
- Try modifying the Dockerfile with different base images (e.g., `alpine:latest`)
- Experiment with adding more `RUN` commands and observe caching behavior
- Practice with `COPY` commands to add your own files to the image

**Key Commands to Remember:**
```bash
# Build an image
docker image build -t my-image-name .

# List images
docker image ls

# Run a container from your custom image
docker container run -d -p 8080:80 my-image-name
```

---

*Happy Dockerizing! 🐳*

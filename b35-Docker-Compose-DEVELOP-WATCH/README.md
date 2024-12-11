# Docker Compose: WATCH

---

## Step 1: Introduction

In this guide, you'll learn to use the **Docker Compose Develop Watch** feature to:

- **Sync code changes** in real-time without rebuilding images.
- Enable the feature with the `--watch` option.

---

## Step 2: Review Dockerfile

**File Location:** `sync-demo/web/Dockerfile`

```dockerfile
# Use the official NGINX base image
FROM nginx:latest

# Copy custom NGINX configuration file to replace the default.conf
COPY ./nginx.conf /etc/nginx/nginx.conf

# Copy static website files to the container's HTML directory
COPY ./html /usr/share/nginx/html

# Expose port 8080 for external access
EXPOSE 8080

# Start NGINX
CMD ["nginx", "-g", "daemon off;"]
```

---

## Step 3: Review Other Files

### HTML Files

- `sync-demo/web/html/index.html`
- `sync-demo/web/html/custom_404.html`

### NGINX Configuration

**File Location:** `sync-demo/web/nginx.conf`

```conf
events { }

http {
  server {
    listen 8080;

    # Serve files from the root html directory for '/'
    location / {
      root /usr/share/nginx/html;  # Serve static files from this directory
      index index.html;  # Serve index.html by default if it exists
    }

    # Custom 404 page - ENABLE below 5 lines to test "sync+restart" option in Docker Compose
    # error_page 404 /custom_404.html;
    # location = /custom_404.html {
    #   root /usr/share/nginx/html;  # Location of the custom 404 page
    #   internal;
    # }

  }
}
```

---

## Step 4: Review `docker-compose.yaml`

**File Location:** `sync-demo/docker-compose.yaml`

```yaml
services:
  web:
    container_name: mywebserver1
    build:
      context: ./web  # The path to the Dockerfile
      dockerfile: Dockerfile  # The Dockerfile to use for building the image
    develop:
      watch:
        # Sync changes to static content
        - path: ./web/html
          action: sync
          target: /usr/share/nginx/html
        # Sync changes to nginx.conf file
        - path: ./web/nginx.conf
          action: sync+restart
          target: /etc/nginx/nginx.conf
    ports:
      - "8080:8080"
```

---

## Step 5: Start the Stack and Verify

Use the `--watch` option to enable the Develop Watch feature.

```bash
# Change directory
cd $(git rev-parse --show-toplevel)/b35-Docker-Compose-DEVELOP-WATCH/sync-demo

# Pull Docker Images and Start Containers with --watch option
docker compose up --watch

# List Docker Containers
docker compose ps
```

Access Application http://localhost:8080

```text
# Observation:
# V1 version of the application will be displayed.
```

---

## Step 6: Test Sync Option: Update `index.html` to Version 2

**File Location:** `sync-demo/web/html/index.html`

Update the content:

```html
<p>Application Version: V2</p>
```

 Access Application http://localhost:8080

```bash
# Observation:
# - Changes in static content will be synced to the container automatically.
# - You should see the V2 version of the application without restarting the container.
```

---

## Step 7: Clean Up

```bash
# Stop and Remove Containers
docker compose down -v
docker volume prune -af
docker network prune -f
```

---

## Conclusion

In this tutorial, you learned how to use the **Docker Compose Develop Watch** feature to:

- **Automatically sync** code or static file changes to running containers.
- This speeds up development by eliminating the need to rebuild images or restart containers for every change.

---

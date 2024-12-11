# Docker Compose: WATCH + Automatic Docker Image Builds

---

## Step 1: Introduction

In this guide, you will learn how to use the **Docker Compose Develop Watch** feature with the rebuild option to:

- **Sync code changes** in real-time without rebuilding images.
- **Automatically rebuild images** when specific configuration files change.

---

## Step 2: Review Dockerfile

**File Location:** `sync-and-rebuild-demo/web/Dockerfile`

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

- `sync-and-rebuild-demo/web/html/index.html`
- `sync-and-rebuild-demo/web/html/custom_404.html`

### NGINX Configuration

**File Location:** `sync-and-rebuild-demo/web/nginx.conf`

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

    # Custom 404 page - ENABLE below 5 lines to test "sync+rebuild" option in Docker Compose
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

**File Location:** `sync-and-rebuild-demo/docker-compose.yaml`

```yaml
services:
  web:
    container_name: mywebserver2
    build:
      context: ./web  # The path to the Dockerfile
      dockerfile: Dockerfile  # The Dockerfile to use for building the image
    develop:
      watch:
        # Sync changes to static content
        - path: ./web/html
          action: sync
          target: /usr/share/nginx/html
        # Rebuild image when nginx.conf changes
        - path: ./web/nginx.conf
          action: rebuild
          target: /etc/nginx/nginx.conf
    ports:
      - "8080:8080"
```

---

## Step 5: Start the Stack and Verify

Use the `--watch` option to enable the Develop Watch feature.

```bash
# Change directory
cd $(git rev-parse --show-toplevel)/b36-Docker-Compose-DEVELOP-WATCH-REBUILD/sync-and-rebuild-demo

# Pull Docker Images and Start Containers with --watch option
docker compose up --watch

# List Docker Containers
docker compose ps
```

Access Application http://localhost:8080

```bash
# Observation:
# V1 version of the application will be displayed.
```

---

## Step 6: Test Sync Option: Update `index.html` to Version 2

**File Location:** `sync-and-rebuild-demo/web/html/index.html`

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

## Step 7: Test Default NGINX 404 Page

Before testing the `rebuild` option, check the default 404 page.

Access a non-existent page to test the 404 page http://localhost:8080/abc

```bash
# Observation:
# The default 404 page of NGINX will be displayed.
```

---

## Step 8: Test Rebuild Option: Update `nginx.conf`

### Step 8-01: Enable Custom 404 Page in `nginx.conf`

Uncomment the following lines in `nginx.conf`:

```conf
    # Custom 404 page - ENABLE below 5 lines to test "sync+rebuild" option in Docker Compose
    error_page 404 /custom_404.html;
    location = /custom_404.html {
      root /usr/share/nginx/html;  # Location of the custom 404 page
      internal;
    }
```

### Step 8-02: Verify if Docker Image is Rebuilt and New Container Created

```bash
# List Docker images
docker images

# Observation:
# 1. You should see a new Docker image built after making changes to 'nginx.conf'.
# 2. The 'rebuild' action defined in the Docker Compose file rebuilt the Docker image when 'nginx.conf' was updated.

# List Containers
docker ps
```

Access a non-existent page to test the custom 404 page http://localhost:8080/abc

```bash
# Observation:
# The custom 404 page of NGINX will be displayed.
```

---

## Step 9: Clean Up

```bash
# Stop and Remove Containers
docker compose down -v
docker volume prune -af
docker network prune -f
```

---

## Conclusion

In this tutorial, you learned to use the **Docker Compose Develop Watch** feature with the rebuild option to:

- **Sync changes** to code or static files in real-time.
- **Automatically rebuild images** and recreate containers when configuration files change.

This improves development speed by removing the need to manually rebuild images or restart containers for each change.

---

## Additional Notes

- **Sync vs. Rebuild**:
  - **Sync**: Updates files without rebuilding the image, suitable for static content or code changes.
  - **Rebuild**: Rebuilds the image when specified files change, needed for updates in `Dockerfile` or configuration files.

- **Efficient Development**: The Develop Watch with rebuild is ideal for environments with frequent configuration changes.

- **Limitations**: Best suited for development, not recommended for production due to frequent image rebuilding.

---

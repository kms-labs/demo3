# Docker Compose: Deploy, Scale, Load Balance

---

## Step 1: Introduction

In this guide, we will explore Docker Compose's `deploy` option to achieve:

1. Scaling the **app-ums** service to 3 containers.
2. Using **web-nginx** to load balance traffic across 3 **app-ums** service containers.

---

## Step 2: Review `docker-compose.yaml`

```yaml
name: ums-stack
services:
  web-nginx:
    image: nginx:latest
    container_name: ums-nginx
    ports:
      - "8080:8080"  # NGINX listens on port 8080 of the host
    depends_on:
      - app-ums
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf  # Custom NGINX configuration

  app-ums:
    image: ghcr.io/stacksimplify/usermgmt-webapp-v6:latest
    ports:
      - "8080"  # Only expose the container's port, let Docker choose the host port
    deploy:
      replicas: 3  # Scale the service to 3 instances
    depends_on:
      - db-mysql
    environment:
      - DB_HOSTNAME=db-mysql
      - DB_PORT=3306
      - DB_NAME=webappdb
      - DB_USERNAME=root
      - DB_PASSWORD=dbpassword11

  db-mysql:
    container_name: ums-mysqldb
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: dbpassword11
      MYSQL_DATABASE: webappdb
    ports:
      - "3306:3306"
    volumes:
      - mydb:/var/lib/mysql

volumes:
  mydb:
```

---

## Step 3: Review `nginx.conf`

```conf
events { }

http {
  resolver 127.0.0.11 ipv6=off;

  upstream app-ums {
    server app-ums:8080;
    # Session persistence using client's IP address for stateful apps (UMS)
    # ip_hash;  # Disable this to see how load balancing works
  }

  server {
    listen 8080;

    location / {
      proxy_pass http://app-ums;
    }
  }
}
```

---

## Step 4: How to Start the UMS Stack

Start the UMS stack by following these steps:

```bash
# Change directory
cd $(git rev-parse --show-toplevel)/b27-Docker-Compose-DEPLOY/ums-stack

# Start the UMS Stack in detached mode
docker compose up -d
```

The `-d` flag runs the services in the background.

---

## Step 5: Verify Services

To verify the MySQL and UMS WebApp services:

```bash
# List running Docker containers
docker compose ps
```

To view the logs of the services:

```bash
docker compose logs db-mysql
docker compose logs app-ums
docker compose logs web-nginx
```

To check logs of individual containers:

```bash
docker logs ums-stack-app-ums-1
docker logs ums-stack-app-ums-2
docker logs ums-stack-app-ums-3
```

---

## Step 6: Verify the Load Balancing

```bash
# Access API that displays the container ID
http://localhost:8080/hello1

# Run a loop to check load balancing between multiple app-ums containers
while true; do curl http://localhost:8080/hello1; echo ""; sleep 1; done

# Observation:
# Requests will be distributed across the containers.
```

---

## Step 7: Stop and Clean Up

To stop and remove the running containers:

```bash
docker compose down
```

To remove the MySQL volume as well:

```bash
docker compose down -v
docker volume prune -af
docker network prune -f
```

---

## Conclusion

In this guide, you learned to:
- Scale a multi-container stack with Docker Compose, including NGINX load balancing and UMS WebApp scaling.
- Use Docker Compose's `deploy` and custom NGINX configurations for scalable, balanced microservices.

---

## Additional Notes

- **Scaling**: The `deploy` option in Docker Compose scales services with replicas, working with load balancers like NGINX.
- **NGINX Configuration**: Custom setups with Docker’s DNS enable precise load balancing.

---

# Docker Compose: Deploy UMS Stack

---

## Step 1: Introduction

This demo showcases deploying a **User Management Web Application (UMS)** and a **MySQL database** using **Docker Compose**. The UMS stack includes:

1. **MySQL Database**: Stores user data.
2. **UMS WebApp**: Manages users with login, create, and list operations, connecting to the database.

---

## Step 2: Prerequisites

Ensure that you have the following tools installed on your system:

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

---

## Step 3: Project Structure

```bash
ums-stack/
├── docker-compose.yaml   # Docker Compose configuration file
```

---

## Step 4: Services

### MySQL Service:
- Based on `mysql:8.0` image.
- Configured via environment variables.
- Exposes port `3306`.
- Stores data in a named volume (`mydb`) for persistence.

### UMS WebApp Service:
- Uses `ghcr.io/stacksimplify/usermgmt-webapp-v6:latest`.
- Connects to the MySQL database.
- Exposes port `8080` on the host.

### docker-compose.yaml

```yaml
name: ums-stack
services:
  myumsapp:
    container_name: ums-app
    image: ghcr.io/stacksimplify/usermgmt-webapp-v6:latest
    ports:
      - "8080:8080"
    depends_on:
      - mysql
    environment:
      - DB_HOSTNAME=mysql
      - DB_PORT=3306
      - DB_NAME=webappdb
      - DB_USERNAME=root
      - DB_PASSWORD=dbpassword11
  mysql:
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

### Key Configuration

#### UMS WebApp Service: `myumsapp`
- **`DB_HOSTNAME`**: Set to `mysql`.
- **`DB_PORT`**: `3306`.
- **`DB_NAME`, `DB_USERNAME`, `DB_PASSWORD`**: Database credentials.

#### MySQL Service: `mysql`
- **`MYSQL_ROOT_PASSWORD`**: Root password.
- **`MYSQL_DATABASE`**: Pre-configured `webappdb` for UMS WebApp.

### Volume: `mydb`
- Named Docker volume for persistent MySQL data across restarts or removals.

---

## Step 5: How to Start the UMS Stack

Follow the steps below to start the UMS Stack:

```bash
# Change directory
cd $(git rev-parse --show-toplevel)/b26-Docker-Compose-UMS-Stack/ums-stack

# Start the UMS Stack in detached mode
docker compose up -d
```

- The `-d` flag runs the services in detached mode, allowing them to run in the background.

---

## Step 6: Verify Services

To ensure that both the MySQL and UMS WebApp services are running:

```bash
# List running Docker containers
docker compose ps
```

To view logs for both services:

```bash
# View logs with docker compose
docker compose logs -f mysql
docker compose logs -f myumsapp

# View logs using container names
docker logs -f ums-mysqldb
docker logs -f ums-app
```

---

## Step 7: Accessing the UMS WebApp

1. Open your browser at http://localhost:8080.
2. Log in with:
   - **Username**: `admin101`
   - **Password**: `password101`
3. After logging in:
   - View users.
   - Create new users.
   - Test login with new users.

---

## Step 8: Connect to MySQL Container

To inspect or interact with the MySQL database, connect to the MySQL container:

```bash
# Connect to MySQL container
docker exec -it ums-mysqldb mysql -u root -pdbpassword11
```

Once inside the MySQL shell, you can run SQL queries to view the `webappdb` database:

```bash
mysql> show schemas;
mysql> use webappdb;
mysql> show tables;
mysql> select * from user;
```

---
## Step 9: Verify the Container to Container communication using Service Names using DNS
```bash
# List running Docker containers
docker compose ps

# Connect to ums-app container
docker exec -it ums-app /bin/bash

# Test DNS - myumsapp
nslookup myumsapp
dig myumsapp

# Test DNS - mysql
nslookup mysql
dig mysql
```

---
## Step 10: Stop and Clean Up

When you're done, stop and remove the running containers:

```bash
# Stop and remove the containers
docker compose down
```

To remove the MySQL volume as well (optional):

```bash
# Stop containers and remove volumes
docker compose down -v
docker volume prune -af
docker network prune -f
```

**Observation**: The command `docker compose down -v` removes both the containers and the named volume `mydb`, thus deleting the persistent data.

---

## Conclusion

In this tutorial, we deployed:
- **MySQL database** and **User Management WebApp** with Docker Compose.
- Configured MySQL for persistence and connected the WebApp for interaction, enabling scalability and extensibility.

---

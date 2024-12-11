# Docker Compose: BUILD

---

## Step 1: Introduction

- **`--build` Flag**: Forces Docker Compose to rebuild images before starting containers. Useful when changes are made to the Dockerfile, code, or environment variables.

- **Automatic Build**: With `--build`, you don't need to manually run `docker compose build` before `docker compose up`. It combines both steps into one.

- **Use Cases**:
  - Code or File Changes
  - Dockerfile Updates
  - Environment Variable Modifications

- **Automatic vs. Manual Build**: Without `--build`, Docker Compose uses existing images. With `--build`, it rebuilds the image regardless of previous builds.


## Step 2: Review Sample Application
- Folder: python-app
- **File: app.py**
```py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "V1: Hello, Docker Compose Build Demo"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

- **File: requirements.txt**
```bash
flask
```
- **File: Dockerfile**
```Dockerfile
# Use the official Python image from the Docker Hub
FROM python:slim

# Set the working directory
WORKDIR /app

# Copy the requirements file and install the dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy the current directory contents into the container at /app
COPY . .

# Expose the app on port 5000
EXPOSE 5000

# Run the application
CMD ["python", "app.py"]
```

## Step 3: Review docker-compose.yaml
```yaml
services:
  web:
    image: my-python-app:latest  # Name of the Docker image
    container_name: my-python-container  # Name of the container
    build:
      context: ./python-app
      dockerfile: Dockerfile  # The Dockerfile to use for building the image
    ports:
      - "5000:5000"
```

## Step 4: Start the Stack and Verify
```bash
# Change directory
cd $(git rev-parse --show-toplevel)/b34-Docker-Compose-BUILD/build-demo

# Pull Docker Images and Start Containers
docker compose up --build -d

# List Docker Containers
docker compose ps
# Observation:
# Verify the Docker image "CREATED" section
```

Access Application http://localhost:5000

```text
# Observation:
# V1 version of application displayed
```

## Step 5: Change Code to V2 version
```py
# Update app.py
return "V2: Hello, Docker Compose Build Demo"
```
## Step 6: Deploy V2 version of Application

```bash
# Change directory
cd $(git rev-parse --show-toplevel)/b34-Docker-Compose-BUILD/build-demo

# Re-build Docker Images and Start Containers
docker compose up --build -d

# Observation:
# 1. Verify the Docker image "CREATED" section
# 2. New Docker image will be created

# List Docker Containers
docker compose ps
# Observation:
# Container will be recreated with new Docker Image
```

Access Application http://localhost:5000

```text
# Observation:
# V2 version of application will be displayed
```

## Step 7: Clean-up
```bash
# Stop and Remove Containers
docker compose down -v
docker volume prune -af
docker network prune -f
```

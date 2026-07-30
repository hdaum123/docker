# Docker Compose – Multi-Container Tic Tac Toe Deployment

## Overview

This project demonstrates how to deploy a multi-container application using **Docker Compose**. Instead of manually creating and managing individual containers, Docker Compose allows the entire application stack to be defined in a single `compose.yaml` file and started with one command.

The deployment consists of:

- A Node.js Tic Tac Toe application
- A MongoDB database
- A dedicated seed container that automatically populates the database

---

# Objectives

The objectives of this task were to:

- Deploy multiple containers using Docker Compose.
- Use the official MongoDB Docker image.
- Use the application image previously built and pushed to Docker Hub.
- Connect the application to MongoDB using environment variables.
- Store database data in a persistent Docker volume.
- Perform both manual and automated database seeding.
- Verify that all services communicate successfully.

---

# Prerequisites

Before completing this task I had already:

- Docker Desktop installed.
- Built the application image using a Dockerfile.
- Tagged the image.
- Pushed the image to Docker Hub.

Application image:

```text
hdaum123/tech610-tttapp:1.2.0
```

---

# Project Structure

```text
docker-compose-ttt/
└── compose.yaml
```

No Dockerfile was required because the application image had already been built during the previous task.

---

# Docker Compose Configuration

The deployment was defined inside a single `compose.yaml` file.

It contained three services:

- database
- app
- seed

and one named volume:

```yaml
volumes:
  mongo-data:
```

---

# Database Service

```yaml
database:
  image: mongo:8.0
  container_name: ttt-mongodb
  ports:
    - "27017:27017"
  volumes:
    - mongo-data:/data/db
```

### Purpose

- Downloads the official MongoDB image.
- Creates the MongoDB container.
- Exposes MongoDB on port 27017.
- Persists database data using a Docker volume.

---

# Application Service

```yaml
app:
  image: hdaum123/tech610-tttapp:1.2.0
  container_name: ttt-app
  ports:
    - "3000:3000"
  environment:
    MONGODB_URI: mongodb://database:27017/tictactoe
  depends_on:
    - database
```

### Purpose

- Pulls my application image from Docker Hub.
- Starts the Node.js application.
- Publishes port 3000.
- Connects to MongoDB using the environment variable.
- Waits for the database container to start first.

---

# Networking

Docker Compose automatically creates an internal network.

Instead of using an IP address, the application connects to MongoDB using the service name:

```text
mongodb://database:27017/tictactoe
```

The service name `database` acts as the hostname.

---

# Persistent Storage

A named Docker volume was created:

```yaml
volumes:
  mongo-data:
```

This ensures the MongoDB data remains available even if the containers are stopped or recreated.

---

# Running the Deployment

Start all services:

```bash
docker compose up -d
```

Check running containers:

```bash
docker compose ps
```

Stop the deployment:

```bash
docker compose down
```

View logs:

```bash
docker compose logs
```

---

# Verifying the Application

The application was successfully accessed using:

```text
http://localhost:3000
```

I also confirmed the application was using the correct environment variable.

```bash
docker exec -it ttt-app sh -c 'echo "$MONGODB_URI"'
```

Output:

```text
mongodb://database:27017/tictactoe
```

This confirmed the application could locate the MongoDB service.

---

# Manual Database Seeding

Before automating the process, I first confirmed that the application could communicate with MongoDB by manually executing the seed script inside the running application container.

```bash
docker exec -it ttt-app node seeds/seed.js
```

Output:

```text
Seeded active app state via /api/seed (10 records).
```

This confirmed:

- the application was running correctly
- MongoDB connectivity was working
- the database was successfully populated

At this stage there was **no dedicated seed service**. The seed script was executed manually from the application container.

---

# Automated Database Seeding

After confirming manual seeding worked, I automated the process by creating a dedicated **seed** service inside `compose.yaml`.

```yaml
seed:
  image: hdaum123/tech610-tttapp:1.2.0
  container_name: ttt-seed
  environment:
    MONGODB_URI: mongodb://database:27017/tictactoe
  depends_on:
    - app
  command: node seeds/seed.js
  restart: "no"
```

This service uses the same application image but performs only one task:

```bash
node seeds/seed.js
```

When the deployment starts:

```bash
docker compose up -d
```

Docker automatically creates the `ttt-seed` container, executes the seed script, inserts the records into MongoDB and exits.

Verification:

```bash
docker compose ps -a
```

Expected result:

```text
ttt-seed    Exited (0)
```

This is the expected behaviour because the seed container completes its task and then stops.

---

# Manual vs Automated Seeding

| Manual Seeding | Automated Seeding |
|----------------|-------------------|
| User manually runs the command. | Docker Compose runs it automatically. |
| Executed inside `ttt-app`. | Executed inside `ttt-seed`. |
| `docker exec -it ttt-app node seeds/seed.js` | `command: node seeds/seed.js` |
| Requires user intervention. | Runs automatically during deployment. |

Although both methods use the same `seeds/seed.js` script, the execution method is different.

---

# Useful Commands

```bash
docker compose up -d
docker compose down
docker compose ps
docker compose ps -a
docker compose logs
docker exec -it ttt-app sh
docker exec -it ttt-mongodb mongosh
docker exec -it ttt-app node seeds/seed.js
```

---

# Challenges Encountered

## Port Conflict

The application was initially inaccessible because another container was already using port 3000.

**Resolution**

Stopped the old container and recreated the deployment.

---

## Database Connectivity

Initially the seed script returned:

```text
Cannot seed scoreboard because mode is "Client-local stateful"
```

**Resolution**

Verified:

- environment variables
- Docker networking
- MongoDB container
- application connectivity

Once the application successfully connected to MongoDB, the seed script completed successfully.

---

# Why No Dockerfile Was Needed

A Dockerfile was **not** required for this task because the application image had already been built and pushed to Docker Hub during the previous exercise.

The Compose file uses:

```yaml
image: hdaum123/tech610-tttapp:1.2.0
```

which downloads an existing image.

A Dockerfile would only be needed if the Compose file used:

```yaml
build: .
```

---

# Key Learning Outcomes

Through this task I learned how to:

- Deploy multiple containers using Docker Compose.
- Connect services using Docker's internal network.
- Configure environment variables between containers.
- Use Docker volumes for persistent storage.
- Perform manual database seeding.
- Automate database seeding using a dedicated service.
- Troubleshoot networking and port conflicts.
- Understand when a Dockerfile is and is not required.

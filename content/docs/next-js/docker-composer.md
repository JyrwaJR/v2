---
title: "Docker Compose Setup"
description: "Comprehensive guide to setting up a Next.js application using Docker and Docker Compose for both development and production."
---

This document provides a detailed explanation for setting up a Next.js application using Docker and Docker Compose. It includes separate configurations for development and production environments to ensure an optimized and seamless workflow.

---

## **Docker Compose Configuration**

### **docker-compose.yml**

This file defines two services: `dev` (for development) and `prod` (for production).

```yaml
services:
  dev:
    build:
      context: .
      dockerfile: Dockerfile.dev
    container_name: quiz-docker-dev
    environment:
      - WATCHPACK_POLLING=true
    volumes:
      - .:/app
      - /app/node_modules
      - /app/.next
    ports:
      - "3000:3000"
    env_file:
      - .env.local

  prod:
    build:
      context: .
      dockerfile: Dockerfile.prod
    container_name: quiz-docker
    ports:
      - "3000:3000"
    env_file:
      - .env.local
```

### **Explanation**

1. **`build`**:
   - **`context`**: Points to the directory containing the Dockerfile.
   - **`dockerfile`**: Specifies the Dockerfile for the respective environment.
2. **`container_name`**:
   - Assigns a unique name to the container.
3. **`volumes`** (only for `dev`):
   - Mounts the local directory to the container for live development.
4. **`ports`**:
   - Maps container port `3000` to the host port `3000`.
5. **`env_file`**:
   - Specifies the `.env.local` file to provide environment variables.

---

## **Dockerfile Configurations**

### **Dockerfile.dev**

Used for the development environment with live reload and debugging capabilities.

```Dockerfile
# Stage 1: Install dependencies for development
FROM node:20-alpine3.19 AS dev
WORKDIR /app

# Install dependencies (including dev dependencies)
COPY package.json package-lock.json ./
RUN npm install

# Copy application source code
COPY . .

# Expose the development port
EXPOSE 3000

# Set environment variable for development
ENV NODE_ENV=development

# Command to run the application in development mode
CMD ["npm", "run", "dev"]
```

### **Dockerfile.prod**

Used for creating a slim production-ready image.

```Dockerfile
# Stage 1: Install dependencies and build the app
FROM node:20-alpine3.19 AS builder
WORKDIR /app

# Install dependencies (including dev dependencies for type checks and builds)
COPY package.json package-lock.json ./
RUN npm install

# Copy application source code
COPY . .

# Build the Next.js app
RUN npm run build

# Stage 2: Create a slim production image
FROM node:20-alpine3.19 AS prod
WORKDIR /app

# Install only production dependencies
COPY package.json package-lock.json ./
RUN npm install --only=production

# Copy built files from the builder stage
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/public ./public
COPY --from=builder /app/package.json ./package.json

# Expose the production port
EXPOSE 3000

# Set environment variable for production
ENV NODE_ENV=production

# Command to run the application in production mode
CMD ["npm", "start"]
```

---

## **How to Use**

### **Run in Development Mode**

1. Build and run the development container:
   ```bash
   docker-compose up dev
   ```
2. This mounts your local directory into the container, enabling live reload for code changes.

### **Before Switching Modes**

Before switching between development and production modes, clean up Docker resources:

```bash
docker-compose down --volumes
docker builder prune
```

### **Run in Production Mode**

1. Build and run the production container:
   ```bash
   docker-compose up prod --build
   ```
2. This uses the optimized production image created in the `Dockerfile.prod`.

---

## **Managing Logs**

### **View Logs**

To check logs of your application:

```bash
docker-compose logs nextjs-app
```

---

## **Key Features of the Setup**

1. **Separate Environments**:

   - Development: Live reload with source code mapping.
   - Production: Optimized build with minimal dependencies.

2. **Multi-Stage Builds**:

   - The production image is created in two stages:
     - Builder stage: Installs dependencies and builds the application.
     - Production stage: Copies the built files and installs only necessary production dependencies.

3. **Environment Variables**:

   - Use `.env.local` for environment-specific configurations.

4. **Volume Mounting**:

   - In development mode, local changes reflect immediately in the container.

5. **Port Mapping**:
   - Both environments expose the application on port `3000`.

---

This setup ensures a smooth workflow for both development and production with a consistent and optimized Docker configuration.

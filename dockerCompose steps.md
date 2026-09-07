Great. 👍 Now add the following to your **`Dockerfile`**:

```dockerfile
# Use Node.js LTS image
FROM node:22-alpine

# Set working directory inside the container
WORKDIR /app

# Copy package files first
COPY package*.json ./

# Install only production dependencies
RUN npm ci --omit=dev

# Copy application source code
COPY . .

# Application port
EXPOSE 3000

# Start the application
CMD ["node", "server.js"]
```

## What this Dockerfile does

```text
node:22-alpine
      ↓
Creates container environment
      ↓
WORKDIR /app
      ↓
Copy package.json
      ↓
Install dependencies
      ↓
Copy source code
      ↓
Start server.js
```

### Why copy `package*.json` first?

Docker can cache this layer:

```text
package.json changes?  → npm install again
Source code changes?   → reuse installed dependencies
```

This makes Docker builds faster.

## Next step

Save the `Dockerfile`.

Then we'll create:

```text
.dockerignore
```

This is important so we don't copy things like:

```text
node_modules/
.env
```

Excellent! 🎉 Your **Shop Service Docker image has been built successfully**.

Now we need to run it as a container.

## Step 3 — Run the Shop Service container

Because your `.env` is excluded from the image, pass it when starting the container:

```powershell
docker run --name bloomworld-shop-service `
  --env-file .env `
  -p 3000:3000 `
  -d bloomworld-shop-service
```

### Architecture right now

```text
Windows Host
│
├── PostgreSQL Container
│     └── Port 5433
│
└── Shop Service Container
      └── Port 3000
```

⚠️ **Important:** This may fail to connect to PostgreSQL because inside the Shop Service container:

```text
127.0.0.1
```

means the **Shop Service container itself**, not your Windows machine or PostgreSQL container.

So let's first run it and see the result.

After running the command, check the logs:

```powershell
docker logs bloomworld-shop-service
```

Send me the output.


into the Docker image.

Commands
docker build -t bloomworld-shop-service .
docker images


Great 👍 Now you understand the important networking concept.

Your current setup is working like this:

```text
Browser
   │
   │ localhost:3000
   ▼
Windows Host Port 3000
   │
   │ Docker port mapping
   ▼
Shop Service Container
   │
   │ DB_HOST=bloomworld-postgres
   │ DB_PORT=5432
   ▼
Docker Network DNS
   │
   ▼
PostgreSQL Container
```

## Next step: Docker Compose

Right now you manually created and managed:

* Docker network
* PostgreSQL container
* Shop Service container

Docker Compose can define all of that in one file and start everything with:

```powershell
docker compose up -d
```

Let's do that next.


Great 👍 Now let's add the Docker Compose configuration.

## Step 3 — Add code to `docker-compose.yml`

Open:

```text
bloomworld/docker-compose.yml
```

Add:

```yaml
services:

  postgres:
    image: postgres:16
    container_name: bloomworld-postgres
    environment:
      POSTGRES_USER: bloomworld
      POSTGRES_PASSWORD: bloomworld123
      POSTGRES_DB: bloomworld
    ports:
      - "5433:5432"

  shop-service:
    build:
      context: ./shop-service
    container_name: bloomworld-shop-service
    env_file:
      - ./shop-service/.env
    ports:
      - "3000:3000"
    depends_on:
      - postgres
```

## Important: Update your `.env`

Because Docker Compose automatically creates a network and gives services DNS names, your `.env` should contain:

```env
DB_USER=bloomworld
DB_HOST=postgres
DB_NAME=bloomworld
DB_PASSWORD=bloomworld123
DB_PORT=5432

PORT=3000
```

Notice:

```text
DB_HOST=postgres
```

`postgres` is the **service name** in:

```yaml
services:
  postgres:
```

So Docker Compose creates this communication:

```text
shop-service
     │
     │ postgres:5432
     ▼
postgres
```

Save both files. Then tell me **done**, and we'll run your entire application with one Docker Compose command.

This is a **common Docker Compose startup timing issue**. Your configuration and network are working. 👍

Look at the order:

```text
Shop Service started
↓
Tried to connect to PostgreSQL ❌
↓
PostgreSQL was still initializing
↓
PostgreSQL became ready ✅
```

Your logs show:

```text
Database connection failed: ECONNREFUSED
```

and shortly after:

```text
database system is ready to accept connections
```

So the problem is **not networking**. The database simply wasn't ready yet.

## Why `depends_on` didn't solve it

You have:

```yaml
depends_on:
  - postgres
```

This means:

> Start the PostgreSQL container before starting the Shop Service container.

But it does **not necessarily mean PostgreSQL is ready to accept connections** before your app starts.

---

# Fix: Add a PostgreSQL health check

Update your `docker-compose.yml`.

Change the `postgres` service to:

```yaml
services:

  postgres:
    image: postgres:16
    container_name: bloomworld-postgres

    environment:
      POSTGRES_USER: bloomworld
      POSTGRES_PASSWORD: bloomworld123
      POSTGRES_DB: bloomworld

    ports:
      - "5433:5432"

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U bloomworld -d bloomworld"]
      interval: 5s
      timeout: 5s
      retries: 10

  shop-service:
    build:
      context: ./shop-service

    container_name: bloomworld-shop-service

    env_file:
      - ./shop-service/.env

    ports:
      - "3000:3000"

    depends_on:
      postgres:
        condition: service_healthy
```

Now the startup flow becomes:

```text
PostgreSQL Container
        │
        ▼
Initializing...
        │
        ▼
Health Check
        │
        ▼
PostgreSQL Healthy ✅
        │
        ▼
Start Shop Service
        │
        ▼
Database Connected ✅
```

---

## Next step

Stop the current Compose setup by pressing:

```text
Ctrl + C
```

Then run:

```powershell
docker compose down
```

Then start again:

```powershell
docker compose up --build
```

You should then see:

```text
PostgreSQL is ready
```

followed by:

```text
Shop Service running on port 3000
Database connected
```

This is actually a good DevOps concept for your assignment: **container startup order is not the same as application readiness**, so health checks are important.

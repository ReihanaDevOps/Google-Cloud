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

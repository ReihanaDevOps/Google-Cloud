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

into the Docker image.

Commands
docker build -t bloomworld-shop-service .
docker images

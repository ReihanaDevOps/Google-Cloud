Great. Let's automate creation of the **`shops` table**. 🚀

## Step 1 — Create a database initialization folder

In your root project folder, your structure currently is:

```text
bloomworld/
├── shop-service/
│   ├── src/
│   ├── Dockerfile
│   └── ...
└── docker-compose.yml
```

Create a new folder:

```text
bloomworld/
└── database/
```

Then inside it, create a file named:

```text
init.sql
```

Your structure should become:

```text
bloomworld/
├── shop-service/
│   ├── src/
│   ├── Dockerfile
│   └── ...
│
├── database/
│   └── init.sql    👈
│
└── docker-compose.yml
```

Create the folder and empty `init.sql` file, then tell me **done**.


Great 👍 Now we need to tell the PostgreSQL container to automatically run your `init.sql`.

## Step 3 — Update `docker-compose.yml`

In your `postgres` service, add a `volumes` section:

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

    volumes:
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql

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

## What this does

```text
Your Computer
database/init.sql
       │
       │ Docker bind mount
       ▼
PostgreSQL Container
/docker-entrypoint-initdb.d/init.sql
       │
       ▼
PostgreSQL automatically executes it
       │
       ▼
shops table created
```

### Important

PostgreSQL's official Docker image runs scripts inside:

```text
/docker-entrypoint-initdb.d/
```

**only when initializing a fresh database**.

Since your current PostgreSQL container already has an initialized database, we should recreate it to test this.

## Step 4 — Stop and remove the current containers

Press:

```text
Ctrl + C
```

Then run:

```powershell
docker compose down
```

Then start everything again:

```powershell
docker compose up --build
```

After PostgreSQL starts, you should see a log related to executing `init.sql`.

Then test:

```text
http://localhost:3000/api/shops
```

It should work even though the database starts fresh, because the `shops` table will now be created automatically.

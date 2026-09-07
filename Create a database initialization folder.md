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

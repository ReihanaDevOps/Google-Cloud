
npm install dotenv - This package allows Node.js to read variables from a .env file(After create .env file manually)
Great. Next, let's add **input validation** to make your API more production-ready. 👍

Right now, someone could send:

```json
{
  "name": "",
  "city": ""
}
```

or even:

```json
{}
```

We should validate the request before saving it to PostgreSQL.

# Step 5 — Install `express-validator`

Inside `shop-service`, run:

```powershell
npm install express-validator
```

This will help us validate API requests.

After it's installed, tell me **done**. Then we'll update the `POST` and `PUT` routes step by step.

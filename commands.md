
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

# Step 1 — Install `express-validator`

Inside `shop-service`, run:

```powershell
npm install express-validator
```

This will help us validate API requests.

After it's installed, tell me **done**. Then we'll update the `POST` and `PUT` routes step by step.

Great. Now we'll add validation step by step.

## Step 2 — Add validation to `POST /api/shops`

Open:

```text
src/routes/shopRoutes.js
```

First, add this import at the top:

```javascript
const { body, validationResult } = require("express-validator");
```

But actually, `validationResult` is better handled in the controller or middleware. Let's keep the route clean.

Use this instead:

```javascript
const { body } = require("express-validator");
```

Now update your routes like this:

```javascript
const express = require("express");
const router = express.Router();
const { body } = require("express-validator");

const {
  getAllShops,
  getShopById,
  createShop,
  updateShop,
  deleteShop
} = require("../controllers/shopController");

router.get("/", getAllShops);

router.get("/:id", getShopById);

router.post(
  "/",
  [
    body("name")
      .trim()
      .notEmpty()
      .withMessage("Shop name is required"),

    body("city")
      .trim()
      .notEmpty()
      .withMessage("City is required")
  ],
  createShop
);

router.put(
  "/:id",
  [
    body("name")
      .trim()
      .notEmpty()
      .withMessage("Shop name is required"),

    body("city")
      .trim()
      .notEmpty()
      .withMessage("City is required")
  ],
  updateShop
);

router.delete("/:id", deleteShop);

module.exports = router;
```

## What this does

Before the request reaches your controller:

```text
Client
   ↓
Validation
   ↓
Controller
   ↓
PostgreSQL
```

For example:

```json
{
  "name": "",
  "city": "Colombo"
}
```

will be detected as invalid.

### Important

We still need to **handle the validation errors** inside the controller. Otherwise, the validation rules run but the API won't properly return the error response.

Next, we'll update `shopController.js` to return clean validation errors.


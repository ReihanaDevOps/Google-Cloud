Files changed
1. shop-service/src/app.js


 const express = require("express");
+const cors = require("cors");
 
 const shopRoutes = require("./routes/shopRoutes");
 
 const app = express();
 
+app.use(cors());
 app.use(express.json());
2. shop-service/package.json — cors added to dependencies (via npm install cors --save)

3. shop-service/package-lock.json — lockfile updated automatically by npm to record the new cors package (and its own small dependency tree)

No frontend files were changed — App.tsx was already correct.

Why CORS was the problem
Your frontend and backend run on different origins:

Frontend (Vite dev server): http://localhost:5173
Backend (shop-service): http://localhost:3000
Different port = different origin, as far as a browser is concerned. When JavaScript in the browser (your fetch call in App.tsx) tries to call an API on a different origin, the browser enforces the Same-Origin Policy: it only allows the response through if the server explicitly says "this origin is allowed" via an Access-Control-Allow-Origin response header.

Your Express app had no such header — plain express() with no CORS middleware never adds it. So:

Browser sends the request, server responds fine (curl proved that — 200 OK with real data).
Browser inspects the response, sees no Access-Control-Allow-Origin header, and blocks the response from reaching your JS code, throwing a CORS error instead.
Your fetch call's catch block catches that error, shops stays [].
With shops empty, the app never renders any shop cards, so the "View Shop" button never appears in the DOM at all.
app.use(cors()) adds the Access-Control-Allow-Origin (and related preflight) headers to every response, telling the browser "any origin may read this" — which unblocks the fetch. I confirmed the fix with curl -H "Origin: http://localhost:5173" and got back Access-Control-Allow-Origin: * in the response headers.

As noted before — cors() with no arguments allows all origins, which is fine for local dev but should be locked down (cors({ origin: "<your-prod-frontend-url>" })) before you deploy this publicly.

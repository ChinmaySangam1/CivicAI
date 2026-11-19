# CivicAI

CivicAI is a civic reporting and accountability prototype based on `cheatsheet.md`. The demo includes a React operations dashboard, report submission, issue map, severity routing, a verification queue, citizen challenges, and an Express API with evidence hashing and guarded closure paths.

## Run locally

Requires Node.js 20 or later.

```sh
npm install
npm run dev
```

The frontend opens at http://localhost:5173. To run the API in another terminal:

```sh
Copy-Item .env.example .env
npm run server
```

The API listens on port 4000. Without MongoDB it uses in-memory storage, which is suitable for a local demo and is cleared when the server stops. Set `MONGODB_URI` for persistent storage. Set `GEMINI_API_KEY` to enable Gemini Vision analysis and before/after comparison. The default model is `gemini-3.8-flash`; override it with `GEMINI_MODEL`. Without a key, issue scoring uses a deterministic, explainable fallback and repair evidence goes to human review. City department names are prototype defaults and should be configured for the target municipality.

## Prototype routes

- `GET /api/health`, `/api/departments`, `/api/complaints` (public responses omit photos, user IDs, and precise location)
- `POST /api/auth/register`, `/api/auth/login`, `GET /api/auth/me`
- `POST /api/auth/bootstrap-admin` (one-time key protected) and `POST /api/auth/provision` (super-admin only)
- `POST /api/analyze` — optional JSON `image` data URL plus description and category
- `POST /api/complaints` — creates a complaint and checks nearby same-category reports before merging; original evidence and merge history are retained
- `POST /api/complaints/:id/assign` — signed OFFICER, ADMIN, or SUPER_ADMIN session required
- `POST /api/complaints/:id/repair-evidence` — signed CONTRACTOR, OFFICER, ADMIN, or SUPER_ADMIN session required, stores the after photo, SHA-256 hash and GPS offset; server-side image comparison selects auto-close, challenge window, or human review
- `POST /api/complaints/:id/challenges` — signed CITIZEN session can challenge a pending closure during its 24-hour window
- `POST /api/complaints/:id/verification-decision` — authorized review path for final states

## Trust and safety notes

Severity is calculated by the backend and capped at 100; the AI does not choose the final score. Prototype SLA values are not official service standards. GPS proximity alone is not sufficient to merge; the API also requires a matching category, time window, and textual overlap or an exact image hash. A hash protects file integrity after receipt but does not prove where or when a photo was taken. The prototype now uses bcrypt password hashes and signed JWT role checks, but still needs deployment key management, rate limits, and a production identity review. The dashboard includes seeded demonstration records and loads API records when available. Offline reports are queued in browser storage and sent when connectivity returns. Do not expose it to real citizen data until authentication, authorization, privacy controls, rate limiting, persistent audit logging and secure media storage are configured.

The live map uses Leaflet with OpenStreetMap tiles and attribution. Repair verification compares the original and after-repair images on the server with Gemini Vision when GEMINI_API_KEY is configured. The client cannot submit its own confidence score. Missing GPS, uncertain image comparison, or missing AI configuration routes the case to human review. Report photo uploads are kept in the prototype record as data URLs; use dedicated private object storage before production.

## Accounts and staff access

Citizen accounts are created from the dashboard sign-in button. Passwords are hashed with bcrypt and sessions use signed JWTs. Set `JWT_SECRET` to a long random value for persistent deployments. If omitted, the API generates an ephemeral key on startup, so tokens expire when the server restarts.

Staff roles cannot be self-registered. Set a one-time `ADMIN_BOOTSTRAP_KEY` in the API environment, then call `POST /api/auth/bootstrap-admin` with that value in the `x-admin-bootstrap-key` header and a name, email, and 12-character password. After the first super admin exists, use `POST /api/auth/provision` with its bearer token to create OFFICER, CONTRACTOR, ADMIN, or additional SUPER_ADMIN accounts. Assignment, repair uploads, challenges, and final decisions check the signed account role instead of trusting a role sent in the request body.

Unauthenticated complaint lists contain approximate coordinates and omit evidence and personal identifiers. Signed staff sessions can see the full operations record; citizen sessions only see reports they submitted or corroborated.

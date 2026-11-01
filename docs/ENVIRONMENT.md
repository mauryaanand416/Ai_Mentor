# Environment and Dependency Matrix

This is the single reference for **which environment variables and external services each part of AI Mentor needs**. It consolidates the five committed `.env.example` files, the runtime validators, and the repository-root setup/development scripts.

Use this document with [`initalSetup.md`](../initalSetup.md) (step-by-step first-time setup). The committed `.env.example` files remain the source of truth for exact variable names.

- **Scope:** local development and deployment
- **Services covered:** `backend/`, `backendAdmin/`, `frontend/`, `frontendAdmin/`, `ai_service/backend/`
- **Verified against:** `upstream/main` at `817b3e8`

Legend used in the matrices:

| Symbol | Meaning                                                               |
| ------ | --------------------------------------------------------------------- |
| ✅     | Required — the service does not work without it                       |
| ⚪     | Optional — the related feature is disabled or falls back if missing   |
| 🔁     | Indirect — used through another service (for example, via AI service) |
| ➖     | Not used by that service                                              |

---

## 1. Services at a glance

| #   | Service     | Folder                | Runtime           | Default port | Health / readiness check       |
| --- | ----------- | --------------------- | ----------------- | ------------ | ------------------------------ |
| 1   | User API    | `backend/`            | Node.js / Express | 5000         | `http://localhost:5000/health` |
| 2   | Learner web | `frontend/`           | React / Vite      | 5173         | `http://localhost:5173`        |
| 3   | Admin API   | `backendAdmin/`       | Node.js / Express | 5001         | `http://localhost:5001/health` |
| 4   | Admin web   | `frontendAdmin/`      | React / Vite      | 5174         | `http://localhost:5174`        |
| 5   | AI service  | `ai_service/backend/` | Python / FastAPI  | 8000         | `http://127.0.0.1:8000/`       |

The Node APIs both connect to the **same PostgreSQL database**. The AI service has no database; it receives requests from the two APIs over HTTP.

---

## 2. Required tool checklist

These are the tools a contributor needs. `npm run setup` checks the tools marked **Required by setup**.

| Tool                   | Needed for                                    | Required by `npm run setup`   | Notes / verification                                          |
| ---------------------- | --------------------------------------------- | ----------------------------- | ------------------------------------------------------------- |
| **Node.js 18+**        | All four Node/React services and root scripts | ✅                            | CI uses Node.js 20. Check with `node --version`               |
| **npm**                | Dependency installation and root scripts      | ✅                            | Bundled with Node.js. Check with `npm --version`              |
| **Python 3.10+**       | AI service (`ai_service/backend/`)            | ✅                            | CI uses Python 3.11. Check with `python --version`            |
| **FFmpeg**             | AI video rendering and media processing       | ✅                            | Must be on `PATH`. Check with `ffmpeg -version`               |
| **Git**                | Clone, branches, commits                      | ➖ (not checked)              | Check with `git --version`                                    |
| **PostgreSQL or Neon** | Both Node APIs                                | ➖ (not checked)              | Local server on `5432`, or a hosted Neon database             |
| **Redis or Memurai**   | Queued AI video generation (`backend/`)       | ⚪ warning only               | Must listen on port `6379` (or set `REDIS_HOST`/`REDIS_PORT`) |
| **Gemini API key**     | All AI text generation in `ai_service`        | ✅ (name check only)          | Required; setup stops if `GEMINI_API_KEY` is absent           |
| **Groq API key**       | LLM fallback when Gemini fails                | ⚪                            | Optional; fallback is disabled without it                     |
| **Cloudinary account** | Image/video uploads and cloud delivery        | ⚪                            | Optional; local storage fallback is used                      |
| **D-ID account**       | AI avatar rendering                           | ⚪                            | Optional; FFmpeg fallback renderer is used                    |
| **Firebase project**   | Google login and learner auth config          | ⚪ feature / ✅ frontend keys | Six public `VITE_FIREBASE_*` values must exist                |
| **SMTP account**       | Password-reset and notification email         | ⚪                            | Can be disabled with `DISABLE_EMAIL=true`                     |
| **Stripe account**     | Card payments                                 | ⚪                            | Test keys are enough; can be disabled                         |
| **Razorpay account**   | Razorpay payments                             | ⚪                            | Test keys are enough; can be disabled                         |

`npm run setup` does **not** create a database and does **not** verify database connectivity. It verifies the AI `.env` file, the Gemini variable name, AI imports, and Redis availability.

---

## 3. External dependency matrix

| Dependency           | `backend` (User API)                                        | `backendAdmin` (Admin API)         | `frontend` (Learner web)         | `frontendAdmin` (Admin web) | `ai_service`                                      |
| -------------------- | ----------------------------------------------------------- | ---------------------------------- | -------------------------------- | --------------------------- | ------------------------------------------------- |
| **PostgreSQL**       | ✅ main datastore                                           | ✅ same database as `backend`      | ➖                               | ➖                          | ➖                                                |
| **Redis / Memurai**  | ⚪ queued video generation only (`REDIS_HOST`/`REDIS_PORT`) | ➖                                 | ➖                               | ➖                          | ➖ (Redis is used by the API, not the AI service) |
| **Gemini**           | 🔁 through AI service for AI routes                         | 🔁 through AI service for syllabus | ➖                               | ➖                          | ✅ primary LLM                                    |
| **Groq**             | 🔁 through AI service                                       | 🔁 through AI service              | ➖                               | ➖                          | ⚪ fallback LLM                                   |
| **Cloudinary**       | ⚪ uploads (disable with `DISABLE_CLOUDINARY=true`)         | ➖                                 | ➖                               | ➖                          | ⚪ cloud delivery; local files otherwise          |
| **D-ID**             | 🔁 through AI service                                       | ➖                                 | ➖                               | ➖                          | ⚪ avatar rendering; FFmpeg fallback              |
| **FFmpeg**           | ➖                                                          | ➖                                 | ➖                               | ➖                          | ✅ for video/TTS media pipeline                   |
| **Firebase Admin**   | ⚪ Google login verification                                | ➖                                 | ➖                               | ➖                          | ➖                                                |
| **Firebase Web SDK** | ➖                                                          | ➖                                 | ✅ required config at Vite start | ➖                          | ➖                                                |
| **SMTP**             | ⚪ email (disable with `DISABLE_EMAIL=true`)                | ➖                                 | ➖                               | ➖                          | ➖                                                |
| **Stripe**           | ⚪ payments (disable with `DISABLE_STRIPE=true`)            | ➖                                 | ⚪ checkout publishable key      | ➖                          | ➖                                                |
| **Razorpay**         | ⚪ payments (disable with `DISABLE_RAZORPAY=true`)          | ➖                                 | ⚪ checkout key ID               | ➖                          | ➖                                                |
| **AI service HTTP**  | ⚪ AI features; app boots without it                        | ⚪ syllabus feature                | ➖                               | ➖                          | ➖                                                |

**Important distinction:** a variable can be "optional for the product" but still checked by a startup validator. `backend` and `frontend` validate their variable sets at startup; `backendAdmin` and `frontendAdmin` do not.

---

## 4. Service-by-service environment matrix

Copy each template before starting:

```bash
cp backend/.env.example            backend/.env
cp backendAdmin/.env.example       backendAdmin/.env
cp frontend/.env.example           frontend/.env
cp frontendAdmin/.env.example      frontendAdmin/.env
cp ai_service/backend/.env.example ai_service/backend/.env
```

> The AI service file is `ai_service/backend/.env`, not `ai_service/.env`.

### 4.1 `backend/.env` — User API

Validated by [`backend/env-validator.js`](../backend/env-validator.js) at startup. In `NODE_ENV=production`, missing or placeholder values **abort startup**. In development they print a warning and the server starts anyway.

| Variable                                                                      | Status              | Notes                                                                  |
| ----------------------------------------------------------------------------- | ------------------- | ---------------------------------------------------------------------- |
| `PORT`                                                                        | ⚪ optional         | Defaults to `5000`; keep `5000` for the root runner                    |
| `LOG_LEVEL`                                                                   | ⚪ optional         | `fatal`…`trace`; defaults to `info`                                    |
| `JWT_SECRET`                                                                  | ✅ required         | Long random string; use the same value in `backendAdmin`               |
| `NEON_DATABASE_URL`                                                           | ✅ required         | PostgreSQL URL. Production must use `sslmode=verify-full`              |
| `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`                     | ⚪ optional         | Local-PostgreSQL fallback, used only when `NEON_DATABASE_URL` is empty |
| `DB_SSL_CA`, `DB_SSL_CA_FILE`                                                 | ⚪ optional         | Inline PEM or mounted CA file for verified TLS                         |
| `ALLOW_INSECURE_DB_SSL`                                                       | ⚪ development only | Must not be `true` in production; production throws                    |
| `DB_POOL_MAX`, `DB_POOL_MIN`, `DB_POOL_ACQUIRE`, `DB_POOL_IDLE`               | ⚪ optional         | Neon-friendly pool settings                                            |
| `FRONTEND_URL`                                                                | ✅ required         | CORS origin; `http://localhost:5173` locally                           |
| `AI_SERVICE_URL`                                                              | ✅ for AI features  | `http://127.0.0.1:8000`; AI routes fail without it                     |
| `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET`        | ✅ unless disabled  | All three, or set `DISABLE_CLOUDINARY=true`                            |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `FROM_EMAIL`, `FROM_NAME` | ✅ unless disabled  | All six, or set `DISABLE_EMAIL=true`                                   |
| `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`                                  | ✅ unless disabled  | Both, or set `DISABLE_STRIPE=true`                                     |
| `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`                                      | ✅ unless disabled  | Both, or set `DISABLE_RAZORPAY=true`                                   |
| `FIREBASE_PROJECT_ID`, `FIREBASE_CLIENT_EMAIL`, `FIREBASE_PRIVATE_KEY`        | ⚪ feature-required | Needed for Google login. Keep the literal `\n` in the private key      |
| `SUPER_ADMIN_NAME`, `SUPER_ADMIN_EMAIL`, `SUPER_ADMIN_PASSWORD`               | ⚪ seed only        | Used by `npm run seed:superadmin` in this folder                       |

**Disable switches** are supported by the validator but are **not present in `backend/.env.example`**. Add them manually when a feature is not needed locally:

```env
DISABLE_CLOUDINARY=true
DISABLE_EMAIL=true
DISABLE_STRIPE=true
DISABLE_RAZORPAY=true
```

If a payment route is called while its keys are missing, the payment service raises `Missing STRIPE_SECRET_KEY environment variable` or `Missing Razorpay environment variables: ...` at request time.

> These flags skip the **startup check** only. Code paths that actually use the integration still need real credentials: Cloudinary uploads, SMTP sending, and payment routes will fail when invoked. Use the flags to keep local startup clean when you do not touch those features.

> The validator checks `NEON_DATABASE_URL` even when the local `DB_*` fallback is used. In development you get a warning; in production startup is aborted. Use Neon, or accept the local warning.

### 4.2 `backendAdmin/.env` — Admin API

There is no environment validator here, so configuration problems surface as runtime errors (database failures, 401s, empty admin pages) rather than a startup banner.

| Variable                                                        | Status              | Notes                                                          |
| --------------------------------------------------------------- | ------------------- | -------------------------------------------------------------- |
| `PORT`                                                          | ⚪ optional         | Defaults to `5001`; keep `5001` for the root runner            |
| `FRONTEND_ADMIN_URL`                                            | ✅ required         | CORS origin for the admin dashboard; `http://localhost:5174`   |
| `JWT_SECRET`                                                    | ✅ required         | Use the same value as `backend/.env`                           |
| `NEON_DATABASE_URL`                                             | ✅ required         | Must point to the **same** database as `backend`               |
| `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`       | ⚪ optional         | Local-PostgreSQL fallback when `NEON_DATABASE_URL` is empty    |
| `DB_SSL_CA`, `DB_SSL_CA_FILE`                                   | ⚪ optional         | Verified-TLS CA configuration                                  |
| `ALLOW_INSECURE_DB_SSL`                                         | ⚪ development only | Must not be `true` in production                               |
| `AI_SERVICE_URL`                                                | ✅ for syllabus     | `http://127.0.0.1:8000`                                        |
| `AI_SERVICE_KEY`                                                | ✅ for syllabus     | Non-empty shared credential sent as the `x-service-key` header |
| `SYLLABUS_SERVICE_TIMEOUT_MS`                                   | ⚪ optional         | Defaults to `15000`                                            |
| `SUPER_ADMIN_NAME`, `SUPER_ADMIN_EMAIL`, `SUPER_ADMIN_PASSWORD` | ⚪ seed only        | Used by `npm run seed:superadmin` in this folder               |

Admin and user accounts are separate. The admin API signs its own JWTs; using the same `JWT_SECRET` keeps local setup consistent and is what the setup guide assumes.

### 4.3 `frontend/.env` — Learner web

Vite validates these at **dev-server start and at build time**. Missing or placeholder values call `process.exit(1)`; the app never starts.

| Variable                            | Status      | Notes                                                 |
| ----------------------------------- | ----------- | ----------------------------------------------------- |
| `VITE_API_BASE_URL`                 | ✅ required | `http://localhost:5000`                               |
| `VITE_FIREBASE_API_KEY`             | ✅ required | Firebase web app config                               |
| `VITE_FIREBASE_AUTH_DOMAIN`         | ✅ required | Firebase web app config                               |
| `VITE_FIREBASE_PROJECT_ID`          | ✅ required | Firebase web app config                               |
| `VITE_FIREBASE_STORAGE_BUCKET`      | ✅ required | Firebase web app config                               |
| `VITE_FIREBASE_MESSAGING_SENDER_ID` | ✅ required | Firebase web app config                               |
| `VITE_FIREBASE_APP_ID`              | ✅ required | Firebase web app config                               |
| `VITE_STRIPE_PUBLISHABLE_KEY`       | ⚪ optional | Required for Stripe checkout UI                       |
| `VITE_RAZORPAY_KEY_ID`              | ⚪ optional | Required for Razorpay checkout UI; must match backend |

Vite inlines `VITE_*` values at build time. Set them **before** `npm run build`, not only before `npm run dev`.

### 4.4 `frontendAdmin/.env` — Admin web

| Variable            | Status      | Notes                                                                                |
| ------------------- | ----------- | ------------------------------------------------------------------------------------ |
| `VITE_API_BASE_URL` | ✅ required | Must be `http://localhost:5001`. The committed example still shows `5000`; change it |

Only this variable is read. The admin dashboard calls `/api/...` through the Vite dev proxy, so a wrong target loads the UI but every page is empty. There is no validator; the failure is silent.

### 4.5 `ai_service/backend/.env` — AI service

| Variable                                                               | Status          | Notes                                                            |
| ---------------------------------------------------------------------- | --------------- | ---------------------------------------------------------------- |
| `GEMINI_API_KEY`                                                       | ✅ required     | `npm run setup` / `setup:ai` and service startup stop without it |
| `GEMINI_MODEL`                                                         | ⚪ optional     | Defaults to `gemini-2.5-flash`                                   |
| `LLM_TIMEOUT_SECONDS`                                                  | ⚪ optional     | Defaults to `60`                                                 |
| `GROQ_API_KEY`                                                         | ⚪ optional     | Enables the Gemini→Groq fallback                                 |
| `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` | ⚪ optional set | All three required together; otherwise local output is kept      |
| `DID_API_KEY`                                                          | ⚪ optional     | Enables D-ID avatar rendering                                    |
| `DID_API_URL`                                                          | ⚪ optional     | Defaults to `https://api.d-id.com`                               |
| `DID_SOURCE_URL`                                                       | ⚪ optional     | Public avatar image; otherwise FFmpeg fallback is used           |

The AI service's Cloudinary configuration is **independent** of `backend`'s Cloudinary configuration. Each service needs its own complete set if it uploads or delivers media to Cloudinary.

---

## 5. Shared values across services

| Value                                 | Used by                             | Must match?                                                                      |
| ------------------------------------- | ----------------------------------- | -------------------------------------------------------------------------------- |
| `NEON_DATABASE_URL`                   | `backend`, `backendAdmin`           | ✅ Yes — both APIs share one database                                            |
| `JWT_SECRET`                          | `backend`, `backendAdmin`           | ✅ Recommended — the setup guide asks for the same value                         |
| `AI_SERVICE_URL`                      | `backend`, `backendAdmin`           | ✅ Yes — both must reach the same AI service                                     |
| `AI_SERVICE_KEY`                      | `backendAdmin` → AI service         | ✅ Non-empty in `backendAdmin`; sent as `x-service-key`                          |
| `RAZORPAY_KEY_ID`                     | `frontend`, `backend`               | ✅ Same Razorpay account; frontend only needs the public key ID                  |
| Stripe keys                           | `frontend`, `backend`               | ✅ Same account **and mode** (`pk_test_` with `sk_test_`, etc.)                  |
| Cloudinary credentials                | `backend`, `ai_service`             | ⚪ Separate configs; both are optional but each needs a complete set             |
| `SUPER_ADMIN_*`                       | `backend` seed, `backendAdmin` seed | ⚪ Independent seed scripts; usually the same administrator account              |
| `FRONTEND_URL` / `FRONTEND_ADMIN_URL` | `backend` / `backendAdmin`          | ✅ Must match the browser origins (`5173` and `5174`)                            |
| Firebase public vs. Admin             | `frontend` / `backend`              | ✅ Same Firebase project; web config in the frontend, service account in the API |

`AI_SERVICE_KEY` is a shared credential between the Node services and the AI service. The current AI service does not verify the header itself, but `backendAdmin` refuses to send a syllabus request when the value is empty.

---

## 6. Minimum working configurations

### Profile A — all five services, no paid add-ons (recommended for contributors)

| Service         | Minimum configuration                                                                                                   |
| --------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `backend`       | `JWT_SECRET`, PostgreSQL (`NEON_DATABASE_URL` or `DB_*`), `FRONTEND_URL`, `AI_SERVICE_URL`, four `DISABLE_*=true` flags |
| `backendAdmin`  | `JWT_SECRET` (same), same PostgreSQL connection, `FRONTEND_ADMIN_URL`, `AI_SERVICE_URL`, `AI_SERVICE_KEY`               |
| `frontend`      | `VITE_API_BASE_URL` + six `VITE_FIREBASE_*` values (a free Firebase project)                                            |
| `frontendAdmin` | `VITE_API_BASE_URL=http://localhost:5001`                                                                               |
| `ai_service`    | `GEMINI_API_KEY` (free tier is enough)                                                                                  |

```env
# backend/.env — minimum without paid integrations
PORT=5000
JWT_SECRET=replace-with-a-long-random-string
FRONTEND_URL=http://localhost:5173
AI_SERVICE_URL=http://127.0.0.1:8000
NEON_DATABASE_URL=postgresql://user:password@host/db?sslmode=verify-full
DISABLE_CLOUDINARY=true
DISABLE_EMAIL=true
DISABLE_STRIPE=true
DISABLE_RAZORPAY=true
```

Expected limitations in this profile:

- `backend` Cloudinary uploads (for example avatar or seed media) are unavailable; the AI service keeps generated media locally instead of uploading it
- no password-reset email
- no payments
- no queued video generation without Redis/Memurai
- AI video rendering needs FFmpeg, which `npm run setup` requires anyway
- AI syllabus, quiz, flashcards, and explain-code still work (Gemini only; Groq fallback disabled)

**Local PostgreSQL instead of Neon:** create the database, set `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD` in both API `.env` files, and leave `NEON_DATABASE_URL` empty. The `backend` validator will warn in development because it still expects `NEON_DATABASE_URL`; the server starts and uses the `DB_*` fallback. In production, use a verified TLS connection string.

### Profile B — full local feature set

Add the following to Profile A:

- Redis or Memurai on `6379` (or `REDIS_HOST`/`REDIS_PORT`) for the video queue
- FFmpeg on `PATH` for video rendering
- real `SMTP_*` values (or keep `DISABLE_EMAIL=true`)
- Stripe/Razorpay test keys and matching `VITE_*` keys (or keep the disable flags)
- complete Cloudinary sets in `backend` and `ai_service` if cloud media is needed
- `GROQ_API_KEY` to enable the LLM fallback
- optional `DID_API_KEY`, `DID_SOURCE_URL` for avatar rendering

### Profile C — one service at a time

| Goal                      | Command                      | Still required                                         |
| ------------------------- | ---------------------------- | ------------------------------------------------------ |
| AI service only           | `npm run dev:ai`             | `ai_service/backend/.env` with Gemini, FFmpeg, venv    |
| User API only             | `npm run dev:backend`        | `backend/.env`, PostgreSQL                             |
| Learner web only          | `npm run dev:frontend`       | `frontend/.env`, plus a healthy User API on port 5000  |
| User experience together  | `npm run dev:user`           | Both user services' env files, PostgreSQL              |
| Admin API only            | `npm run dev:backend-admin`  | `backendAdmin/.env`, PostgreSQL                        |
| Admin web only            | `npm run dev:frontend-admin` | `frontendAdmin/.env`, plus a healthy Admin API on 5001 |
| Admin experience together | `npm run dev:admin`          | Both admin services' env files, PostgreSQL             |
| Everything                | `npm run dev`                | All five `.env` files                                  |

Standalone frontend modes check the matching API health endpoint before starting Vite.

---

## 7. Common setup failures

| Symptom                                                                 | Cause                                                               | Fix                                                                                 |
| ----------------------------------------------------------------------- | ------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| `Environment Variable Error — Frontend` and Vite exits                  | A `VITE_*` variable is missing or still starts with `your_`         | Fill `frontend/.env`; all seven required values must be real                        |
| Admin panel loads but every page is empty                               | `frontendAdmin/.env` still points at port `5000`                    | Set `VITE_API_BASE_URL=http://localhost:5001` and restart Vite                      |
| `Unable to connect: ... SequelizeConnectionRefusedError`                | Database unreachable or wrong URL                                   | Check `NEON_DATABASE_URL` / `DB_*`; confirm PostgreSQL is running                   |
| `Invalid production database TLS configuration ... sslmode=verify-full` | Production URL does not use verified TLS                            | Use `sslmode=verify-full`; do not rely on `ALLOW_INSECURE_DB_SSL` in production     |
| `Invalid database TLS configuration ...` in development                 | A non-verified `sslmode` is present                                 | Use `sslmode=verify-full`, or set `ALLOW_INSECURE_DB_SSL=true` for development only |
| `Missing STRIPE_SECRET_KEY environment variable`                        | Payment route used without a Stripe key                             | Add a test key or set `DISABLE_STRIPE=true`                                         |
| `Missing Razorpay environment variables: ...`                           | Razorpay route used without keys                                    | Add both keys or set `DISABLE_RAZORPAY=true`                                        |
| `Environment Variable Warning — Backend` in development                 | Missing/placeholder required values or local DB without Neon URL    | Fill the listed values; production aborts instead of warning                        |
| `Missing AI environment file`                                           | `ai_service/backend/.env` does not exist                            | Copy `ai_service/backend/.env.example` and add `GEMINI_API_KEY`                     |
| `GEMINI_API_KEY is not configured`                                      | Empty or placeholder Gemini key                                     | Add a real Gemini key, then rerun `npm run setup:ai`                                |
| `❌ GEMINI_API_KEY not found` at AI startup                             | Wrong `.env` location                                               | The file must be `ai_service/backend/.env`                                          |
| `ModuleNotFoundError: No module named 'config'`                         | Uvicorn started from `ai_service/` instead of `ai_service/backend/` | Start via `npm run dev:ai` or change into `ai_service/backend`                      |
| `FFmpeg not found` / setup aborts on FFmpeg                             | FFmpeg missing or not on `PATH`                                     | Install FFmpeg, restart the terminal, rerun `npm run setup`                         |
| Redis warning, or video jobs stay `queued`                              | Nothing listening on port `6379`                                    | Start Redis/Memurai, or set `REDIS_HOST`/`REDIS_PORT`, then restart `npm run dev`   |
| `Not allowed by CORS` in the browser                                    | `FRONTEND_URL` / `FRONTEND_ADMIN_URL` mismatch                      | Set them to `http://localhost:5173` / `http://localhost:5174`                       |
| Google login does nothing                                               | Missing/invalid Firebase Admin values                               | Check `FIREBASE_PRIVATE_KEY`; keep the literal `\n` characters                      |
| `AI service Python environment not found`                               | Venv missing or not detected                                        | Run `npm run setup:ai`, or set `AI_SERVICE_PYTHON` to the interpreter path          |
| `EADDRINUSE` / port already in use                                      | Another process owns the port                                       | Stop it, or let the runner reuse an already-healthy service                         |
| PowerShell blocks venv activation                                       | Execution policy                                                    | `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`              |

---

## 8. How the repository-root runner coordinates services

Two scripts in [`scripts/`](../scripts/) own the local workflow.

### 8.1 `npm run setup` — `scripts/install-dependencies.cjs`

1. Verifies `node`, `npm`, `python`, and `ffmpeg`; stops on a missing tool.
2. Installs the root workspace and all four Node/React projects (`npm ci` when a lockfile exists, otherwise `npm install`).
3. Creates or reuses the isolated Python environment at `ai_service/.venv` (override with `AI_SERVICE_VENV`).
4. Installs `ai_service/backend/requirements.txt` into that environment.
5. Validates that `ai_service/backend/.env` exists and contains a non-placeholder `GEMINI_API_KEY`.
6. Imports key AI modules to confirm the environment is usable.
7. Warns if nothing listens on Redis port `6379`.
8. Never creates, overwrites, or prints `.env` values.

Use `npm run setup:ai` to skip the Node installs and prepare only the Python environment.

### 8.2 `npm run dev` — `scripts/dev-runner.cjs`

| Command                      | Mode             | Services started                                    |
| ---------------------------- | ---------------- | --------------------------------------------------- |
| `npm run dev`                | `all`            | User API, learner web, Admin API, admin web, AI API |
| `npm run dev:user`           | `user`           | User API, learner web                               |
| `npm run dev:admin`          | `admin`          | Admin API, admin web                                |
| `npm run dev:backend`        | `backend`        | User API                                            |
| `npm run dev:frontend`       | `frontend`       | Learner web (after User API health check)           |
| `npm run dev:backend-admin`  | `backend-admin`  | Admin API                                           |
| `npm run dev:frontend-admin` | `frontend-admin` | Admin web (after Admin API health check)            |
| `npm run dev:ai`             | `ai`             | AI API                                              |

Runner behavior:

- **Ports are fixed in the script:** `5000`, `5173`, `5001`, `5174`, `8000`. If you change `PORT` in an `.env`, the runner's health checks and URLs no longer match. Keep the defaults for local development.
- **Readiness detection:** each service is considered ready when its startup marker appears. APIs are then health-checked (`/health` for the Node APIs, `/` for the AI service).
- **Timeout:** a service that is not ready within 120 seconds is marked failed and the runner stops the services it started.
- **Reuse:** selected services that are already healthy are marked "already running" and are not started again.
- **Shutdown:** `Ctrl+C` stops only processes started by the current terminal; reused services keep running.
- **AI interpreter resolution:** `AI_SERVICE_PYTHON`, then `ai_service/.venv/Scripts/python.exe` (Windows layout), then `python`. On macOS/Linux, point `AI_SERVICE_PYTHON` at `ai_service/.venv/bin/python`.
- **AI environment:** the runner sets `PYTHONUTF8=1` and `PYTHONIOENCODING=utf-8` for the Python service.

---

## 9. Deployment checklist

- Set `NODE_ENV=production` for both APIs. `backend` then treats missing or placeholder variables as fatal.
- Use a verified PostgreSQL connection: `sslmode=verify-full`, and provide `DB_SSL_CA`/`DB_SSL_CA_FILE` when the provider requires a CA. Never enable `ALLOW_INSECURE_DB_SSL` in production.
- Provide real `JWT_SECRET`, payment keys, SMTP credentials, Firebase Admin credentials, and Cloudinary credentials — or explicitly disable features you do not ship.
- Provide `AI_SERVICE_URL` (and, for the admin API, `AI_SERVICE_KEY`) from the deployed APIs to the deployed AI service.
- Set `REDIS_HOST` and `REDIS_PORT` when Redis is not on `127.0.0.1:6379`. These variables are read by `backend/queues/videoQueue.js` but are not in `.env.example`.
- Install FFmpeg on the AI service host or image, and ensure it is on `PATH`.
- Build each frontend with its `VITE_*` values set for the target environment. Vite values are compiled into the bundle.
- Match CORS origins exactly: `FRONTEND_URL` for the user API, `FRONTEND_ADMIN_URL` for the admin API.
- Store secrets in the platform's secret manager. Never commit `.env` files; CI runs secret scanning with gitleaks.

---

## 10. Verification checklist

| Check                    | Command / URL                                    | Expected                               |
| ------------------------ | ------------------------------------------------ | -------------------------------------- |
| Tools installed          | `node -v`, `python --version`, `ffmpeg -version` | Versions print without errors          |
| Dependencies installed   | `npm run setup`                                  | Ends with `[READY]`                    |
| AI environment validated | `npm run setup:ai`                               | `GEMINI_API_KEY=<configured>`          |
| User API alive           | `http://localhost:5000/health/live`              | `{"status":"alive"}`                   |
| User API ready           | `http://localhost:5000/health/ready`             | `{"status":"ready"}`                   |
| Admin API alive          | `http://localhost:5001/health/live`              | `{"status":"alive"}`                   |
| AI service alive         | `http://localhost:8000/`                         | Service running message                |
| AI service docs          | `http://localhost:8000/docs`                     | Swagger UI                             |
| Learner app              | `http://localhost:5173`                          | Login page                             |
| Admin app                | `http://localhost:5174`                          | Admin login page with data after login |
| No secrets staged        | `git status`                                     | No `.env` files                        |

---

## 11. Sources reviewed

- `backend/.env.example`, `backend/env-validator.js`, `backend/config/db.js`, `backend/queues/videoQueue.js`, `backend/services/paymentGatewayServices.js`
- `backendAdmin/.env.example`, `backendAdmin/config/db.js`, `backendAdmin/utils/aiService.js`
- `frontend/.env.example`, `frontend/vite.config.js`
- `frontendAdmin/.env.example`, `frontendAdmin/vite.config.js`
- `ai_service/backend/.env.example`, `ai_service/backend/config.py`, `ai_service/README.md`
- `scripts/install-dependencies.cjs`, `scripts/dev-runner.cjs`
- `package.json`, `README.md`, `initalSetup.md`

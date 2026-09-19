# Opsora SRE — Environment Variables & Cloud Configuration Guide

> **Audience:** DevOps Engineers, Platform Administrators  
> **Applicable Hosts:** Render, Vercel, AWS ECS, Self-Hosted Docker

---

## 1. Why You Did Not See Env Keys During Render Deployment

When deploying via **Render Blueprints** (`render.yaml`), Render automatically provisions and links infrastructure without requiring manual input:

1. **Auto-Generated Cryptographic Keys**: `APP_KEY` is provisioned automatically with `generateValue: true` and normalized into a 32-byte AES-256 key on startup by `docker-entrypoint.sh`.
2. **Managed Database Link**: `DATABASE_URL` is automatically wired from the managed PostgreSQL database instance (`opsora-db`) via `fromDatabase: connectionString`.
3. **Pre-configured Production Presets**: `DB_CONNECTION=pgsql`, `SESSION_DRIVER=cookie`, `CACHE_STORE=database`, and `QUEUE_CONNECTION=database` are applied directly from `render.yaml`.

Because all variables are pre-defined in the blueprint infrastructure code, Render starts the build immediately without displaying an interactive input questionnaire.

---

## 2. Where to View & Edit Environment Keys in Render

You can view, edit, or add environment variables at any time in the **Render Dashboard**:

1. Log in to [dashboard.render.com](https://dashboard.render.com).
2. Select your **`opsora-sre`** web service (or worker service).
3. In the left-hand navigation menu, click **Environment**.
4. Here you will see all active environment variables:
   - Click **Add Environment Variable** to add custom keys (e.g., SMTP or AWS S3).
   - Click the pencil icon or value box to update an existing variable.
   - Click **Save Changes** &rarr; Render will automatically trigger a rolling zero-downtime redeploy with the updated keys.

---

## 3. Complete Environment Variables Inventory

| Variable | Required? | Default / Example | Purpose & Notes |
|---|---|---|---|
| `APP_NAME` | Required | `"Opsora SRE"` | Platform name displayed on dashboards and notification emails |
| `APP_ENV` | Required | `production` | Application environment (`production` disables debug tools) |
| `APP_DEBUG` | Required | `false` | Must always be `false` in production to prevent stack traces |
| `APP_KEY` | Required | `base64:...` | 32-byte AES-256 encryption key for session cookies & encrypted data |
| `APP_URL` | Required | `https://opsora-sre.onrender.com` | Base URL used for email links and password reset tokens |
| `LOG_CHANNEL` | Required | `stderr` (Docker) / `stack` (Local) | Docker/Render streams logs directly to container standard error |
| `DB_CONNECTION` | Required | `pgsql` | Database driver (`pgsql` for PostgreSQL, `mysql` for MySQL) |
| `DATABASE_URL` | Required (PG) | `postgres://user:pass@host:5432/db` | Connection string automatically injected by Render PostgreSQL |
| `SESSION_DRIVER` | Required | `cookie` or `database` | Storage mechanism for user sessions |
| `SESSION_LIFETIME` | Optional | `120` | Inactivity timeout in minutes before requiring sign-in re-authentication |
| `CACHE_STORE` | Required | `database` | System health and rate-limiting cache backend |
| `QUEUE_CONNECTION` | Required | `database` | Asynchronous queue driver for email dispatch and audit logs |

---

## 4. Configuring Production Email Dispatch (SMTP / SES)

By default, local environments use `MAIL_MAILER=log`. For live deployments to dispatch handover notices, incident alerts, and password reset links to real mailboxes, add these variables in your Render **Environment** tab:

### Option A: Standard Enterprise SMTP (Gmail / Google Workspace, SendGrid, etc.)
```ini
MAIL_MAILER=smtp
MAIL_HOST=smtp.sendgrid.net
MAIL_PORT=587
MAIL_USERNAME=apikey
MAIL_PASSWORD=your_sendgrid_api_key
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=sre-alerts@your-company.com
MAIL_FROM_NAME="Opsora SRE"
```

### Option B: Amazon Simple Email Service (SES)
```ini
MAIL_MAILER=ses
AWS_ACCESS_KEY_ID=your_aws_key
AWS_SECRET_ACCESS_KEY=your_aws_secret
AWS_DEFAULT_REGION=us-east-1
MAIL_FROM_ADDRESS=sre-alerts@your-company.com
MAIL_FROM_NAME="Opsora SRE"
```

---

## 5. Synchronizing Variables Across Workers

In `render.yaml`, the background queue worker (`opsora-sre-worker`) and scheduler (`opsora-sre-scheduler`) automatically inherit `APP_KEY` and `DATABASE_URL` from the main web service.

If you add custom variables (such as `MAIL_*`) to the web service, be sure to also add them to the worker service if queue jobs handle background email dispatch.

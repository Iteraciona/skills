# Environment Variables Reference

The `.env` file is auto-created from `.env.example` during project scaffolding. Below is every variable with its purpose and default.

## Application

| Variable | Required | Default | Description |
|---|---|---|---|
| `ENVIRONMENT` | ✅ Yes | `DEVELOPMENT` | Must be `DEVELOPMENT` or `PRODUCTION`. Controls server behavior (DB strictness, banner color, host binding). |
| `DEFAULT_PORT` | ✅ Yes | `3001` | HTTP port the server listens on. |
| `ENDPOINT` | No | `localhost` | Production host/IP to bind the server. |
| `ENDPOINT_DEV` | No | `0.0.0.0` | Development host (0.0.0.0 = all interfaces). |

## Authentication & Security

| Variable | Required | Default | Description |
|---|---|---|---|
| `ACCESS_TOKEN` | No | — | Static access token for simple auth flows. |
| `JWT_KEY` | No | — | Secret key for signing JWTs. |
| `JWT_PASSPRHASE` | No | — | Additional passphrase for JWT. |
| `TOKEN_USER` | No | — | User for token-based basic auth. |
| `TOKEN_PASS` | No | — | Password for token-based basic auth. |

## MongoDB

| Variable | Required | Default | Description |
|---|---|---|---|
| `MONGO_USER` | ⚠️ * | — | Database username. |
| `MONGO_PASSWORD` | ⚠️ * | — | Database password. |
| `MONGO_PROTOCOL` | No | `mongodb+srv` | Connection protocol (`mongodb` or `mongodb+srv`). |
| `MONGO_SERVER` | ⚠️ * | — | Database server host (e.g., `cluster0.xxxxx.mongodb.net`). |
| `MONGO_PORT` | No | `27017` | MongoDB port (only used with `mongodb` protocol). |
| `MONGO_DB` | ⚠️ * | — | Database name. |
| `MONGO_AUTH_SOURCE` | No | — | Authentication database. |
| `MONGO_RETRY_WRITES` | No | — | Enable retry writes (`true`/`false`). |
| `MONGO_W` | No | — | Write concern. |
| `MONGO_APP_NAME` | No | — | App name for MongoDB connection. |

> ⚠️ * Required for database connectivity. In `DEVELOPMENT` mode, the server starts without these (with a warning). In `PRODUCTION` mode, missing values trigger a confirmation prompt.

## Email (Resend)

| Variable | Required | Default | Description |
|---|---|---|---|
| `RESEND_API_KEY` | No | — | API key from [Resend](https://resend.com). |
| `NO_REPLY_EMAIL_ADDRESS` | No | — | No-reply sender address. |
| `DEFAULT_SENDER` | No | — | Default sender name/email. |
| `DEFAULT_RECEIPT_ADDRESS` | No | — | Default recipient for system emails. |

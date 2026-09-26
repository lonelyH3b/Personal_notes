# Ghost Admin API JWT: Environment Variable Access in n8n

## Problem

While generating a JWT for the Ghost Admin API in an n8n Code node, accessing the API key with:

```js
const ghostAdminKey = $env.GHOST_ADMIN_API_KEY;
```

resulted in:

```text
Error: access to env vars denied
```

The API key itself was correctly defined in the Docker environment, but n8n was blocking Code nodes from accessing environment variables.

## Cause

n8n's `N8N_BLOCK_ENV_ACCESS_IN_NODE` configuration controls access to environment variables from expressions and Code nodes.

I had not explicitly configured it in my Docker Compose setup, so `$env.GHOST_ADMIN_API_KEY` was denied.

## Fix

Added the following environment variable to the n8n service in `docker-compose.yml`:

```yaml
environment:
  - GHOST_ADMIN_API_KEY=${GHOST_ADMIN_API_KEY}
  - N8N_BLOCK_ENV_ACCESS_IN_NODE=false
```

The actual API key remains in `.env`:

```env
GHOST_ADMIN_API_KEY=your_key_here
```

and `.env` is excluded from Git:

```gitignore
.env
```

After restarting the container, the Code node could access:

```js
const ghostAdminKey = $env.GHOST_ADMIN_API_KEY;
```

without exposing the permanent API key directly in the workflow code.

## Security Note

The important distinction is:

* **Don't hard-code the Ghost Admin API key in the workflow.**
* Keep the permanent secret in `.env`.
* Pass it into the Docker container through environment variables.
* Use `$env` inside n8n only when environment-variable access is intentionally enabled.
* Generate a short-lived JWT rather than sending the permanent Ghost Admin API key to the API.

The JWT in this workflow expires after 5 minutes:

```js
exp: now + 300
```

### Lesson

When `$env` produces `access to env vars denied` in an n8n Code node, check `N8N_BLOCK_ENV_ACCESS_IN_NODE` before assuming the environment variable itself is missing.

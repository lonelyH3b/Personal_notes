# Allowing Node.js Built-in Modules in n8n Code Nodes

## Problem

While working with an n8n **Code node**, I tried to use Node.js's built-in `crypto` module for hashing/signing:

```javascript
const crypto = require('crypto');
```

However, n8n returned an error indicating that the built-in function/module was not allowed.

The problem was **not with JavaScript or the `crypto` module itself**. n8n restricts access to Node.js built-in modules in Code nodes for security reasons.

---

## Fix for Self-Hosted n8n with Docker Compose

Since my n8n instance is running through **Docker Compose**, I needed to explicitly allow the `crypto` module through an environment variable.

In `docker-compose.yml`:

```yaml
services:
  n8n:
    image: n8nio/n8n
    environment:
      - NODE_FUNCTION_ALLOW_BUILTIN=crypto
```

Then recreate/restart the container so the environment variable takes effect:

```bash
docker compose down
docker compose up -d
```

After that, the Code node can use:

```javascript
const crypto = require('crypto');
```

---

## Why This Works

n8n does not automatically allow every Node.js built-in module inside its Code node.

The environment variable:

```text
NODE_FUNCTION_ALLOW_BUILTIN=crypto
```

tells n8n:

> Allow the Code node to access the Node.js built-in `crypto` module.

If multiple built-in modules are needed, they can be specified as a comma-separated list:

```yaml
- NODE_FUNCTION_ALLOW_BUILTIN=crypto,fs
```

Only enable the modules that are actually required.

---

## Important Lesson

The solution depends on **how n8n is deployed**.

Because I am running n8n with Docker Compose, the environment variable belongs in the Docker Compose configuration rather than being something I add directly inside the JavaScript code.

### General pattern

```text
n8n Code node
      ↓
Node.js built-in module blocked
      ↓
Allow required module through environment variable
      ↓
Docker Compose environment
      ↓
Restart/recreate n8n container
      ↓
Code node can use the module
```

## Takeaway

When n8n says a Node.js built-in module such as `crypto` is not allowed, **don't try to work around it by rewriting the JavaScript immediately**.

First check whether the required built-in module needs to be explicitly allowed through the n8n environment configuration.

For a Docker Compose deployment:

```yaml
environment:
  - NODE_FUNCTION_ALLOW_BUILTIN=crypto
```

Then recreate the container.

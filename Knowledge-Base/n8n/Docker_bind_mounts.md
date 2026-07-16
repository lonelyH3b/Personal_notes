# n8n + Docker: Understanding File Access and Bind Mounts

## Problem

I encountered the following error while trying to read a CSV file in n8n:

```text
Access to the file is not allowed.
Allowed paths: /home/node/.n8n-files
```

Initially, I thought the issue was with Docker, but the actual cause was n8n's filesystem security.

---

## Layer 1: Docker Bind Mount

A Docker container has its own isolated filesystem. It **cannot access files on the host machine unless they are mounted**.

For example:

```yaml
volumes:
  - ./data:/data
```

This creates the following mapping:

```
Host Machine                  Docker Container

./data          ----------->  /data
```

If the host contains:

```
project/
├── docker-compose.yml
└── data/
    └── data.csv
```

then the container sees:

```
/data/data.csv
```

The left side (`./data`) is always the **host path**, while the right side (`/data`) is the **container path**.

---

## Layer 2: n8n File Access Security

Even though Docker allows the container to see `/data/data.csv`, **n8n itself may refuse to access it**.

By default, recent versions of n8n only allow filesystem access within:

```
/home/node/.n8n-files
```

This is a security feature that prevents workflows from reading arbitrary files inside the container.

For example, without this restriction, a malicious workflow could attempt to read sensitive files like:

* `/etc/passwd`
* SSH keys
* Database credentials
* Environment files

To reduce this risk, n8n only allows access to trusted directories.

---

## Recommended Solution

Instead of mounting the host directory to `/data`, mount it directly to n8n's allowed directory:

```yaml
services:
  n8n:
    image: n8nio/n8n

    volumes:
      - ~/.n8n:/home/node/.n8n
      - ./data:/home/node/.n8n-files
```

Now the mapping becomes:

```
Host Machine                  Docker Container

./data          ----------->  /home/node/.n8n-files
```

If the host contains:

```
data/
└── data.csv
```

then the file can be accessed in n8n using:

```
/home/node/.n8n-files/data.csv
```

---

## Mental Model

There are **two separate permission layers**.

### Docker Layer

Question:

> Can the container see the file?

Solved by using a **bind mount**.

Example:

```
./data  --->  /home/node/.n8n-files
```

---

### n8n Layer

Question:

> Even if the container can see the file, is n8n allowed to access it?

Solved by using one of n8n's allowed directories (or configuring its filesystem allowlist).

---

## Key Takeaway

When working with files in Dockerized n8n:

1. Mount the host folder into the container using a bind mount.
2. Ensure the mount target is inside an n8n-approved directory (by default, `/home/node/.n8n-files`).
3. Always use the **container path** in n8n nodes, **not** the host path.

Remember:

* **Host path** → where the file exists on my computer.
* **Container path** → where the application inside Docker accesses the file.
* **n8n path restriction** → which container paths n8n is permitted to use.

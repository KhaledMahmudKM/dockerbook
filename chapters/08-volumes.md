# Data and Volumes

A container's writable layer disappears when the container is removed. That is exactly what you want for the container itself, but not for a database's files or a user's uploads. To keep data beyond a container's life, Docker gives you **volumes**, **bind mounts**, and **tmpfs mounts**. This chapter explains when to use each.

---

## The problem: the writable layer is ephemeral

Everything a running container writes goes into its thin writable layer (Chapter 2). Remove the container and that layer, and everything in it, is gone. Replace the container with a new version and you start from the image again, with no memory of what the old one stored.

So durable data must live **outside** the container's writable layer. That is what mounts provide: a path inside the container backed by storage that outlives the container.

---

## Three kinds of storage

<figure class="diagram">
<svg viewBox="0 0 720 290" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A container can mount a Docker-managed volume, a bind mount to a host directory, or a tmpfs mount held in memory." xmlns="http://www.w3.org/2000/svg">
<style>
.d8{ font-family:var(--sans),sans-serif; }
.d8 .cont{ fill:none; stroke:var(--accent); stroke-width:2; }
.d8 .inner{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.3; }
.d8 .store{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.d8 .vol{ fill:var(--panel); stroke:var(--accent); stroke-width:1.8; }
.d8 .h{ fill:var(--text); font-size:15px; font-weight:700; }
.d8 .s{ fill:var(--text-dim); font-size:13px; }
.d8 .cap{ fill:var(--text-faint); font-size:12.5px; }
.d8 .ar{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#d8a); }
</style>
<defs><marker id="d8a" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker></defs>
<g class="d8">
<rect x="30" y="40" width="190" height="210" rx="10" class="cont"/><text x="125" y="28" text-anchor="middle" class="h">Container</text>
<rect x="50" y="60" width="150" height="34" rx="6" class="inner"/><text x="125" y="82" text-anchor="middle" class="s">/var/lib/data</text>
<rect x="50" y="118" width="150" height="34" rx="6" class="inner"/><text x="125" y="140" text-anchor="middle" class="s">/app/src</text>
<rect x="50" y="176" width="150" height="34" rx="6" class="inner"/><text x="125" y="198" text-anchor="middle" class="s">/tmp/cache</text>
<rect x="400" y="52" width="300" height="50" rx="8" class="vol"/><text x="550" y="74" text-anchor="middle" class="h">Volume (Docker-managed)</text><text x="550" y="92" text-anchor="middle" class="s">managed by Docker · best for data</text>
<rect x="400" y="118" width="300" height="50" rx="8" class="store"/><text x="550" y="140" text-anchor="middle" class="h">Bind mount (host path)</text><text x="550" y="158" text-anchor="middle" class="s">/home/you/project · best for dev</text>
<rect x="400" y="184" width="300" height="50" rx="8" class="store"/><text x="550" y="206" text-anchor="middle" class="h">tmpfs (in memory)</text><text x="550" y="224" text-anchor="middle" class="s">RAM only · secrets, scratch</text>
<path class="ar" d="M200 77 L400 77"/>
<path class="ar" d="M200 135 L400 143"/>
<path class="ar" d="M200 193 L400 209"/>
<text x="360" y="276" text-anchor="middle" class="cap">A mount maps a path inside the container to storage that outlives it (volume, bind) or lives in RAM (tmpfs).</text>
</g>
</svg>
</figure>

### 1. Volumes: the default choice for data

A **volume** is storage that Docker creates and manages, stored in Docker's own area on the host. Volumes are the recommended way to persist application data (databases, uploads) because Docker handles them, they are easy to back up, and they are portable across containers.

```bash
docker volume create appdata        # create a named volume
docker run -d --name db \
  -v appdata:/var/lib/postgresql/data postgres   # mount it
docker volume ls                    # list volumes
docker volume inspect appdata       # where/what it is
docker volume rm appdata            # remove it
```

The syntax `-v NAME:/path/in/container` mounts a named volume. Remove and recreate the `db` container and the data in `appdata` is still there, because it never lived in the container.

### 2. Bind mounts: your host filesystem

A **bind mount** maps a specific **host directory** into the container. The container sees your real files, live. This is ideal for **development**: edit code on your machine and the running container sees the change immediately.

```bash
docker run -d -p 8000:8000 \
  -v "$(pwd)":/app myapp:1.0        # mount the current dir into /app
```

The giveaway is that the left side is a path (`$(pwd)`, `/home/you/project`) rather than a name. Bind mounts are powerful but tie the container to your host's layout, and a mount can shadow what the image shipped at that path, so prefer volumes for data and bind mounts for live source during development.

### 3. tmpfs mounts: memory only

A **tmpfs mount** lives in RAM and never touches disk. It vanishes when the container stops. Use it for sensitive scratch data (secrets you do not want written to disk) or fast temporary space:

```bash
docker run --tmpfs /app/cache myapp:1.0
```

---

## -v versus --mount

Two syntaxes do the mounting. `-v` is compact; `--mount` is explicit and self-documenting:

```bash
# -v : short form
docker run -v appdata:/data postgres

# --mount : verbose, keyword form (recommended for clarity)
docker run --mount type=volume,source=appdata,target=/data postgres
docker run --mount type=bind,source="$(pwd)",target=/app myapp:1.0
```

They are equivalent for common cases; `--mount` is preferred in scripts and documentation because each part is named.

---

## Cleaning up volumes

Volumes are **not** removed when their container is removed, by design, so you do not lose data by accident. Over time unused ("dangling") volumes accumulate:

```bash
docker volume ls                 # list all volumes
docker volume prune              # remove volumes not used by any container
```

> **`docker volume prune` deletes data.** It removes volumes no container currently references, which is often exactly the data you meant to keep between container rebuilds if nothing is running. Check `docker volume ls` first.

---

## Key Takeaways

- A container's writable layer is ephemeral; durable data must live in a mount.
- **Volumes** (Docker-managed) are the default for application data, portable and easy to back up.
- **Bind mounts** map a host directory in, ideal for live source code during development.
- **tmpfs** mounts live in RAM for secrets or scratch and vanish with the container.
- Volumes outlive their containers and are removed only explicitly. `docker volume prune` deletes unused ones, so check first.

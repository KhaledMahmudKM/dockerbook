# Docker Compose

Real applications are rarely one container. A web app needs a database; maybe a cache, a queue, a worker. Starting each with a long `docker run` command (in the right order, on the right network, with the right volumes) gets tedious and error-prone. **Docker Compose** replaces all of that with a single file and a single command.

---

## What Compose is

Compose lets you describe a multi-container application in one YAML file (`compose.yaml`) and manage the whole set together:

```bash
docker compose up -d       # start everything
docker compose down        # stop and remove everything
```

Compose is built into modern Docker as the `docker compose` subcommand (the older standalone `docker-compose` is the previous version). It reads the file, creates a network for the app, starts the services in dependency order, and wires them together.

---

## A complete example

A Python API plus a PostgreSQL database:

```yaml
services:
  api:
    build: .                     # build from the Dockerfile here
    ports:
      - "8000:8000"              # publish to the host
    environment:
      DATABASE_URL: postgres://app:secret@db:5432/app
    depends_on:
      - db

  db:
    image: postgres:16           # pull a ready-made image
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: app
    volumes:
      - dbdata:/var/lib/postgresql/data   # persist the database

volumes:
  dbdata:                        # a named volume, created automatically
```

Start it:

```bash
docker compose up -d
```

Compose creates a network, starts `db` and `api`, and, crucially, the `api` reaches the database at the hostname **`db`**, because Compose puts both services on the same network with name resolution (Chapter 9). That is why the `DATABASE_URL` says `@db:5432`.

<figure class="diagram">
<svg viewBox="0 0 720 258" style="width:100%;height:auto;max-width:720px" role="img" aria-label="One compose file defines several services that Compose starts together on a shared network, with a published port to the host and a named volume for the database." xmlns="http://www.w3.org/2000/svg">
<style>
.d10{ font-family:var(--sans),sans-serif; }
.d10 .file{ fill:var(--panel); stroke:var(--accent); stroke-width:2; }
.d10 .net{ fill:var(--bg-soft); stroke:var(--accent); stroke-width:1.8; }
.d10 .svc{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.d10 .vol{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.d10 .h{ fill:var(--text); font-size:15px; font-weight:700; }
.d10 .s{ fill:var(--text-dim); font-size:13px; }
.d10 .mono{ font-family:var(--mono),monospace; fill:var(--text-dim); font-size:12.5px; }
.d10 .cap{ fill:var(--text-faint); font-size:12.5px; }
.d10 .ar{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#d10a); }
</style>
<defs><marker id="d10a" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker></defs>
<g class="d10">
<rect x="20" y="70" width="140" height="60" rx="8" class="file"/><text x="90" y="96" text-anchor="middle" class="h">compose.yaml</text><text x="90" y="116" text-anchor="middle" class="s">one file</text>
<path class="ar" d="M160 100 L228 100"/><text x="194" y="91" text-anchor="middle" class="mono">up</text>
<rect x="230" y="34" width="300" height="150" rx="10" class="net"/><text x="380" y="26" text-anchor="middle" class="h">app network (auto-created)</text>
<rect x="255" y="80" width="110" height="60" rx="8" class="svc"/><text x="310" y="106" text-anchor="middle" class="h">api</text><text x="310" y="126" text-anchor="middle" class="s">built from .</text>
<rect x="395" y="80" width="110" height="60" rx="8" class="svc"/><text x="450" y="106" text-anchor="middle" class="h">db</text><text x="450" y="126" text-anchor="middle" class="s">postgres:16</text>
<path class="ar" d="M365 110 L395 110"/><text x="380" y="70" text-anchor="middle" class="mono">"db"</text>
<rect x="590" y="80" width="110" height="60" rx="8" class="vol"/><text x="645" y="106" text-anchor="middle" class="h">dbdata</text><text x="645" y="126" text-anchor="middle" class="s">volume</text>
<path class="ar" d="M505 110 L590 110"/>
<path class="ar" d="M310 184 L310 210"/><text x="322" y="204" class="mono">host :8000 → api:8000</text>
<text x="360" y="244" text-anchor="middle" class="cap">Compose starts all services on one network (name resolution), publishes ports, and creates named volumes.</text>
</g>
</svg>
</figure>

---

## The common Compose keys

| Key | Purpose |
|---|---|
| `services` | The containers that make up the app |
| `image` | Use a ready-made image for a service |
| `build` | Build the service's image from a Dockerfile |
| `ports` | Publish ports (`"host:container"`) |
| `environment` | Environment variables for the service |
| `volumes` | Mount volumes or bind mounts |
| `depends_on` | Start order (not readiness, see below) |
| `networks` | Attach services to specific networks |

Top-level `volumes:` and `networks:` declare named volumes and networks that Compose creates for you.

> **The `version:` field is obsolete.** Old Compose files began with `version: "3"`. Modern Compose ignores it and will warn if present. Just omit it and start the file with `services:`.

---

## Everyday Compose commands

```bash
docker compose up -d           # create and start, in the background
docker compose ps              # what's running in this project
docker compose logs -f         # follow logs from all services
docker compose logs -f api     # just one service
docker compose exec api bash   # shell into a running service
docker compose build           # (re)build services that use 'build'
docker compose down            # stop and remove containers + network
docker compose down -v         # also remove named volumes (deletes data!)
```

Run these from the folder containing `compose.yaml`. The project is named after that folder, so Compose knows which set of containers it owns.

---

## depends_on is order, not readiness

`depends_on` controls **start order**, not whether a dependency is *ready*. Compose will start `db` before `api`, but it does not wait for PostgreSQL to finish initializing and accept connections. For true readiness, add a **healthcheck** to the dependency and have the dependent wait on it, or make your application retry its connection on startup (the more robust habit).

---

## Why Compose matters

- **One command** brings up or tears down the whole stack.
- **Reproducible**: the file is the definitive description; commit it to Git.
- **Readable**: the architecture of your app is visible at a glance.
- **Shareable**: a teammate runs `docker compose up` and has the identical environment.

For a single-host app and for local development, Compose is the tool you will reach for constantly. (Multi-host orchestration is Kubernetes' domain, beyond this book.)

---

## Key Takeaways

- Compose describes a multi-container app in `compose.yaml` and manages it as one unit.
- `docker compose up -d` / `down` start and stop the whole stack; services reach each other by name on the auto-created network.
- Key keys: `services`, `image`/`build`, `ports`, `environment`, `volumes`, `depends_on`.
- Omit the obsolete `version:` field; start the file with `services:`.
- `depends_on` sets start order, not readiness: use healthchecks or retry logic for that.

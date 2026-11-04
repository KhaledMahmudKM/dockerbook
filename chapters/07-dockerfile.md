# Building Images with a Dockerfile

So far you have run images other people built. Now you build your own. A **Dockerfile** is a plain-text recipe: a list of instructions that Docker executes top to bottom to produce an image. Each instruction adds a layer, and understanding that is the key to writing good Dockerfiles.

---

## A first Dockerfile

Here is a complete Dockerfile for a small Python web application:

```dockerfile
# 1. Start from a base image
FROM python:3.12-slim

# 2. Set the working directory inside the image
WORKDIR /app

# 3. Copy dependency list and install (cached separately — see below)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 4. Copy the rest of the application
COPY . .

# 5. Document the port the app listens on
EXPOSE 8000

# 6. The command that runs when a container starts
CMD ["python", "app.py"]
```

Build it into an image, then run it:

```bash
docker build -t myapp:1.0 .      # the "." is the build context (this folder)
docker run -d -p 8000:8000 myapp:1.0
```

`-t myapp:1.0` tags the image; the `.` tells Docker the **build context** is the current directory.

---

## The common instructions

| Instruction | What it does |
|---|---|
| `FROM` | The base image to start from (first instruction) |
| `WORKDIR` | Set the working directory for later instructions |
| `COPY` | Copy files from the build context into the image |
| `ADD` | Like COPY, but also handles URLs and auto-extracts archives |
| `RUN` | Execute a command at **build** time (install packages, compile) |
| `ENV` | Set an environment variable in the image |
| `ARG` | A build-time variable (passed with `--build-arg`) |
| `EXPOSE` | Document which port the app listens on |
| `CMD` | Default command run at **container start** |
| `ENTRYPOINT` | The fixed executable; `CMD` becomes its arguments |
| `USER` | Switch to a non-root user for following instructions |
| `LABEL` | Add metadata (maintainer, version) |

Prefer `COPY` over `ADD` unless you specifically need ADD's URL/extract behavior — COPY is predictable.

### CMD vs ENTRYPOINT

Both define what runs when a container starts, and they are easy to confuse:

- **`CMD`** sets a default that is easy to override: `docker run myapp echo hi` replaces it.
- **`ENTRYPOINT`** sets the fixed program; anything after the image name becomes its arguments.

A common pattern combines them — `ENTRYPOINT ["python", "app.py"]` with `CMD ["--port", "8000"]` — so the container always runs your app, and `CMD` supplies default, overridable arguments.

---

## RUN vs CMD — build time vs run time

This trips up newcomers constantly:

- **`RUN`** happens while the image is being **built**. It bakes its result into a layer (installing packages, creating files).
- **`CMD`** / **`ENTRYPOINT`** happen when a **container starts** from the finished image. They are not run during the build.

So `RUN pip install ...` installs dependencies into the image once, at build time; `CMD ["python","app.py"]` launches the app every time a container starts.

---

## How a build becomes layers

Each instruction that changes the filesystem creates a new read-only layer, stacked on the base. The image is the sum of those layers.

<figure class="diagram">
<svg viewBox="0 0 720 300" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Each Dockerfile instruction adds a layer on top of the base image, producing the final image; unchanged layers are reused from the build cache." xmlns="http://www.w3.org/2000/svg">
<style>
.d7{ font-family:var(--sans),sans-serif; }
.d7 .instr{ fill:var(--panel); stroke:var(--border); stroke-width:1.3; }
.d7 .layer{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.d7 .base{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.d7 .final{ fill:none; stroke:var(--accent); stroke-width:2; }
.d7 .mono{ font-family:var(--mono),monospace; fill:var(--text-dim); font-size:11.5px; }
.d7 .h{ fill:var(--text); font-size:13px; font-weight:700; }
.d7 .s{ fill:var(--text-dim); font-size:11.5px; }
.d7 .cap{ fill:var(--text-faint); font-size:12px; }
.d7 .ar{ stroke:var(--text-dim); stroke-width:1.5; fill:none; marker-end:url(#d7a); }
.d7 .cache{ fill:var(--accent); font-size:11px; font-weight:600; }
</style>
<defs><marker id="d7a" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker></defs>
<g class="d7">
<text x="130" y="22" text-anchor="middle" class="h">Dockerfile</text>
<rect x="20" y="34" width="220" height="26" rx="5" class="instr"/><text x="30" y="51" class="mono">FROM python:3.12-slim</text>
<rect x="20" y="66" width="220" height="26" rx="5" class="instr"/><text x="30" y="83" class="mono">WORKDIR /app</text>
<rect x="20" y="98" width="220" height="26" rx="5" class="instr"/><text x="30" y="115" class="mono">COPY requirements.txt .</text>
<rect x="20" y="130" width="220" height="26" rx="5" class="instr"/><text x="30" y="147" class="mono">RUN pip install ...</text>
<rect x="20" y="162" width="220" height="26" rx="5" class="instr"/><text x="30" y="179" class="mono">COPY . .</text>
<rect x="20" y="194" width="220" height="26" rx="5" class="instr"/><text x="30" y="211" class="mono">CMD ["python","app.py"]</text>
<path class="ar" d="M250 125 L310 125"/>
<text x="500" y="22" text-anchor="middle" class="h">Image layers</text>
<rect x="400" y="30" width="200" height="26" rx="5" class="layer"/><text x="500" y="47" text-anchor="middle" class="s">metadata (CMD) — no fs change</text>
<rect x="400" y="60" width="200" height="26" rx="5" class="layer"/><text x="500" y="77" text-anchor="middle" class="s">app code</text>
<rect x="400" y="90" width="200" height="26" rx="5" class="layer"/><text x="500" y="107" text-anchor="middle" class="s">installed dependencies</text>
<rect x="400" y="120" width="200" height="26" rx="5" class="layer"/><text x="500" y="137" text-anchor="middle" class="s">requirements.txt</text>
<rect x="400" y="150" width="200" height="26" rx="5" class="base"/><text x="500" y="167" text-anchor="middle" class="s">python:3.12-slim (base)</text>
<rect x="394" y="24" width="212" height="158" rx="8" class="final"/>
<text x="500" y="200" text-anchor="middle" class="cache">▲ reused from cache if unchanged</text>
<text x="360" y="236" text-anchor="middle" class="cap">Each filesystem-changing instruction adds a layer. On rebuild, Docker reuses cached layers up to the first change.</text>
<text x="360" y="258" text-anchor="middle" class="cap">That is why dependencies are copied and installed before the app code.</text>
</g>
</svg>
</figure>

---

## The build cache — and instruction order

Docker caches each layer. On a rebuild, it reuses cached layers from the top **until it hits the first instruction whose inputs changed**; everything from there down is rebuilt.

This is why the sample Dockerfile copies `requirements.txt` and installs dependencies *before* copying the application code:

- Edit your app code and rebuild → only the `COPY . .` layer and below rebuild. The expensive `pip install` layer is **reused from cache**. Fast.
- If you instead did `COPY . .` first, every code change would invalidate the dependency install and reinstall everything. Slow.

**Order instructions from least- to most-frequently-changed.** It is the single biggest lever on build speed.

> **BuildKit.** Modern Docker builds with **BuildKit** (the default since Docker 23.0). It builds independent stages in parallel, skips unused steps, and caches more cleverly. You get it automatically with `docker build`.

---

## The build context and .dockerignore

The `.` in `docker build -t myapp .` is the **build context** — the files Docker sends to the builder. Everything in that folder is sent, so a stray `node_modules/`, `.git/`, or large data file makes builds slow and images bloated.

Exclude them with a **`.dockerignore`** file (same idea as `.gitignore`):

```
.git
node_modules
*.log
__pycache__
.env
```

This keeps the context small and prevents secrets (like `.env`) from being copied into images by an over-broad `COPY . .`.

---

## Key Takeaways

- A **Dockerfile** is a recipe; `docker build -t name:tag .` turns it into an image.
- Know the core instructions, and the pairs that confuse: `COPY` vs `ADD`, `CMD` vs `ENTRYPOINT`, `RUN` (build time) vs `CMD` (run time).
- Each filesystem-changing instruction adds a **layer**; the image is the stack.
- Docker caches layers and rebuilds from the first change down — **order instructions least- to most-changed**, dependencies before code.
- Use a **`.dockerignore`** to keep the build context small and keep secrets out of images.

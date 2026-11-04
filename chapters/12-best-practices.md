# Best Practices

Docker is easy to use and easy to misuse. A few habits separate images that are small, fast, secure, and reproducible from ones that are bloated and fragile. This chapter collects the practices that matter most, with **multi-stage builds** as the centerpiece.

---

## Start from a small, specific base

The base image sets your floor for size and attack surface.

- **Prefer slim or minimal bases.** `python:3.12-slim` instead of `python:3.12`; `alpine`-based images where they fit. The difference is often hundreds of megabytes.
- **Pin the version.** `python:3.12-slim`, not `python:latest`. `latest` drifts (Chapter 6); a pinned tag keeps builds reproducible.
- **Consider distroless** for production — images with just your app and its runtime, no shell or package manager, so there is far less to exploit.

---

## Multi-stage builds: build fat, ship thin

Many apps need a heavy toolchain to *build* (compilers, dev headers, build tools) but almost none of it to *run*. A **multi-stage build** uses one stage to build and a second, clean stage that copies only the finished artifact — so the toolchain never ships.

```dockerfile
# ---- Stage 1: build ----
FROM golang:1.22 AS build
WORKDIR /src
COPY . .
RUN go build -o /app/server ./cmd/server   # needs the full Go toolchain

# ---- Stage 2: runtime ----
FROM gcr.io/distroless/base-debian12
COPY --from=build /app/server /server       # copy ONLY the binary
EXPOSE 8000
ENTRYPOINT ["/server"]
```

The final image contains the compiled binary and nothing else — no Go compiler, no source. It can drop from gigabytes to a few megabytes.

<figure class="diagram">
<svg viewBox="0 0 720 260" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A build stage with the full toolchain compiles the app; only the resulting binary is copied into a small final runtime image, leaving the toolchain behind." xmlns="http://www.w3.org/2000/svg">
<style>
.d12{ font-family:var(--sans),sans-serif; }
.d12 .big{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.d12 .small{ fill:var(--panel); stroke:var(--accent); stroke-width:2; }
.d12 .item{ fill:var(--panel); stroke:var(--border); stroke-width:1.2; }
.d12 .art{ fill:var(--panel); stroke:var(--accent); stroke-width:1.6; }
.d12 .h{ fill:var(--text); font-size:13px; font-weight:700; }
.d12 .s{ fill:var(--text-dim); font-size:11.5px; }
.d12 .cap{ fill:var(--text-faint); font-size:11.5px; }
.d12 .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#d12a); }
.d12 .x{ fill:var(--text-faint); font-size:11px; }
</style>
<defs><marker id="d12a" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker></defs>
<g class="d12">
<rect x="30" y="40" width="280" height="160" rx="10" class="big"/><text x="170" y="32" text-anchor="middle" class="h">Stage 1 · build  (large)</text>
<rect x="55" y="60" width="110" height="30" rx="5" class="item"/><text x="110" y="80" text-anchor="middle" class="s">compiler/SDK</text>
<rect x="175" y="60" width="110" height="30" rx="5" class="item"/><text x="230" y="80" text-anchor="middle" class="s">build tools</text>
<rect x="55" y="100" width="110" height="30" rx="5" class="item"/><text x="110" y="120" text-anchor="middle" class="s">source code</text>
<rect x="175" y="100" width="110" height="30" rx="5" class="item"/><text x="230" y="120" text-anchor="middle" class="s">dev headers</text>
<rect x="95" y="150" width="150" height="34" rx="6" class="art"/><text x="170" y="172" text-anchor="middle" class="s">compiled binary ✔</text>
<path class="ar" d="M250 167 C 330 167, 360 120, 430 120"/><text x="360" y="150" text-anchor="middle" class="s">COPY --from=build</text>
<rect x="430" y="70" width="260" height="110" rx="10" class="small"/><text x="560" y="62" text-anchor="middle" class="h">Stage 2 · runtime  (small)</text>
<rect x="470" y="105" width="180" height="40" rx="6" class="art"/><text x="560" y="130" text-anchor="middle" class="s">compiled binary + runtime</text>
<text x="560" y="165" text-anchor="middle" class="x">no compiler · no source · no build tools</text>
<text x="360" y="232" text-anchor="middle" class="cap">Only the artifact crosses into the final image — the toolchain stays behind and never ships.</text>
</g>
</svg>
</figure>

Multi-stage builds are the highest-leverage single practice for image size and security.

---

## Order layers for caching

From Chapter 7, but worth repeating because it is the biggest build-speed lever: **order instructions from least- to most-frequently-changed.** Copy and install dependencies before copying application code, so editing code does not invalidate the dependency-install layer.

Also **combine related `RUN` steps** so you do not leave intermediate cruft in separate layers:

```dockerfile
RUN apt-get update \
 && apt-get install -y --no-install-recommends curl ca-certificates \
 && rm -rf /var/lib/apt/lists/*
```

Cleaning the apt cache *in the same `RUN`* keeps it out of the layer entirely.

---

## Keep secrets out of images

An image is distributed and its layers are inspectable. **Never bake secrets into one.**

- Do not `COPY` a `.env`, private key, or password into an image. Even if a later layer deletes it, it remains in the earlier layer and can be extracted.
- Pass secrets at **run time** via environment variables, mounted files, or a secrets manager — not build time.
- Use a **`.dockerignore`** so a broad `COPY . .` cannot sweep in `.env`, `.git`, or keys.

---

## Run as a non-root user

By default a container's process runs as root inside the container, which widens the blast radius if it is compromised. Create and switch to an unprivileged user:

```dockerfile
RUN useradd --system appuser
USER appuser
```

Distroless "nonroot" variants and many official images make this easy. Combined with not publishing ports wider than necessary (Chapter 9), it meaningfully reduces risk.

---

## One concern per container

Design each container to run **one** main process — a web server, *or* a database, *not* both. It keeps images small, scaling independent, logs clean (one stream per container), and failures isolated. Multiple cooperating containers are what Compose (Chapter 10) is for.

---

## Scan and keep images current

- **Scan** images for known vulnerabilities (`docker scout`, or your registry's scanner) and rebuild on base-image updates.
- **Rebuild periodically** so security fixes in the base reach your image.
- **Add a healthcheck** so Docker (and Compose) can tell whether your app is actually serving, not just running:

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -fsS http://localhost:8000/health || exit 1
```

---

## A best-practices checklist

| Do | Why |
|---|---|
| Use slim/minimal, version-pinned base images | Smaller, fewer vulnerabilities, reproducible |
| Use multi-stage builds | Ship the artifact, not the toolchain |
| Order layers least- to most-changed; combine `RUN`s | Fast rebuilds, smaller layers |
| Add a `.dockerignore` | Small context, no secrets copied in |
| Keep secrets out of images; pass at run time | Images are distributable and inspectable |
| Run as a non-root user | Smaller blast radius |
| One main process per container | Simpler scaling, logging, failure isolation |
| Add a `HEALTHCHECK`; scan images | Know the app is healthy; catch known CVEs |

---

## Key Takeaways

- Start from **small, version-pinned** base images; consider distroless for production.
- **Multi-stage builds** ship only the artifact, leaving the build toolchain behind — the top practice for size and security.
- Order layers for caching and combine `RUN` steps to keep images small and builds fast.
- Never bake secrets into images; run as **non-root**; use a `.dockerignore`.
- Run one process per container, add healthchecks, and scan images for vulnerabilities.

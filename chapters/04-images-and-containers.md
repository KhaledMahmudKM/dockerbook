# Images and Containers

Two words do most of the work in Docker: **image** and **container**. Getting the distinction clear early makes everything else easy.

- An **image** is a read-only template — a packaged filesystem plus metadata describing how to run it. It is built once and does not change.
- A **container** is a running (or stopped) **instance** of an image, with its own thin writable layer on top.

The relationship is like a program on disk versus a process running it, or a class versus an object. One image can spawn many containers, each independent.

```
        image (template)              containers (instances)
   nginx:latest  ──────────►  ┌── web1  (running)
                              ├── web2  (running)
                              └── web3  (stopped)
```

Run `docker run nginx` three times and you have three containers from one image. Each has the same starting files (the shared read-only layers from Chapter 2) but its own writable layer, its own processes, and its own state.

---

## Where images come from

Images live in **registries** — servers that store and distribute them. The default public registry is **Docker Hub**, home to millions of images: official ones like `nginx`, `python`, `postgres`, and community ones.

The first time you reference an image Docker does not have locally, it **pulls** it from the registry. After that the image is cached on your machine and reused. You can also pull explicitly:

```bash
docker pull nginx          # download the nginx image
docker images              # list local images
```

Chapter 10 returns to registries — tagging, pushing your own images, and private registries.

---

## What happens when you run a container

`docker run` looks simple, but it sets several components in motion. Tracing it is the clearest way to understand Docker's architecture.

<figure class="diagram">
<svg viewBox="0 0 720 340" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The docker CLI sends the request to the daemon, which pulls the image from a registry if needed and asks containerd and runc to create the container using kernel namespaces and cgroups." xmlns="http://www.w3.org/2000/svg">
<style>
.d4{ font-family:var(--sans),sans-serif; }
.d4 .cli{ fill:var(--panel); stroke:var(--accent); stroke-width:2; }
.d4 .comp{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.d4 .reg{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.d4 .kernel{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.d4 .h{ fill:var(--text); font-size:13px; font-weight:700; }
.d4 .s{ fill:var(--text-dim); font-size:11.5px; }
.d4 .cap{ fill:var(--text-faint); font-size:12px; }
.d4 .ar{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#d4a); }
.d4 .ara{ stroke:var(--accent); stroke-width:1.8; fill:none; marker-end:url(#d4aa); stroke-dasharray:5 4; }
.d4 .num{ fill:var(--accent); font-size:11px; font-weight:700; }
</style>
<defs><marker id="d4a" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="d4aa" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker></defs>
<g class="d4">
<rect x="30" y="40" width="150" height="48" rx="8" class="cli"/><text x="105" y="60" text-anchor="middle" class="h">docker CLI</text><text x="105" y="77" text-anchor="middle" class="s">you type the command</text>
<rect x="30" y="140" width="150" height="150" rx="8" class="comp"/><text x="105" y="162" text-anchor="middle" class="h">dockerd</text><text x="105" y="179" text-anchor="middle" class="s">(daemon)</text>
<rect x="45" y="192" width="120" height="40" rx="6" class="comp"/><text x="105" y="216" text-anchor="middle" class="s">containerd</text>
<rect x="45" y="240" width="120" height="40" rx="6" class="comp"/><text x="105" y="264" text-anchor="middle" class="s">runc</text>
<rect x="250" y="140" width="180" height="150" rx="8" class="kernel"/><text x="340" y="165" text-anchor="middle" class="h">Linux kernel</text><text x="340" y="196" text-anchor="middle" class="s">namespaces</text><text x="340" y="216" text-anchor="middle" class="s">cgroups</text><text x="340" y="250" text-anchor="middle" class="s">→ container</text><text x="340" y="270" text-anchor="middle" class="s">process running</text>
<rect x="520" y="40" width="170" height="60" rx="8" class="reg"/><text x="605" y="66" text-anchor="middle" class="h">Registry</text><text x="605" y="84" text-anchor="middle" class="s">Docker Hub, etc.</text>
<path class="ar" d="M105 88 L105 140"/><text x="118" y="116" class="num">1</text>
<path class="ara" d="M180 150 C 360 120, 470 95, 518 80"/><text x="350" y="112" class="num">2 pull image (if needed)</text>
<path class="ar" d="M165 212 L250 200"/><text x="185" y="196" class="num">3</text>
<path class="ar" d="M165 260 L250 250"/><text x="185" y="280" class="num">4</text>
<text x="360" y="322" text-anchor="middle" class="cap">1 CLI → daemon  ·  2 daemon pulls image  ·  3 containerd prepares it  ·  4 runc creates namespaces + cgroups and starts the process</text>
</g>
</svg>
</figure>

Step by step:

1. **The CLI sends the request** to the Docker daemon (`dockerd`) over its API.
2. **The daemon pulls the image** from the registry if it is not already cached.
3. **`dockerd` hands off to `containerd`**, which manages the image and prepares a snapshot: the read-only image layers plus a new writable layer.
4. **`containerd` invokes `runc`**, which creates the namespaces and cgroups (Chapter 2) and starts your process inside them. A small *shim* process stays behind as the container's parent, so the container keeps running even if the daemon restarts.

The result is a running container: your process, isolated and resource-limited, on a layered filesystem. Everything you do later — ports, volumes, networks — is configuration layered onto this flow.

---

## Images are immutable; containers are disposable

Two habits of mind that Docker rewards:

- **Images don't change.** You never edit an image in place; you build a new one (Chapter 7). This makes images reliable artifacts — the `nginx:1.27` image is the same bytes everywhere, forever.
- **Containers are cheap and disposable.** Because all durable truth lives in the image (and in volumes, Chapter 8), you can throw a container away and start a fresh one at any time. "Cattle, not pets." If a container misbehaves, you usually delete it and `docker run` a new one rather than repairing it.

This is why changes made *inside* a running container — an edited file, an installed package — vanish when the container is removed, unless you captured them in an image or a volume. Chapter 5 shows this in practice.

---

## Key Takeaways

- An **image** is a read-only template; a **container** is a running instance of one, with its own writable layer.
- One image can produce many independent containers.
- Images come from **registries** (Docker Hub by default) and are cached locally after the first pull.
- `docker run` flows CLI → `dockerd` → `containerd` → `runc`, which uses kernel namespaces and cgroups to start the process.
- Treat images as immutable artifacts and containers as disposable; durable data belongs in images or volumes.

# Running Containers

This chapter is the hands-on core: starting containers with the options you will use daily, listing them, getting inside them, reading their logs, and removing them. Along the way you will see the container lifecycle that ties these commands together.

---

## docker run and its essential options

`docker run [options] IMAGE [command]` creates a container from an image and starts it. The options are most of the skill:

| Option | Meaning |
|---|---|
| `-d`, `--detach` | Run in the background and print the container ID |
| `-it` | Interactive + TTY, for a shell you type into |
| `--name NAME` | Give the container a friendly name |
| `-p HOST:CONTAINER` | Publish a container port to the host |
| `-e KEY=VALUE` | Set an environment variable |
| `-v NAME:/path` | Mount a volume (Chapter 8) |
| `--rm` | Delete the container automatically when it exits |
| `--restart POLICY` | Restart automatically (`no`, `on-failure`, `always`, `unless-stopped`) |

Two common shapes:

```bash
# A background web server, reachable at http://localhost:8080
docker run -d --name web -p 8080:80 nginx

# An interactive shell in a throwaway Ubuntu container
docker run -it --rm ubuntu bash
```

In the first, `-d` detaches and `-p 8080:80` maps host port 8080 to the container's port 80. In the second, `-it` gives you a shell and `--rm` cleans up on exit, so you can poke around and leave no trace.

---

## Listing containers

```bash
docker ps          # running containers
docker ps -a       # all containers, including stopped ones
```

`docker ps` shows the container ID, image, command, status, ports, and name. A container you ran without `--rm` that has exited still appears in `docker ps -a` until you remove it.

---

## The container lifecycle

A container moves through a small set of states. Knowing them explains what each command does.

<figure class="diagram">
<svg viewBox="0 0 720 260" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A container is created, then runs; it can be paused and unpaused, stopped and started again, and finally removed." xmlns="http://www.w3.org/2000/svg">
<style>
.d5{ font-family:var(--sans),sans-serif; }
.d5 .state{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.d5 .run{ fill:var(--panel); stroke:var(--accent); stroke-width:2; }
.d5 .gone{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; stroke-dasharray:5 4; }
.d5 .h{ fill:var(--text); font-size:15px; font-weight:700; }
.d5 .s{ fill:var(--text-dim); font-size:13px; }
.d5 .ar{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#d5a); }
.d5 .lbl{ fill:var(--accent); font-size:13px; font-weight:600; }
.d5 .cap{ fill:var(--text-faint); font-size:12.5px; }
</style>
<defs><marker id="d5a" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker></defs>
<g class="d5">
<rect x="20" y="92" width="110" height="46" rx="8" class="state"/><text x="75" y="113" text-anchor="middle" class="h">Created</text><text x="75" y="129" text-anchor="middle" class="s">docker create</text>
<rect x="200" y="92" width="120" height="46" rx="8" class="run"/><text x="260" y="113" text-anchor="middle" class="h">Running</text><text x="260" y="129" text-anchor="middle" class="s">docker start / run</text>
<rect x="200" y="12" width="120" height="42" rx="8" class="state"/><text x="260" y="32" text-anchor="middle" class="h">Paused</text><text x="260" y="48" text-anchor="middle" class="s">docker pause</text>
<rect x="430" y="92" width="120" height="46" rx="8" class="state"/><text x="490" y="113" text-anchor="middle" class="h">Stopped</text><text x="490" y="129" text-anchor="middle" class="s">(exited)</text>
<rect x="620" y="92" width="80" height="46" rx="8" class="gone"/><text x="660" y="113" text-anchor="middle" class="h">Removed</text><text x="660" y="129" text-anchor="middle" class="s">rm</text>
<path class="ar" d="M130 115 L200 115"/><text x="165" y="108" text-anchor="middle" class="lbl">run</text>
<path class="ar" d="M260 92 L260 54"/><text x="300" y="76" text-anchor="middle" class="lbl">pause</text>
<path class="ar" d="M248 54 L248 92" transform="translate(-36,0)"/><text x="196" y="76" text-anchor="middle" class="lbl">unpause</text>
<path class="ar" d="M320 109 L430 109"/><text x="375" y="102" text-anchor="middle" class="lbl">stop</text>
<path class="ar" d="M430 124 L320 124"/><text x="375" y="140" text-anchor="middle" class="lbl">start</text>
<path class="ar" d="M550 115 L620 115"/><text x="585" y="108" text-anchor="middle" class="lbl">rm</text>
<text x="360" y="192" text-anchor="middle" class="cap">A stopped container still exists (docker ps -a) and keeps its writable layer until you remove it.</text>
<text x="360" y="214" text-anchor="middle" class="cap">Removing a container discards its writable layer; the image it came from is untouched.</text>
</g>
</svg>
</figure>

The matching commands:

```bash
docker stop web        # graceful stop (SIGTERM, then SIGKILL)
docker start web       # start a stopped container again
docker restart web     # stop then start
docker pause web       # freeze its processes
docker unpause web     # resume
docker rm web          # remove a stopped container
docker rm -f web       # force-remove even if running
```

`stop` is graceful: it signals the process to shut down and waits before killing it. `kill` is immediate. A **stopped** container is not gone: it keeps its writable layer and still shows in `docker ps -a`, so you can `start` it again or inspect it. **Removing** it is what discards that writable layer.

---

## Getting inside a running container

To run a command, often a shell, inside a container that is already running, use `exec`:

```bash
docker exec -it web bash      # interactive shell in 'web'
docker exec web ls /usr/share/nginx/html
```

`exec` is for *running* containers. (To get a shell in an image that would otherwise exit immediately, start it interactively with `docker run -it IMAGE bash` instead.)

> **Changes inside a container are not saved to the image.** If you `exec` in and install a package or edit a file, that change lives only in the container's writable layer. Remove the container and it is gone. To make a change permanent, put it in a Dockerfile (Chapter 7); to persist data, use a volume (Chapter 8).

---

## Logs and inspection

```bash
docker logs web            # the container's stdout/stderr
docker logs -f web         # follow (stream) the logs
docker inspect web         # full JSON: config, networks, mounts, state
docker stats               # live CPU/memory/IO per container
docker top web             # processes running inside the container
```

`docker logs` is usually the first stop when something misbehaves. Containers are expected to log to stdout/stderr, and that is what you see here. `docker inspect` is the deep reference when you need a container's exact IP, mounts, or environment.

---

## A quick end-to-end session

```bash
docker run -d --name web -p 8080:80 nginx   # start
curl localhost:8080                          # it serves the nginx page
docker logs web                              # see the access log
docker exec -it web bash                     # look inside
docker stop web && docker rm web             # stop and clean up
```

That loop (run, use, inspect, remove) is the rhythm of working with containers.

---

## Key Takeaways

- `docker run` creates and starts a container; master `-d`, `-it`, `--name`, `-p`, `-e`, and `--rm`.
- `docker ps` lists running containers; `docker ps -a` includes stopped ones.
- Containers move through **created → running → (paused) → stopped → removed**; a stopped container still exists until you `rm` it.
- `docker exec -it` runs a shell in a running container; `docker logs` shows its output.
- Changes made inside a container are lost on removal unless captured in an image or a volume.

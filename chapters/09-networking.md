# Networking

Containers need to talk: to each other, to the host, and to the outside world. Docker gives each container its own network namespace (Chapter 2) and connects them through **networks**. This chapter covers the network types, how containers find each other, and how you expose them with published ports.

---

## The default bridge network

When you install Docker it creates a default network called `bridge`. Unless you say otherwise, every container attaches to it and gets a private IP on an internal subnet. Containers on the bridge can reach the outside world (via NAT through the host), and the host can reach them, but they are isolated from your LAN.

```bash
docker network ls            # list networks
docker network inspect bridge
```

On the **default** bridge, containers can reach each other only by IP address, since there is no name resolution. That is inconvenient, which is why you almost always create your own network instead.

---

## User-defined networks: containers by name

Create your own bridge network and containers on it get **automatic DNS**: they can reach each other by **container name**. This is the single most useful networking habit in Docker.

```bash
docker network create appnet

docker run -d --name db  --network appnet postgres
docker run -d --name api --network appnet myapi:1.0
```

Now the `api` container can connect to the database at the hostname **`db`**, and Docker resolves it to the right container IP. No IP addresses to track, and it keeps working when containers restart with new IPs.

<figure class="diagram">
<svg viewBox="0 0 720 266" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Containers on a user-defined bridge network reach each other by name; a published port maps a host port to a container port so outside clients can connect." xmlns="http://www.w3.org/2000/svg">
<style>
.d9{ font-family:var(--sans),sans-serif; }
.d9 .net{ fill:var(--bg-soft); stroke:var(--accent); stroke-width:2; }
.d9 .cont{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.d9 .host{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.d9 .world{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.d9 .h{ fill:var(--text); font-size:15px; font-weight:700; }
.d9 .s{ fill:var(--text-dim); font-size:13px; }
.d9 .mono{ font-family:var(--mono),monospace; fill:var(--text-dim); font-size:12.5px; }
.d9 .cap{ fill:var(--text-faint); font-size:12.5px; }
.d9 .ar{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#d9a); }
.d9 .ara{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#d9aa); }
</style>
<defs><marker id="d9a" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="d9aa" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker></defs>
<g class="d9">
<rect x="230" y="34" width="300" height="150" rx="10" class="net"/><text x="380" y="26" text-anchor="middle" class="h">user-defined network "appnet"</text>
<rect x="255" y="84" width="110" height="60" rx="8" class="cont"/><text x="310" y="110" text-anchor="middle" class="h">api</text><text x="310" y="130" text-anchor="middle" class="s">:8000</text>
<rect x="395" y="84" width="110" height="60" rx="8" class="cont"/><text x="450" y="110" text-anchor="middle" class="h">db</text><text x="450" y="130" text-anchor="middle" class="s">:5432</text>
<path class="ar" d="M365 114 L395 114"/><text x="380" y="72" text-anchor="middle" class="mono">connect to "db"</text>
<rect x="30" y="84" width="130" height="60" rx="8" class="world"/><text x="95" y="110" text-anchor="middle" class="h">Browser</text><text x="95" y="130" text-anchor="middle" class="s">on the host/LAN</text>
<rect x="230" y="212" width="300" height="40" rx="8" class="host"/><text x="380" y="237" text-anchor="middle" class="s">host port 8080 → api:8000</text>
<path class="ara" d="M160 114 C 196 114, 196 232, 228 232"/>
<path class="ara" d="M310 184 L360 210"/>
<text x="40" y="182" class="mono">localhost:8080</text>
<text x="556" y="100" class="cap">Inside the network:</text><text x="556" y="116" class="cap">by name.</text>
<text x="556" y="146" class="cap">From outside: via a</text><text x="556" y="162" class="cap">published host port.</text>
</g>
</svg>
</figure>

---

## Publishing ports to the outside

By default a container's ports are reachable only from within its Docker network. To let something **outside** Docker (your browser, another machine) connect, **publish** a port with `-p HOST:CONTAINER`:

```bash
docker run -d -p 8080:80 nginx
```

This maps port 8080 on the host to port 80 in the container, so `http://localhost:8080` reaches nginx. The two numbers are independent. `-p 8080:80` is common precisely because many apps listen on 80 or 8000 inside but you want a different, non-privileged port outside.

> **Publishing exposes to more than localhost.** By default `-p 8080:80` binds **all** host interfaces, so the container is reachable from other machines on your network, and Docker's firewall rules can take effect ahead of a host firewall like UFW. To restrict a port to the host only, bind it explicitly: `-p 127.0.0.1:8080:80`. (Recent Docker releases have tightened some of this, but binding to localhost is the reliable way to keep a service private.)

`EXPOSE` in a Dockerfile only *documents* a port; it does **not** publish it. Only `-p` (or Compose's `ports:`) actually opens a path from the host.

---

## The built-in network drivers

| Driver | Use |
|---|---|
| `bridge` | Default. Private network on a single host; the everyday choice |
| `host` | Container shares the host's network stack directly, no isolation, no port mapping needed |
| `none` | No networking at all, fully isolated |
| `overlay` | Spans multiple hosts (Docker Swarm / clusters) |
| `macvlan` | Gives a container its own MAC/IP on the physical LAN |

For almost all single-host work you use `bridge` (a user-defined one). `host` trades isolation for raw performance; `none` is for containers that must not touch the network.

---

## Managing networks

```bash
docker network create appnet         # create
docker network ls                    # list
docker network inspect appnet        # details: subnet, connected containers
docker network connect appnet web    # attach a running container
docker network disconnect appnet web # detach
docker network prune                 # remove unused networks
```

A container can be attached to several networks at once, a pattern used to separate, say, a front-end network from a database network so the web tier cannot reach the database directly.

---

## Key Takeaways

- Each container has its own network namespace; Docker connects them through **networks**.
- The default `bridge` has no name resolution: create a **user-defined** network so containers reach each other by **name**.
- Publish ports with `-p HOST:CONTAINER` to reach a container from outside Docker; `EXPOSE` only documents.
- `-p` binds all interfaces by default: use `-p 127.0.0.1:8080:80` to keep a service host-only.
- Drivers: `bridge` (default), `host` (no isolation), `none` (none), plus `overlay`/`macvlan` for advanced cases.

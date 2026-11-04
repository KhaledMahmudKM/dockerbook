# Registries and Image Distribution

An image is only useful if you can get it where it needs to run. **Registries** are the servers that store and distribute images — you have been pulling from one (Docker Hub) all along. This chapter shows the other direction: tagging, logging in, and **pushing** your own images, plus private and alternative registries.

---

## How distribution works

The flow is symmetrical. You **pull** an image to run it; you **push** an image to share it.

<figure class="diagram">
<svg viewBox="0 0 720 220" style="width:100%;height:auto;max-width:720px" role="img" aria-label="You build an image locally, push it to a registry, and others pull it from the registry to run it." xmlns="http://www.w3.org/2000/svg">
<style>
.d11{ font-family:var(--sans),sans-serif; }
.d11 .node{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.d11 .reg{ fill:var(--bg-soft); stroke:var(--accent); stroke-width:2; }
.d11 .h{ fill:var(--text); font-size:13px; font-weight:700; }
.d11 .s{ fill:var(--text-dim); font-size:11.5px; }
.d11 .cap{ fill:var(--text-faint); font-size:11.5px; }
.d11 .push{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#d11p); }
.d11 .pull{ stroke:var(--text-dim); stroke-width:1.8; fill:none; marker-end:url(#d11g); }
.d11 .lbl{ font-size:11.5px; font-weight:600; }
</style>
<defs><marker id="d11p" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="d11g" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker></defs>
<g class="d11">
<rect x="40" y="80" width="150" height="60" rx="8" class="node"/><text x="115" y="106" text-anchor="middle" class="h">Your machine</text><text x="115" y="126" text-anchor="middle" class="s">docker build</text>
<rect x="285" y="70" width="170" height="80" rx="10" class="reg"/><text x="370" y="104" text-anchor="middle" class="h">Registry</text><text x="370" y="124" text-anchor="middle" class="s">Docker Hub / private</text>
<rect x="550" y="80" width="150" height="60" rx="8" class="node"/><text x="625" y="106" text-anchor="middle" class="h">Any host</text><text x="625" y="126" text-anchor="middle" class="s">docker run</text>
<path class="push" d="M190 100 L285 100"/><text x="237" y="90" text-anchor="middle" class="lbl" fill="var(--accent)">push</text>
<path class="pull" d="M455 120 L550 120"/><text x="502" y="140" text-anchor="middle" class="lbl" fill="var(--text-dim)">pull</text>
<text x="370" y="192" text-anchor="middle" class="cap">Build once, push to a registry, pull anywhere — the registry is how images travel between machines.</text>
</g>
</svg>
</figure>

---

## Docker Hub: logging in and pushing

**Docker Hub** is the default public registry. To push, you need a (free) account and must log in:

```bash
docker login                 # prompts for your Docker Hub username/password or token
```

Then the push is three steps: **build**, **tag with your repository name**, **push**.

```bash
# 1. build an image
docker build -t myapp:1.0 .

# 2. tag it for your Docker Hub namespace (your username)
docker tag myapp:1.0 yourname/myapp:1.0

# 3. push
docker push yourname/myapp:1.0
```

The repository name must include your namespace (`yourname/...`) so the registry knows where it belongs. Anyone can now `docker pull yourname/myapp:1.0` (if the repo is public) or `docker run yourname/myapp:1.0`.

> **Use access tokens, not your password.** For `docker login` in scripts or CI, create a Docker Hub **access token** and use that instead of your account password. You can revoke a token without changing your password, and it can be scoped.

---

## Tags are how you version releases

Pushing is also where a tagging convention pays off. Publish several tags for one build so consumers can choose how precise to be:

```bash
docker tag myapp:1.0 yourname/myapp:1.0
docker tag myapp:1.0 yourname/myapp:1.4.2
docker tag myapp:1.0 yourname/myapp:latest
docker push yourname/myapp --all-tags
```

A consumer can then pin `yourname/myapp:1.4.2` for reproducibility, or track `yourname/myapp:1` for compatible updates. Remember that `latest` is just a convention (Chapter 6) — publish it, but do not rely on it for anything that must be reproducible.

---

## Public vs private repositories

A Docker Hub repository can be **public** (anyone can pull) or **private** (only you and those you grant access). Private repos keep proprietary images off the open internet; pulling one requires `docker login` with authorized credentials.

---

## Other registries

Docker Hub is one option among many. Any registry speaks the same protocol, so `push`/`pull` work identically once you tag with the registry's hostname:

| Registry | Typical reference |
|---|---|
| Docker Hub | `yourname/myapp:1.0` |
| GitHub Container Registry | `ghcr.io/yourname/myapp:1.0` |
| GitLab | `registry.gitlab.com/group/project:1.0` |
| AWS ECR / Google Artifact Registry / Azure ACR | cloud-specific hostnames |

To push to any of them: `docker login <registry-host>`, tag with that host in the name, and `docker push`.

### Running your own registry

You can host a private registry yourself with Docker's own `registry` image:

```bash
docker run -d -p 5000:5000 --name registry registry:2
docker tag myapp:1.0 localhost:5000/myapp:1.0
docker push localhost:5000/myapp:1.0
```

This is handy for air-gapped environments or a team's internal images. In production you would add TLS and authentication in front of it.

---

## A note on trust and scanning

Images you pull are code you will run. Reduce risk by preferring **official** and **verified** images, pinning versions or digests, and scanning images for known vulnerabilities — Docker's `docker scout` and similar tools report CVEs in an image's layers. Chapter 12 folds this into broader best practices.

---

## Key Takeaways

- Registries distribute images: **push** to share, **pull** to run.
- To publish to Docker Hub: `docker login`, tag as `yourname/repo:tag`, then `docker push`.
- Use **access tokens** (not your password) for logins, and publish meaningful version tags.
- Repositories can be public or private; the same `push`/`pull` works for GHCR, cloud registries, or a self-hosted `registry:2`.
- Treat pulled images as code — prefer official/verified images, pin versions, and scan for vulnerabilities.

# Working with Images

You now run containers comfortably. This chapter focuses on the images behind them: how to read an image's name, pull and list images, tag them, inspect their layers, and clean them up. Building your own images is the next chapter; here you master the ones that already exist.

---

## Anatomy of an image reference

An image is identified by a reference with up to four parts. Most of them have defaults, which is why short names like `nginx` work.

<figure class="diagram">
<svg viewBox="0 0 720 176" style="width:100%;height:auto;max-width:720px" role="img" aria-label="An image reference is registry slash repository colon tag, where registry and repository namespace default to Docker Hub and library, and the tag defaults to latest." xmlns="http://www.w3.org/2000/svg">
<style>
.d6{ font-family:var(--mono),monospace; }
.d6 .seg{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.d6 .sega{ fill:var(--panel); stroke:var(--accent); stroke-width:2; }
.d6 .t{ fill:var(--text); font-size:15px; font-weight:700; }
.d6 .lbl{ fill:var(--text-dim); font-size:13px; font-family:var(--sans),sans-serif; }
.d6 .def{ fill:var(--text-faint); font-size:13px; font-family:var(--sans),sans-serif; }
.d6 .cap{ fill:var(--text-faint); font-size:12.5px; font-family:var(--sans),sans-serif; }
</style>
<g class="d6">
<rect x="115" y="20" width="150" height="40" rx="6" class="seg"/><text x="190" y="45" text-anchor="middle" class="t">registry</text>
<text x="277" y="45" text-anchor="middle" class="t">/</text>
<rect x="290" y="20" width="200" height="40" rx="6" class="seg"/><text x="390" y="45" text-anchor="middle" class="t">namespace/repo</text>
<text x="502" y="45" text-anchor="middle" class="t">:</text>
<rect x="515" y="20" width="120" height="40" rx="6" class="sega"/><text x="575" y="45" text-anchor="middle" class="t">tag</text>
<text x="190" y="82" text-anchor="middle" class="lbl">where it lives</text><text x="190" y="100" text-anchor="middle" class="def">default: docker.io</text>
<text x="390" y="82" text-anchor="middle" class="lbl">which image</text><text x="390" y="100" text-anchor="middle" class="def">default namespace: library</text>
<text x="575" y="82" text-anchor="middle" class="lbl">which version</text><text x="575" y="100" text-anchor="middle" class="def">default: latest</text>
<text x="115" y="140" class="cap">nginx</text><text x="275" y="140" class="cap">→</text><text x="295" y="140" class="cap">docker.io/library/nginx:latest</text>
<text x="115" y="162" class="cap">ghcr.io/acme/api:1.4</text><text x="275" y="162" class="cap">→</text><text x="295" y="162" class="cap">a specific version of acme's api image on GitHub's registry</text>
</g>
</svg>
</figure>

- **`nginx`** expands to **`docker.io/library/nginx:latest`**: Docker Hub, the official `library` namespace, newest tag.
- **`python:3.12`** is the `python` official image at tag `3.12`.
- **`ghcr.io/acme/api:1.4`** is a specific image on GitHub's registry.

> **`latest` is just a tag, not "newest."** It is the default when you omit a tag, but it points to whatever the publisher last tagged as `latest`, which may be old. For anything you depend on, name a real version (`nginx:1.27`) so your results are reproducible.

### Digests: pinning exactly

A tag can be moved to point at new bytes. A **digest** cannot, because it is a content hash:

```bash
docker pull nginx@sha256:abc123...   # exactly these bytes, forever
```

Use a digest when you need an image to be byte-for-byte identical every time (CI, security-sensitive deployments).

---

## Pulling and listing

```bash
docker pull python:3.12     # download a specific image
docker images               # list local images
docker image ls             # same thing (newer form)
```

`docker images` shows the repository, tag, image ID, age, and size. The same underlying layers are shared between images, so the sizes shown are not simply additive on disk.

### Searching Docker Hub

```bash
docker search postgres      # find images on Docker Hub
```

Prefer **official images** (the `library` namespace, e.g. `postgres`) and **verified publishers**, which are maintained and scanned. Treat random community images the way you would any untrusted download.

---

## Tagging images

A **tag** is just a label pointing at an image. `docker tag` adds another name to an existing image and copies nothing:

```bash
docker tag nginx:latest myregistry.example.com/web:1.0
```

You tag images to give them meaningful versions and to prepare them for pushing to a registry (Chapter 10). One image can carry many tags at once.

---

## Looking inside an image

```bash
docker image inspect nginx      # full JSON metadata
docker history nginx            # the layers, newest on top
```

`docker history` is the practical window onto Chapter 2's layers: each row is one layer, created by one build instruction, with its size. It shows you *why* an image is as big as it is, which is invaluable when you start slimming your own images in Chapter 11.

---

## Removing images and reclaiming space

```bash
docker rmi python:3.12          # remove an image by name/ID
docker image rm python:3.12     # same thing
```

An image cannot be removed while a container is using it; remove the container first (or `docker rmi -f`). Over time, pulled and built images pile up, including untagged **dangling** images from rebuilds. Reclaim space with:

```bash
docker image prune             # remove dangling (untagged) images
docker image prune -a          # remove all images not used by a container
docker system df               # see what Docker is using on disk
```

> **`prune` deletes.** `docker image prune -a` removes every image no running or stopped container references. You will re-pull or rebuild them next time they are needed. Handy for reclaiming space, but know what it does before running it on a machine with images you cannot easily recreate.

---

## Key Takeaways

- An image reference is `registry/namespace/repo:tag`; `nginx` means `docker.io/library/nginx:latest`.
- `latest` is a default tag, not "newest": pin real versions (or digests) for reproducibility.
- `docker pull`/`docker images` fetch and list; `docker search` finds images on Docker Hub (prefer official/verified).
- `docker tag` adds names; `docker history` reveals an image's layers and sizes.
- `docker rmi` removes images; `docker image prune` reclaims space but deletes, so use it knowingly.

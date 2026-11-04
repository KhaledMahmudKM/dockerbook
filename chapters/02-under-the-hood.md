# Under the Hood: The Technology Behind Containers

Docker did not invent a new kind of isolation. It packaged, polished, and made usable a set of **Linux kernel features** that already existed. Understanding those features demystifies containers completely: once you see the pieces, "container" stops being magic and becomes a precise, almost simple idea.

A container is really just one or more ordinary Linux processes, with three things added:

1. **Namespaces** — so the process sees its own isolated world.
2. **Control groups (cgroups)** — so the process can only use the resources you allow.
3. **A union filesystem** — so the process gets a packaged, layered root filesystem.

Everything else Docker does is built on top of these three primitives.

---

## Process virtualization vs machine virtualization

It helps to place containers next to the older idea of virtual machines.

**Machine virtualization** emulates hardware. A *hypervisor* — KVM, Xen, VMware, Hyper-V — presents virtual CPUs, memory, and disks, and a full guest operating system boots on top. Tools like **libvirt** provide a common API and toolkit to manage these VMs (libvirt drives KVM/QEMU and others). The isolation boundary is the virtual machine, and it is strong because the guest has its own kernel.

**Process virtualization** — what containers do — adds no virtual hardware and no second kernel. The container's processes run *directly* on the host kernel; the kernel simply shows each container a restricted view of the system and caps what it can consume. There is nothing to boot and no hardware to emulate, which is why containers are so light and fast.

| | Machine virtualization (VMs) | Process virtualization (containers) |
|---|---|---|
| Boundary | Virtual machine | Isolated group of processes |
| Kernel | One per guest | Shared host kernel |
| Managed by | Hypervisor + e.g. libvirt | Container runtime (runc) |
| Isolation strength | Stronger (separate kernel) | Lighter (shared kernel) |
| Cost | GBs, seconds to boot | MBs, milliseconds to start |

> Because containers share the host **kernel**, Linux containers need a Linux kernel. That is why Docker on Windows and macOS quietly runs a small Linux VM — the containers still need Linux underneath.

---

## A short history: from LXC to runc

The kernel features came first. **LXC** (Linux Containers), released around 2008, was the first widely used userspace tooling to combine namespaces and cgroups into something resembling today's containers.

When Docker launched in 2013 it originally used LXC as its execution engine. In 2014 Docker replaced LXC with its own library, **libcontainer**, written in Go and talking to the kernel directly. libcontainer was later donated to the community and became **runc**, the small reference tool at the heart of the **OCI** (Open Container Initiative) standard. Today Docker runs containers through `containerd` and `runc` (Chapter 4 traces the full path), but every one of them ultimately just sets up namespaces and cgroups — exactly what LXC did.

---

## Namespaces: what a container *sees*

A **namespace** wraps a global system resource so that processes inside the namespace see their own isolated instance of it. The kernel provides several kinds, and a container gets its own set of each:

| Namespace | Isolates | So the container has its own… |
|---|---|---|
| **PID** | Process IDs | Process tree; its first process is PID 1 |
| **NET** | Network stack | Interfaces, IP addresses, routes, ports |
| **MNT** | Mount points | Filesystem layout and mounts |
| **UTS** | Hostname & domain | Hostname (so it can differ from the host) |
| **IPC** | Inter-process comms | Shared memory, semaphores, message queues |
| **USER** | User & group IDs | UID/GID range (root inside ≠ root outside) |

(There is also a **cgroup** namespace, which hides the host's cgroup layout.) Together these make the processes inside a container believe they have a machine to themselves — their own PID 1, their own `eth0`, their own hostname, their own root filesystem — while they are really just processes on the host.

<figure class="diagram">
<svg viewBox="0 0 720 320" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A container is a group of processes wrapped in namespaces that limit what it sees, and cgroups that limit what it can use, all on the shared host kernel." xmlns="http://www.w3.org/2000/svg">
<style>
.d2{ font-family:var(--sans),sans-serif; }
.d2 .outer{ fill:none; stroke:var(--accent); stroke-width:2; }
.d2 .panel{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.d2 .chip{ fill:var(--panel); stroke:var(--border); stroke-width:1.2; }
.d2 .proc{ fill:var(--panel-hover); stroke:var(--accent); stroke-width:1.5; }
.d2 .kernel{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.d2 .h{ fill:var(--text); font-size:14px; font-weight:700; }
.d2 .s{ fill:var(--text-dim); font-size:12px; }
.d2 .cap{ fill:var(--text-faint); font-size:12px; }
.d2 .lbl{ fill:var(--accent); font-size:11.5px; font-weight:600; }
</style>
<g class="d2">
<rect x="20" y="30" width="680" height="210" rx="10" class="outer"/>
<text x="36" y="24" class="lbl">CONTAINER = isolated processes</text>
<text x="40" y="58" class="h">Namespaces — what it sees</text>
<rect x="40" y="70" width="128" height="34" rx="6" class="chip"/><text x="104" y="92" text-anchor="middle" class="s">PID · process tree</text>
<rect x="178" y="70" width="128" height="34" rx="6" class="chip"/><text x="242" y="92" text-anchor="middle" class="s">NET · interfaces</text>
<rect x="316" y="70" width="128" height="34" rx="6" class="chip"/><text x="380" y="92" text-anchor="middle" class="s">MNT · filesystem</text>
<rect x="40" y="112" width="128" height="34" rx="6" class="chip"/><text x="104" y="134" text-anchor="middle" class="s">UTS · hostname</text>
<rect x="178" y="112" width="128" height="34" rx="6" class="chip"/><text x="242" y="134" text-anchor="middle" class="s">IPC · shared mem</text>
<rect x="316" y="112" width="128" height="34" rx="6" class="chip"/><text x="380" y="134" text-anchor="middle" class="s">USER · uid/gid</text>
<rect x="470" y="66" width="210" height="80" rx="8" class="proc"/><text x="575" y="100" text-anchor="middle" class="h">app processes</text><text x="575" y="122" text-anchor="middle" class="s">PID 1, children…</text>
<text x="40" y="182" class="h">cgroups — what it can use</text>
<rect x="40" y="194" width="150" height="32" rx="6" class="panel"/><text x="115" y="214" text-anchor="middle" class="s">CPU</text>
<rect x="200" y="194" width="150" height="32" rx="6" class="panel"/><text x="275" y="214" text-anchor="middle" class="s">memory</text>
<rect x="360" y="194" width="150" height="32" rx="6" class="panel"/><text x="435" y="214" text-anchor="middle" class="s">I/O</text>
<rect x="520" y="194" width="160" height="32" rx="6" class="panel"/><text x="600" y="214" text-anchor="middle" class="s">pids (count)</text>
<rect x="20" y="256" width="680" height="34" rx="8" class="kernel"/><text x="360" y="278" text-anchor="middle" class="h">Shared Linux kernel (one kernel, many containers)</text>
<text x="360" y="312" text-anchor="middle" class="cap">Namespaces limit what a container sees; cgroups limit what it can consume; both are kernel features.</text>
</g>
</svg>
</figure>

---

## Control groups: what a container *can use*

Namespaces hide things; **control groups (cgroups)** limit things. A cgroup accounts for and caps a process group's use of:

- **CPU** — shares or hard quotas
- **Memory** — a ceiling, beyond which the kernel kills or throttles the process
- **Block I/O** — disk read/write bandwidth
- **PIDs** — the number of processes it may create (a fork-bomb guard)

Without cgroups, one container could starve every other container and the host. With them, you can run `docker run --memory=256m --cpus=1.5 ...` and the kernel enforces it. Modern systems use **cgroups v2**, a cleaner unified hierarchy that is now the default on current distributions.

---

## Union filesystems: what a container *runs from*

The third pillar is storage. A container's root filesystem is not one directory — it is a **stack of layers** combined by a **union (overlay) filesystem**, usually OverlayFS.

- The image provides one or more **read-only layers** (the `lowerdir`s). These are shared between every container started from that image.
- Each container gets its own thin **writable layer** on top (the `upperdir`).
- The union filesystem presents the stack as a single **merged** view.

When a container reads a file, it comes from whichever layer has it. When a container *writes* to a file that lives in a read-only layer, the file is first **copied up** into the writable layer and changed there — the original image layer is never modified. This is **copy-on-write (CoW)**.

<figure class="diagram">
<svg viewBox="0 0 720 300" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Read-only image layers plus a per-container writable layer are merged into one view; writing a file from a lower layer copies it up into the writable layer." xmlns="http://www.w3.org/2000/svg">
<style>
.d3{ font-family:var(--sans),sans-serif; }
.d3 .ro{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.d3 .rw{ fill:none; stroke:var(--accent); stroke-width:2; }
.d3 .merged{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.d3 .h{ fill:var(--text); font-size:13.5px; font-weight:700; }
.d3 .s{ fill:var(--text-dim); font-size:12px; }
.d3 .cap{ fill:var(--text-faint); font-size:12px; }
.d3 .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#d3arrow); }
.d3 .lbl{ fill:var(--text-faint); font-size:11px; }
</style>
<defs><marker id="d3arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker></defs>
<g class="d3">
<text x="150" y="22" text-anchor="middle" class="h">Layer stack (one container)</text>
<rect x="40" y="36" width="220" height="34" rx="6" class="rw"/><text x="150" y="58" text-anchor="middle" class="s">Writable layer (upperdir) — per container</text>
<rect x="40" y="78" width="220" height="30" rx="6" class="ro"/><text x="150" y="98" text-anchor="middle" class="s">Image layer 3 — app code (RO)</text>
<rect x="40" y="114" width="220" height="30" rx="6" class="ro"/><text x="150" y="134" text-anchor="middle" class="s">Image layer 2 — dependencies (RO)</text>
<rect x="40" y="150" width="220" height="30" rx="6" class="ro"/><text x="150" y="170" text-anchor="middle" class="s">Image layer 1 — base OS files (RO)</text>
<text x="150" y="198" text-anchor="middle" class="lbl">read-only layers are shared by all containers of this image</text>
<path class="ar" d="M270 110 L430 110"/>
<text x="350" y="100" text-anchor="middle" class="s">union / overlay</text>
<rect x="440" y="60" width="240" height="90" rx="8" class="merged"/><text x="560" y="92" text-anchor="middle" class="h">Merged view</text><text x="560" y="114" text-anchor="middle" class="s">what the container sees:</text><text x="560" y="132" text-anchor="middle" class="s">one unified filesystem</text>
<path class="ar" d="M150 150 C 120 130, 120 95, 148 74"/>
<text x="95" y="120" text-anchor="middle" class="s" transform="rotate(-90 95 120)">copy-on-write</text>
<text x="360" y="246" text-anchor="middle" class="cap">Editing a file from a read-only layer copies it up into the writable layer; the image is never changed.</text>
<text x="360" y="268" text-anchor="middle" class="cap">Delete the container and its writable layer is gone; the shared image layers remain.</text>
</g>
</svg>
</figure>

This layered design is why Docker is so efficient:

- **Shared layers** — ten containers from the same image share one copy of the read-only layers; only their thin writable layers differ.
- **Fast starts** — starting a container just adds a new writable layer; nothing is copied.
- **Small updates** — rebuilding an image reuses unchanged layers from cache (Chapter 7).
- **Clean teardown** — deleting a container removes only its writable layer.

Chapter 5 and Chapter 6 return to layers from the practical side.

> **Storage drivers.** The component that stacks the layers is the *storage driver* (or, in recent Docker, a containerd *snapshotter*). `overlay2` has long been the default on Linux; Docker Engine 29.0 and later default fresh installs to the containerd image store. The concept — read-only layers plus a copy-on-write writable layer — is the same throughout.

---

## Putting the three together

A container, precisely, is: **a group of processes** (namespaces give them an isolated view), **resource-limited** (cgroups cap what they use), **running on a layered root filesystem** (a union filesystem gives them their files) — all on the **shared host kernel**. Docker's job is to make creating that combination as easy as `docker run`.

---

## Key Takeaways

- Containers are built from existing Linux kernel features, not a new form of virtualization.
- **Namespaces** isolate what a container *sees*: PID, NET, MNT, UTS, IPC, USER.
- **cgroups** limit what a container *can use*: CPU, memory, I/O, and process count.
- A **union/overlay filesystem** gives a container a stack of read-only image layers plus a writable layer, with **copy-on-write**.
- LXC first combined these; Docker used LXC, then libcontainer, now `runc`/OCI — but the primitives are unchanged.
- Because containers share the host kernel, Linux containers need Linux — hence the hidden VM on Windows and macOS.

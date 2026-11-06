# Introduction to Docker

Docker is a platform for building, shipping, and running applications in **containers**. A container bundles an application with its libraries, dependencies, and configuration into a single unit that runs the same way on a laptop, a server, or a cloud instance. "It works on my machine" stops being an excuse, because the machine travels with the app.

---

## The problem Docker solves

Software rarely runs alone. An application needs a specific language runtime, particular library versions, environment variables, and system packages. Moving it from a developer's laptop to a test server to production means recreating that environment every time, and small differences cause large failures.

Containers solve this by packaging the application *and its environment* together. The same container image runs identically everywhere Docker runs. Setup shrinks from a page of instructions to a single `docker run`.

---

## Containers vs virtual machines

The usual point of comparison is the virtual machine. Both isolate workloads, but they do it at different layers.

A **virtual machine** virtualizes *hardware*. A hypervisor runs multiple guest operating systems, each a full OS with its own kernel, on top of the host. That isolation is strong but heavy: every VM carries a complete operating system, takes gigabytes of disk and memory, and boots in the tens of seconds.

A **container** virtualizes the *operating system*. Containers share the host's kernel and isolate just the application's processes, filesystem, and network. There is no guest OS to boot, so a container starts in milliseconds and adds only megabytes of overhead.

<figure class="diagram">
<svg viewBox="0 0 720 380" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Each virtual machine packages an app, its libraries, and a full guest OS, and all VMs run on a hypervisor. Each container packages only an app and its libraries, and all containers share the host kernel through a container engine." xmlns="http://www.w3.org/2000/svg">
<style>
.d1{ font-family:var(--sans),sans-serif; }
.d1 .wrap{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:5 3; }
.d1 .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.d1 .gos{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.d1 .acc{ fill:none; stroke:var(--accent); stroke-width:2; }
.d1 .host{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.d1 .hw{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.d1 .h{ fill:var(--text); font-size:15px; font-weight:700; }
.d1 .s{ fill:var(--text-dim); font-size:13px; }
.d1 .tag{ fill:var(--accent); font-size:13px; font-weight:700; }
.d1 .cap{ fill:var(--text-faint); font-size:12.5px; }
</style>
<g class="d1">
<text x="182" y="24" text-anchor="middle" class="h">Virtual Machines</text>
<rect x="20" y="40" width="104" height="162" rx="8" class="wrap"/><text x="72" y="59" text-anchor="middle" class="tag">VM 1</text>
<rect x="28" y="68" width="88" height="36" rx="6" class="box"/><text x="72" y="91" text-anchor="middle" class="s">App A</text>
<rect x="28" y="110" width="88" height="36" rx="6" class="box"/><text x="72" y="133" text-anchor="middle" class="s">bins / libs</text>
<rect x="28" y="152" width="88" height="40" rx="6" class="gos"/><text x="72" y="177" text-anchor="middle" class="s">Guest OS</text>
<rect x="130" y="40" width="104" height="162" rx="8" class="wrap"/><text x="182" y="59" text-anchor="middle" class="tag">VM 2</text>
<rect x="138" y="68" width="88" height="36" rx="6" class="box"/><text x="182" y="91" text-anchor="middle" class="s">App B</text>
<rect x="138" y="110" width="88" height="36" rx="6" class="box"/><text x="182" y="133" text-anchor="middle" class="s">bins / libs</text>
<rect x="138" y="152" width="88" height="40" rx="6" class="gos"/><text x="182" y="177" text-anchor="middle" class="s">Guest OS</text>
<rect x="240" y="40" width="104" height="162" rx="8" class="wrap"/><text x="292" y="59" text-anchor="middle" class="tag">VM 3</text>
<rect x="248" y="68" width="88" height="36" rx="6" class="box"/><text x="292" y="91" text-anchor="middle" class="s">App C</text>
<rect x="248" y="110" width="88" height="36" rx="6" class="box"/><text x="292" y="133" text-anchor="middle" class="s">bins / libs</text>
<rect x="248" y="152" width="88" height="40" rx="6" class="gos"/><text x="292" y="177" text-anchor="middle" class="s">Guest OS</text>
<rect x="20" y="212" width="324" height="34" rx="6" class="acc"/><text x="182" y="234" text-anchor="middle" class="s">Hypervisor</text>
<rect x="20" y="254" width="324" height="34" rx="6" class="host"/><text x="182" y="276" text-anchor="middle" class="s">Host Operating System</text>
<rect x="20" y="296" width="324" height="34" rx="6" class="hw"/><text x="182" y="318" text-anchor="middle" class="s">Physical Hardware</text>
<text x="538" y="24" text-anchor="middle" class="h">Containers</text>
<rect x="376" y="88" width="104" height="114" rx="8" class="wrap"/><text x="428" y="106" text-anchor="middle" class="tag">Container 1</text>
<rect x="384" y="114" width="88" height="36" rx="6" class="box"/><text x="428" y="137" text-anchor="middle" class="s">App A</text>
<rect x="384" y="156" width="88" height="36" rx="6" class="box"/><text x="428" y="179" text-anchor="middle" class="s">bins / libs</text>
<rect x="486" y="88" width="104" height="114" rx="8" class="wrap"/><text x="538" y="106" text-anchor="middle" class="tag">Container 2</text>
<rect x="494" y="114" width="88" height="36" rx="6" class="box"/><text x="538" y="137" text-anchor="middle" class="s">App B</text>
<rect x="494" y="156" width="88" height="36" rx="6" class="box"/><text x="538" y="179" text-anchor="middle" class="s">bins / libs</text>
<rect x="596" y="88" width="104" height="114" rx="8" class="wrap"/><text x="648" y="106" text-anchor="middle" class="tag">Container 3</text>
<rect x="604" y="114" width="88" height="36" rx="6" class="box"/><text x="648" y="137" text-anchor="middle" class="s">App C</text>
<rect x="604" y="156" width="88" height="36" rx="6" class="box"/><text x="648" y="179" text-anchor="middle" class="s">bins / libs</text>
<rect x="376" y="212" width="324" height="34" rx="6" class="acc"/><text x="538" y="234" text-anchor="middle" class="s">Container Engine (Docker)</text>
<rect x="376" y="254" width="324" height="34" rx="6" class="host"/><text x="538" y="276" text-anchor="middle" class="s">Host Operating System (shared kernel)</text>
<rect x="376" y="296" width="324" height="34" rx="6" class="hw"/><text x="538" y="318" text-anchor="middle" class="s">Physical Hardware</text>
<text x="360" y="364" text-anchor="middle" class="cap">Each VM carries its own guest OS. Containers carry only the app and share one kernel.</text>
</g>
</svg>
</figure>

| | Virtual machine | Container |
|---|---|---|
| Isolates at | Hardware (hypervisor) | OS (kernel features) |
| Includes a guest OS | Yes, a full one | No, shares the host kernel |
| Start-up time | Tens of seconds | Milliseconds |
| Size | Gigabytes | Megabytes |
| Overhead per instance | High | Low |

Containers are not "lightweight VMs". They are a different mechanism. Chapter 2 opens up exactly how the kernel provides that isolation.

> **They are complementary.** In practice containers often run *inside* VMs, for example a cloud VM hosting dozens of containers. You get the hardware isolation of the VM boundary and the density and speed of containers within it.

---

## What you get when you install Docker

Installing Docker gives you a few cooperating pieces:

- **The Docker CLI** (`docker`): the command-line tool you type into.
- **The Docker daemon** (`dockerd`): a background service that does the real work of building images and running containers.
- **An image registry client**: to pull and push images from registries like Docker Hub.

You type a command; the CLI sends it to the daemon; the daemon makes it happen. Chapter 3 installs all of this, and Chapter 4 traces a `docker run` through the full architecture.

---

## Why Docker became the standard

- **Portability**: one image runs anywhere Docker runs.
- **Speed**: containers start in milliseconds, so iteration is fast.
- **Density**: many containers fit where a few VMs would.
- **Consistency**: development, testing, and production share the same image.
- **Ecosystem**: millions of ready-made images on Docker Hub, and an open standard (OCI) so images work across tools.

---

## Key Takeaways

- Docker packages an application with its environment into a portable **container**.
- Containers virtualize the **operating system** and share the host kernel; VMs virtualize **hardware** and each run a full guest OS.
- Containers are far smaller and faster to start than VMs, and the two are often combined.
- Installing Docker gives you a **CLI**, a **daemon**, and registry access.
- Docker's portability, speed, and ecosystem made containers the default way to ship software.

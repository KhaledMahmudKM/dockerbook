# Answers to Chapter Questions

Answers and short explanations for the **Check your understanding** questions at the end of each chapter. Try the questions first; each section links back to them.

## Introduction to Docker

**1. A. An application with its libraries, dependencies and configuration**

The same image runs the same way on a laptop, a server or a cloud instance.

**2. C. Hardware, through a hypervisor**

Each VM runs a full guest OS with its own kernel.

**3. B. The host's kernel**

They isolate processes, filesystem and network, but there is no guest OS to boot.

**4. D. Milliseconds**

A VM boots in tens of seconds; a container just starts a process.

**5. B. They are complementary: containers often run inside VMs**

A cloud VM hosting dozens of containers combines VM isolation with container density.

**6. D. The Docker daemon, dockerd**

You type a command; the CLI sends it to the daemon; the daemon makes it happen.

**7. A. They are a different mechanism from VMs, based on OS-level isolation**

Chapter 2 shows how the kernel provides the isolation.

**8. C. Portability: one image runs anywhere Docker runs**

Speed, density and consistency between development, testing and production also matter.

[Back to the questions](01-introduction.md#check-your-understanding)

## Under the Hood: The Technology Behind Containers

**1. D. Namespaces, cgroups and a union filesystem**

A container is ordinary processes plus these three things.

**2. B. PID**

NET isolates the network stack; MNT the mounts; UTS the hostname.

**3. D. UTS**

UTS isolates hostname and domain name.

**4. A. Limit and account for resources such as CPU, memory, I/O and number of processes**

Namespaces hide things; cgroups limit things.

**5. C. The file is copied up into the container's writable layer and changed there**

This is copy-on-write; the image layer is never modified.

**6. D. They share the read-only image layers; only their thin writable layers differ**

Shared layers also make starts fast.

**7. B. LXC**

Docker replaced it with libcontainer in 2014, which became runc.

**8. C. Linux containers share the host kernel, so they need a Linux kernel**

The VM is hidden in day-to-day use.

[Back to the questions](02-under-the-hood.md#check-your-understanding)

## Installing Docker

**1. A. Docker Desktop**

Docker Desktop bundles Engine, CLI, Compose and a GUI, and manages the Linux VM.

**2. C. WSL 2**

Enable WSL 2 first if prompted.

**3. D. From Docker's apt or yum repositories, so upgrades flow through the package manager**

The convenience script is ideal for labs and getting started.

**4. B. docker run hello-world**

It pulls a tiny image and prints "Hello from Docker!".

**5. C. docker info**

These are the mechanisms from Chapter 2.

**6. A. It grants root-equivalent access to the host**

A user in the group can start a container that mounts the whole filesystem; rootless mode is an alternative.

**7. A. A starter Dockerfile, .dockerignore and compose.yaml for a project**

It asks a few questions and writes idiomatic files for your language.

**8. C. Through its own GUI**

Docker Engine on Linux upgrades through the package manager.

[Back to the questions](03-installation.md#check-your-understanding)

## Images and Containers

**1. B. A read-only template: a packaged filesystem plus metadata on how to run it**

It is built once and does not change.

**2. C. A running or stopped instance of the image, with its own writable layer**

Like a process running a program, or an object of a class.

**3. A. The default public registry**

It hosts official images such as nginx, python and postgres.

**4. A. Docker pulls it from the registry and caches it**

Later runs reuse the cached image.

**5. C. runc**

The CLI talks to dockerd, which hands off to containerd, which invokes runc.

**6. B. So the container keeps running even if the daemon restarts**

The container does not depend on dockerd staying up.

**7. D. It is lost with the container's writable layer**

Capture changes in a Dockerfile, or data in a volume.

**8. B. Containers are disposable: replace a misbehaving one instead of repairing it**

Durable truth lives in the image and in volumes.

[Back to the questions](04-images-and-containers.md#check-your-understanding)

## Running Containers

**1. A. -d**

It prints the container ID and returns your prompt.

**2. C. Maps host port 8080 to the container's port 80**

The format is HOST:CONTAINER.

**3. B. Deletes the container automatically when it exits**

Good for throwaway interactive sessions.

**4. D. docker ps -a**

A container run without --rm stays in docker ps -a until you remove it.

**5. B. stop signals the process to shut down and waits; kill ends it immediately**

A stopped container keeps its writable layer and can be started again.

**6. D. docker exec -it NAME bash**

exec is for running containers; docker run starts a new one.

**7. A. To stdout and stderr**

docker logs is usually the first stop when something misbehaves.

**8. C. unless-stopped**

always also restarts it after the daemon restarts, even if you stopped it.

[Back to the questions](05-running-containers.md#check-your-understanding)

## Working with Images

**1. D. docker.io/library/nginx:latest**

Docker Hub, the official library namespace, and the latest tag.

**2. B. Just the default tag, pointing to whatever the publisher last tagged latest**

Pin a real version, such as nginx:1.27, for reproducibility.

**3. D. A digest is a content hash, so it always refers to exactly the same bytes**

A tag can be moved to new bytes; a digest cannot.

**4. A. Adds another name to an existing image, copying nothing**

One image can carry many tags.

**5. C. docker history**

It shows why an image is as big as it is.

**6. D. The container depends on its layers; remove the container first**

Or force it with docker rmi -f.

**7. B. An untagged image, often left over from rebuilds**

docker image prune removes them.

**8. C. Official images and verified publishers**

Treat random community images like any untrusted download.

[Back to the questions](06-working-with-images.md#check-your-understanding)

## Building Images with a Dockerfile

**1. A. The build context is the current directory**

-t tags the image; the context is what COPY can see.

**2. C. FROM**

FROM sets the base image.

**3. D. COPY is predictable; ADD also fetches URLs and auto-extracts archives**

Use ADD only when you need its extra behaviour.

**4. B. While the image is being built**

CMD and ENTRYPOINT run when a container starts.

**5. C. python app.py --port 9000**

Arguments after the image name replace CMD, which supplies ENTRYPOINT's arguments.

**6. A. Documents which port the app listens on**

Only -p or Compose's ports: publish a port.

**7. A. So code edits do not invalidate the cached dependency-install layer**

The cache is reused until the first instruction whose inputs changed.

**8. C. USER**

Running as non-root reduces the damage if the container is compromised.

[Back to the questions](07-dockerfile.md#check-your-understanding)

## Data and Volumes

**1. B. A container's writable layer is lost when the container is removed**

Durable data must live outside the writable layer.

**2. C. Volume**

Docker manages volumes; they are portable and easy to back up.

**3. A. During development, so the container sees source code edits live**

It maps a host directory into the container.

**4. A. The left side is a host path, not a name**

-v appdata:/var/lib/postgresql/data would be a named volume.

**5. C. In RAM only, gone when the container stops**

Useful for sensitive scratch data you do not want on disk.

**6. B. It is explicit: each part is named**

They are equivalent for common cases.

**7. D. It remains, by design, so you do not lose data by accident**

Unused volumes accumulate; clean them up deliberately.

**8. B. It deletes volumes no container references, which may be data you meant to keep**

Check docker volume ls first.

[Back to the questions](08-volumes.md#check-your-understanding)

## Networking

**1. A. Only by IP address, since there is no name resolution**

That is why you usually create your own network.

**2. C. Automatic DNS, so they reach each other by container name**

An api container can reach the database at the hostname db.

**3. B. All host interfaces, so other machines can reach it**

Use -p 127.0.0.1:8080:80 to keep it private to the host.

**4. D. No, it only documents it**

Only -p, or Compose's ports:, opens a path from the host.

**5. B. host**

No isolation and no port mapping needed.

**6. D. overlay**

It is used by Docker Swarm and clusters.

**7. A. macvlan**

The container appears as its own device on the network.

**8. C. Containers can restart with new IPs, while the name keeps working**

Docker resolves the name to the current container IP.

[Back to the questions](09-networking.md#check-your-understanding)

## Docker Compose

**1. D. A multi-container application, managed as one unit**

One file and one command replace many docker run commands.

**2. B. As the docker compose subcommand**

The standalone docker-compose is the previous version.

**3. D. Compose puts the services on one network with name resolution, and db is the service name**

The api service reaches the database at the hostname db.

**4. A. build**

image uses a ready-made image instead.

**5. C. Omit it; modern Compose ignores it and warns**

Start the file with services:.

**6. D. Start order only, not readiness**

Add a healthcheck, or make the application retry its connection.

**7. B. After the folder containing compose.yaml**

That is how it knows which containers it owns.

**8. C. docker compose up -d**

docker compose down stops and removes it.

[Back to the questions](10-compose.md#check-your-understanding)

## Registries and Image Distribution

**1. A. Build, tag with your repository name, push**

Log in first with docker login.

**2. C. So the registry knows which account the repository belongs to**

Then anyone can pull yourname/myapp:1.0 if the repository is public.

**3. D. An access token, which can be scoped and revoked**

Revoking a token does not require changing your password.

**4. B. So consumers choose how precise to be: pin exactly or follow compatible updates**

latest is just a convention; do not rely on it for reproducibility.

**5. C. docker login with authorized credentials**

Private repositories keep proprietary images off the open internet.

**6. A. Log in to it, tag with its hostname (ghcr.io/...), then push**

All registries speak the same protocol.

**7. A. Docker's own registry image**

In production, put TLS and authentication in front of it.

**8. C. docker scout**

Prefer official images, pin versions or digests, and scan.

[Back to the questions](11-registries.md#check-your-understanding)

## Best Practices

**1. B. python:3.12-slim**

Slim and pinned: smaller, and reproducible.

**2. C. An image with just the app and its runtime, without a shell or package manager**

Less software means less to exploit.

**3. A. Builds with a heavy toolchain in one stage and ships only the finished artifact in a clean final stage**

It is the highest-leverage single practice for image size and security.

**4. A. Otherwise the cache stays in an earlier layer and still adds to the image size**

Each instruction creates its own layer.

**5. C. No, it remains in the earlier layer and can be extracted**

Pass secrets at run time, and use .dockerignore.

**6. B. To stop a broad COPY . . from sweeping in files such as .env, .git or keys**

It also makes the build context smaller.

**7. D. It reduces the damage if the container is compromised**

Create and switch to an unprivileged user with USER.

**8. B. It keeps images small, scaling independent, logs clean and failures isolated**

Cooperating containers are what Compose is for.

[Back to the questions](12-best-practices.md#check-your-understanding)

## Troubleshooting and Cleanup

**1. A. docker ps -a, docker logs and docker inspect**

Start with the status, then the logs.

**2. C. The main process finished normally**

Exited (1) means it failed.

**3. B. Its main process, bash, has no terminal and nothing to do, and a container runs only as long as its main process**

Use docker run -it ubuntu bash, or run a long-lived process.

**4. D. The tag does not exist**

'pull access denied' means private or misspelled.

**5. B. docker inspect shows OOMKilled: true**

Raise the limit or fix the leak.

**6. D. Caps the container's RAM through cgroups, and the kernel kills it if it exceeds the cap**

--cpus caps CPU in the same way.

**7. A. docker system df**

See first, then prune deliberately.

**8. C. docker system prune -a --volumes**

Plain docker system prune does not touch volumes.

[Back to the questions](13-troubleshooting.md#check-your-understanding)

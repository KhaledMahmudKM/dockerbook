# Concise Book on Docker

A practical, hands-on guide to containers with Docker, explained with diagrams.

<img src="cover.svg" alt="Cover Image" width="100%" >

---

## Overview

Docker made containers mainstream. It lets you package an application together with everything it needs to run, ship that package anywhere, and start it in seconds. This book teaches Docker the way you will actually use it: building images, running containers, wiring up storage and networking, and composing multi-container applications, with **diagrams throughout** to make the concepts stick.

It is a *concise* book: practical, task-focused, and light on ceremony. Every command is one you will really type, and every diagram explains a mechanism you will really meet.

---

## What This Book Covers

- What containers are, and the Linux technology that makes them work
- Installing Docker on Linux, Windows, and macOS
- Images and containers: the core distinction
- Running and managing containers
- Building your own images with a Dockerfile
- Persisting data with volumes and bind mounts
- Connecting containers with Docker networks
- Multi-container applications with Docker Compose
- Sharing images through registries like Docker Hub
- Best practices, troubleshooting, and cleanup
- A command reference, grouped by purpose, and a glossary

---

## Who This Book Is For

- Students in software, networking, cybersecurity, cloud, and DevOps
- Developers packaging and shipping applications
- System administrators adopting containers
- Anyone who wants to understand not just *how* to use Docker, but *why* it works

You should be comfortable with the command line and basic system concepts (processes, files, networking). No prior container experience is assumed.

---

## How to Read It

Part I builds the mental model, including a chapter on the kernel technology behind containers, so read it in order the first time. Parts II and III are more reference-like; jump to the topic you need from the sidebar. The appendix collects the important commands into tables you can scan while you work.

> **Convention.** Commands are shown for a terminal with Docker installed. On Linux you may need `sudo` unless your user is in the `docker` group; Windows and macOS users run the same commands inside Docker Desktop's terminal.

---

## Notes

Docker evolves quickly. Command output, image contents, and defaults can change between releases, so always check the official documentation at **docs.docker.com** for the current details.

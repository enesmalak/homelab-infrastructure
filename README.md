# Homelab Infrastructure and Service Orchestration

![Infra](https://img.shields.io/badge/Infra-Proxmox-orange)
![Containers](https://img.shields.io/badge/Stack-Docker%20%7C%20LXC-blue)
![Security](https://img.shields.io/badge/Security-Zero%20Trust-green)

This repository documents the architecture and configuration of my Proxmox-based homelab, designed for high availability, secure remote access, and efficient storage utilization. The setup focuses on balancing performance, isolation, and maintainability.

## Infrastructure Overview
The environment is managed using Proxmox VE with a hybrid virtualization approach, combining virtual machines and LXC containers.

- **Service Management:** Docker containers are orchestrated via Komodo for streamlined deployment and lifecycle management.
- **Performance Optimization:** Resource-intensive workloads (e.g. video transcoding and network filtering) run inside LXC containers to reduce overhead and allow near-native hardware access.

## Storage Architecture
Storage is managed using MDADM (Linux Software RAID), optimized for a mixed set of 500GB and 1TB drives.

- **Data Array (RAID 5):** ~1.8 TB usable capacity, built from a combination of full disks and partitions to maximize available space while maintaining single-drive fault tolerance.
- **Backup Array (RAID 1):** ~430 GB mirrored array dedicated to critical backups, physically separated from the main data array.
- **Monitoring:** Disk health and S.M.A.R.T. metrics are monitored using Scrutiny.

## Networking & Security
The network follows a Zero Trust approach for secure external access.

- **Access Control:** Services are exposed via Cloudflare Tunnels, removing the need for open inbound ports and minimizing the attack surface. A `cloudflared` connector container maintains the outbound-only tunnel to Cloudflare's edge.
- **Traffic Routing:** Incoming traffic is securely routed through encrypted tunnels to internal services using dedicated subdomains under malakmedia.de.
- **Network Filtering:** AdGuard Home (LXC) acts as the primary DNS resolver, providing network-wide ad blocking and DNS-over-HTTPS (DoH).

## Deployed Services

### Administration & Infrastructure
- **Komodo (Docker):** Centralized platform for orchestrating and managing Docker container deployments.
- **Scrutiny (Docker):** Web UI and API for monitoring S.M.A.R.T. data, providing disk health insights and alerting.
- **AdGuard Home (LXC):** Network-wide DNS resolver with ad-blocking, tracking protection, and DNS-over-HTTPS (DoH).
- **Home Assistant (LXC):** Smart home automation platform running in an LXC container for low overhead and direct USB passthrough (e.g. Zigbee/Bluetooth).
- **Cloudflared (Docker):** Cloudflare Tunnel connector providing secure, outbound-only ingress with no exposed inbound ports.
- **Observability (Docker):** Prometheus exporter stack — **node-exporter** (host metrics), **cAdvisor** (per-container metrics), and **prometheus-pve-exporter** (Proxmox VE metrics) — scraped by an external Prometheus/Grafana instance.

### Media & Storage Optimization
- **Jellyfin (Docker):** Self-hosted media server with Intel QuickSync support (/dev/dri) for hardware-accelerated transcoding.
- **Immich (Docker):** Self-hosted photo and video backup with machine-learning features (facial recognition, object/CLIP search) backed by a pgvector database.
- **Audiobookshelf (Docker):** Server for managing, tracking, and streaming audiobooks and podcasts.
- **Unmanic (LXC):** Automated media optimization worker that scans libraries and converts video files to H.265/HEVC to improve storage efficiency.

### Productivity & Utilities
- **Paperless-ngx (Docker):** Document management system with OCR for automated indexing and archiving.
- **Mealie (Docker):** Recipe management and meal planning application.
- **BentoPDF (Docker):** Lightweight web app for local PDF processing (merge, split, compress).
- **IT-Tools (Docker):** Collection of web-based utilities for everyday IT and development tasks.

### Self-Hosted AI
A fully on-premise AI stack — no user or conversation data leaves the network.
- **Honcho (Docker):** Self-hosted, OpenAI-compatible memory and personalization backend (Postgres + pgvector, Redis, and a dedicated embeddings service). Runs locally in place of the hosted cloud so each project's data stays isolated on-prem.
- **Open WebUI (Docker):** Web front-end for interacting with local and API-based LLMs.
- **RPG App (Docker):** A custom, self-built AI-driven tabletop-RPG engine (FastAPI). Attaches to the Honcho network for persistent, per-campaign NPC and world memory — an example of building on top of the self-hosted AI backend rather than only deploying off-the-shelf services.

## Backup and Disaster Recovery
Data integrity is ensured through a structured and automated backup strategy:

- **Automation:** Daily backups scheduled at 03:30.
- **Retention:** 7-day rolling retention policy.
- **Storage:** Backup archives are stored on a dedicated RAID 1 array, ensuring availability in case of primary array failure.

## Service Endpoints
- **Jellyfin** → https://kino.malakmedia.de  
- **Audiobookshelf** → https://audio.malakmedia.de  
- **BentoPDF** → https://pdf.malakmedia.de
- **IT-Tools** → https://tools.malakmedia.de

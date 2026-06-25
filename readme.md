### WhatsApp Subscriber Notification Platform

A Dockerized **n8n + Evolution API** stack that powers an automated WhatsApp channel for a Brazilian publishing client: readers opt in via a keyword, new articles are detected from RSS feeds, and matching subscribers receive a broadcast — all on infrastructure cost low enough to justify a channel with no direct revenue tie-in.

Meta's official WhatsApp Business API was ruled out on a per-message cost basis, which pushed the design toward an unofficial gateway (Evolution API) behind a residential proxy, with deliberate anti-ban engineering (jittered send delays, batching, per-recipient dedup) built in from the start rather than added after a ban.

**Full write-up, architecture diagrams, and the engineering tradeoffs behind these decisions:** [acoelho.dev](https://acoelho.dev)

### Stack

- **n8n** — orchestration layer; all business logic (subscriber lifecycle, RSS polling, message delivery, data retention) lives here as workflows.
- **Evolution API** — unofficial WhatsApp gateway; sends outbound messages and forwards inbound events to n8n via webhook.
- **PostgreSQL** — persistent store for n8n state and the project's data tables (participants, interactions, news, delivery receipts).
- **Redis** — caching/session layer for Evolution API connection stability.
- **Caddy** — reverse proxy with automatic TLS in front of n8n and the Evolution API manager.
- **Webshare** — residential proxy that Evolution API routes through, to reduce ban risk from datacenter-IP traffic patterns.

### Workflows

- **Participant Manager + Admin Messages** — webhook-triggered on every inbound message; handles subscriber opt-in/opt-out keywords, lifecycle messaging, and a hidden admin broadcast channel.
- **RSS Check** — hourly scheduled poll of the client's article feeds; dedups against previously-seen posts and triggers delivery for anything new.
- **Send Message to Participants** — shared sub-workflow called by both of the above; the single delivery path for the system, with batching, randomized send delays, and per-recipient dedup.
- **Data cleanup jobs** — two scheduled workflows that prune stale interaction state and old delivery records so the data tables don't grow unbounded.

### Deployment

Deployed via a two-stage GitHub Actions pipeline (`.github/workflows/deploy.yml`): a validation job checks the Docker Compose config before anything touches the server, then a deploy job SSHes into the VPS, resets the working copy to match `main`, and rolls out new images with `docker compose pull && docker compose up -d --remove-orphans`. `Caddyfile` handles TLS termination for both the n8n and Evolution API endpoints.

### Setup

Requires Docker Desktop and Git. Copy `.env.example` to `.env` and fill in the required keys, then `docker compose up -d` to start the stack.

### License
Free to use.

# WhatsApp Feed Automation

**An automated WhatsApp messaging system that turns RSS-published articles into opt-in subscriber broadcasts, built for a real client under real constraints.**

Built by Alexis Coelho · [acoelho.dev](https://acoelho.dev) · [LinkedIn](https://linkedin.com/in/coelhoalexis) · **[Read the full case study](https://acoelho.dev/en/projects/n8n_whatsapp)**, with architecture diagrams and the engineering tradeoffs behind each decision.

## The constraint

The client, an established publishing company with two news platforms, wanted a direct-to-reader channel to drive traffic back to its sites without wearing out its email newsletter. The goal was engagement, not revenue: no message produced a sale, so the channel had to run as a minimal operating cost.

That one constraint ruled out Meta's official WhatsApp Business API, which bills per message. The system uses Evolution API, an unofficial gateway, instead, and engineers against that choice's risks from day one: paced and batched sends, a residential proxy, session persistence, and a ready replacement route for the number. Readers opt in by messaging a keyword and leave with one word.

## Stack

- **n8n:** the orchestration layer. All business logic (subscriber lifecycle, RSS polling, message delivery, data retention) lives here as workflows.
- **Evolution API:** an unofficial WhatsApp gateway. It sends outbound messages and forwards inbound events to n8n via webhook.
- **PostgreSQL:** the persistent store for n8n state and the project's data tables (participants, interactions, news, delivery receipts).
- **Redis:** the caching and session layer, for Evolution API connection stability.
- **Caddy:** a reverse proxy with automatic TLS in front of n8n and the Evolution API manager.
- **Webshare:** a residential proxy that Evolution API routes through, to reduce ban risk from datacenter-IP patterns.

## Workflows

The exports in [`workflows/`](workflows) are redacted: example domains and zeroed IDs.

- **Participant Manager + Admin Messages:** webhook-triggered on every inbound message. It handles opt-in and opt-out keywords, lifecycle messaging, and a hidden admin broadcast channel.
- **RSS Check:** an hourly poll of the client's article feeds. It dedups against previously seen posts and triggers delivery for anything new.
- **Send Message to Participants:** the shared sub-workflow both of the above call, and the system's single delivery path. It handles batching, randomized send delays, and per-recipient dedup, so a re-run can never double-send.
- **Data cleanup jobs:** two scheduled workflows that prune stale interaction state and old delivery records, so the data tables don't grow unbounded.

## Deployment

A two-stage GitHub Actions pipeline ([`.github/workflows/deploy.yml`](.github/workflows/deploy.yml)) runs on every push to `main`:
1. **Validate.** A job checks the Docker Compose config before anything touches the server.
2. **Deploy.** A job connects to the VPS over SSH, resets the working copy to match `main`, and rolls out new images with `docker compose pull && docker compose up -d --remove-orphans`.

[`Caddyfile`](Caddyfile) handles TLS termination for both the n8n and Evolution API endpoints. Secrets live only in the server's `.env`, and [`.env.example`](.env.example) documents the keys it needs.

## What I'd do differently at scale

If the channel ever justified the costs or compliance became a hard requirement, the right move would be to migrate to Meta's official API. For a cost-constrained engagement experiment, Evolution + proxy was the correct pragmatic call. However, it is definitely not a permanent, enterprise-scale architecture solution.

## Setup

It needs Docker Desktop and Git. Copy `.env.example` to `.env`, fill in the required keys, then run `docker compose up -d` to start the stack.

## License

[MIT](LICENSE.md)
